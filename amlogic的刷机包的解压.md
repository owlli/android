# amlogic的刷机包的解压



amlogic的刷机包格式为img,使用file查看可以看到是数据文件

```shell
lzh@ws:s905$ file aml_upgrade_package.img 
aml_upgrade_package.img: data
```

这里面的数据是安卓系统bootloader,system,recovery,vendor分区的数据,推测是被amlogic加密成这个img文件了

在网上找到了解压方法,创建一个test.c文件内容如下

```c
#include <errno.h>
#include <inttypes.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>

uint32_t convert(uint8_t *test, uint64_t loc) {
    return ntohl((test[loc] << 24) | (test[loc+1] << 16) | (test[loc+2] << 8) | test[loc+3]);
}

void main(int argc, char * argv[]) {
    FILE *fileptr;
    uint8_t *buffer;
    long filelen;

    FILE *f;
    char *filename;
    uint64_t record;
    uint64_t record_loc;
    uint64_t file_loc;
    uint64_t file_size;

    printf("file is %s\n", argv[1]);
    fileptr = fopen(argv[1], "rb");
    fseek(fileptr, 0, SEEK_END);
    filelen = ftell(fileptr);
    rewind(fileptr);

    buffer = (uint8_t *)malloc((filelen+1)*sizeof(uint8_t));
    fread(buffer, filelen, 1, fileptr);
    fclose(fileptr);

    for (record = 0; record < (uint8_t)buffer[0x18]; record = record + 1){
        record_loc = 0x40 + (record * 0x240);

        filename = (malloc(32));                                                                     
        sprintf(filename,"%s.%s",(char *)&buffer[record_loc+0x120], (char *)&buffer[record_loc+0x20]);

        file_loc = convert(buffer,record_loc+0x10);
        file_size = convert(buffer,record_loc+0x18);

        f = fopen(filename, "wb");
        if (f == NULL) {
            printf("ERROR: could not open output\n");
            printf("the error was: %s\n",strerror(errno));
            free(filename);
            continue;
        }
        fwrite(&(buffer[file_loc]), sizeof(uint8_t), (size_t)file_size, f); 
        fclose(f);
        free(filename);
    }   
    free(buffer);
}
```

gcc编译

```shell
gcc test.c -o test
```

解压

```shell
lzh@ws:s905$ ./test aml_upgrade_package.img 
```



其中system.PARTITION就是安卓的system分区

```shell
lzh@ws:s905$ file system.PARTITION 
system.PARTITION: Android sparse image, version: 1.0, Total of 286720 4096-byte output blocks in 1912 input chunks.
```

使用simg2img工具可以把"Android sparse image"转换成"Linux rev 1.0 ext4 filesystem data"

```shell
lzh@ws:s905$ simg2img system.PARTITION system.img
```

然后就可以挂载到一个目录查看system.img里面的文件了

如果没有simg2img工具,直接在ubuntu的bash下敲这个命令应该会给出这个命令的apt安装方法

也可以用安卓源码里自带simg2img工具,源码目录在安卓源码下的

`system/core/libsparse/simg2img.c // 将sparse image转换为raw image；`和`system/core/libsparse/img2simg.c // 将raw image转换为sparse image；`



如果编译过安卓系统,可以在out/host/linux-x86/bin/目录中找到,如果有安卓源码但没编译过安卓系统,通过下面命令生成simg2img和img2simg

```shell
#进安卓源码目录
lzh@ws:nougat$ source ./build/envsetup.sh 
lzh@ws:nougat$ lunch 需要编译的安卓选项
lzh@ws:nougat$ make img2simg_host
lzh@ws:nougat$ make simg2img_host
```

编译完成后的文件在安卓源码下的`out/host/linux-x86/bin/img2simg`目录





## 参考链接

[amlogic S905X udpate imge的压缩和解压](https://blog.csdn.net/sy373466062/article/details/71651657/)

[Android中system.img的两种格式及其相互转换方法](https://blog.csdn.net/howellzhu/article/details/43165507)













