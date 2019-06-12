# Android源码编译



## 编译环境的搭建

* 先安装一些必要的软件,一般搜一下就有,不同系统需要的软件可能不同

* 安卓编译都需要jdk,不同版本的安卓系统可能需要的jdk版本不同,所以系统可能需要安装不同版本的jdk

ubuntu下java版本的切换

```shell
update-alternatives --config java
```

可能需要将系统中java开头的命令全部切换一下,比如

java,javac,javadoc,javah,javap

## 编译

* 在安卓源码目录下,一般都会有已经编辑好的编译系统脚本,根据不同的参加选择编译uboot,内核,文件系统等

如果没有,一般都会先执行下面命令配置所需环境变量

```shell
source ./build/envsetup.sh
```

**这个命令一定要在安卓源码的根目录下执行**

执行完后,一般就使用以下几条命令

lunch	配置硬件版本(这个必须最先执行)

m		编译所有模块

mm		编译当面目录下的模块

mma	编译当面目录下的模块,并自动解决依赖关系

mmm	后面加模块所在的目录名

mmm	后面加模块所在的目录名,并自动解决依赖关系

可以通过hmm命令查看所有可以执行的命令

也可以通过下面的命令编译单个模块

```shell
make LOCAL_MODULE模块名
```

这个命令会先检查所有Android.mk文件,找到其中的模块名编译

* Android.mk文件一定要按照规范的格式编写,否则报错,比如

```shell
build/core/base_rule.mk:141: test:local_built_module and local installed module must not be defined by component makefiles. 
```

解决办法:

在第一行

LOCAL_PATH:= $(call my-dir)

下加入

include $(CLEAR_VARS)  

我在google groups上找到了一篇帖子--[How to add 3rd party library](https://groups.google.com/forum/#!topic/android-framework/qLZ743pSEBo),里面有关于这个问题的讨论,现摘抄总结的一楼回复:

> To sum up, adding 3rd party libraries, which means libraries are not 
>
> originated by Android but come from others, should edit the 
>
> Android.mk. 
>
> For the Android.mk format should be required following basic 
>
> information, 
>
> /======================== 
>
> LOCAL_PATH:= $(call my-dir) 
> include $(CLEAR_VARS) 
>
> ========================/ 
>
> To start the new compile content in the same Android.mk should add 
>
> include $(CLEAR_VARS) again such as, 
>
> /======================== 
>
> LOCAL_PATH:= $(call my-dir) 
> include $(CLEAR_VARS) 
>
> [Compile Content part 1] 
>
> ... 
>
> include $(CLEAR_VARS) 
>
> [Compile Content part 2] 
>
> ... 
>
> ========================/ 
>
> (For more details such as LOCAL_MODULE and LOCAL_SRC_FILES..., please 
>
> study more by yourselves) 
>
> However, if you find no above basic information in some Android.mk, it 
>
> should be already incleded in the parent directory, and you are 
>
> looking into sub-directory. 
>
> For example, you can see the Android.mk under "hardware 
>
> \libhardware_legacy\
>
> flashlight\". 
>
> You see no above basic information in this Android.mk, but you can 
>
> find the information in the parent directory of flashliht, "hardware 
>
> \libhardware_legacy" 
>
> To adding the 3rd party libraries, you should have some basic 
>
> knowledge about the libraries. 
>
> For those *.a are for static libraries, and *.so are for shared 
>
> libraries. 
>
> Static Libraries: 
>
> /=============================
>
> = 
>
> LOCAL_PREBUILT_LIBS :=         1lib.a \ 
>
>                         2lib.a \ 
>
>                         3lib.a \ 
>
>                         4lib.a 
>
> include $(BUILD_MULTI_PREBUILT) 
>
> LOCAL_STATIC_LIBRARIES :=         1lib \ 
>
>                         2lib \ 
>
>                         3lib \ 
>
>                         4lib 
>
> ==============================
>
> / 
>
> Shared Libraries: 
>
> /=============================
>
> = 
>
> LOCAL_PREBUILT_LIBS :=         alib.so 
>
> include $(BUILD_MULTI_PREBUILT) 
>
> LOCAL_SHARED_LIBRARIES := alib 
>
> ==============================
>
> / 
>
> Remember, different kind of libraries should be required different way 
>
> to ontegration!! 
>
> Hope someone who has the same problem in the future can find this 
>
> information useful!! 