# linux下使用gradlew编译Android apk



## 方法

### 给源码中的gradlew执行权限

```shell
chmod +x gradlew
```



### 解决gradle依赖

在源码目录执行

```shell
./gradlew
```

第一编译会提示下载gradle-4.6-all.zip,我用a网络下载总是无法下载,提示连接超时,连到b网络就可以下载了,原因未明



### 配置Android SDK

解决gradle依赖后还是无法编译,提示

```shell
SDK location not found. Define location with sdk.dir in the local.properties
```

需要配置Android SDK

下载

在http://tools.android-studio.org/index.php/sdk/下载android-sdk_r24.4.1-linux.tgz

解压

```shell
tar -zvxf android-sdk_r24.4.1-linux.tgz -C /usr/local
```

配置环境变量

在~/.bashrc最后加上

```shell
export ANDROID_HOME=/usr/local/android-sdk-linux
export PATH=$ANDROID_HOME/tools:$PATH 
```

执行

```shell
source ~/.bashrc
```

安装sdk包

安装所有包(不推荐)

```shell
android update sdk --no-ui
```

查看所有包

```shell
android list sdk --all
```

上面会打印所有包及它的序列号,按序列号安装

```shell
android update sdk -u --all --filter 1,2,4,49
```

我这里安装的是

>  1- Android SDK Tools, revision 25.2.5
>
> 4- Android SDK Build-tools, revision 28.0.3
>
> 49- SDK Platform Android 8.1.0, API 27, revision 3

不同使用不同版本的源码所需的sdk可能会不同,在执行gradlew时会给出提示,按提示安装即可

### 编译

我这里执行

```shell
./gradlew assembleDebug
```

就可以编译apk了



## 可能出现问题

### jdk版本和源码中不符

执行gradlew时提示

```shell
* What went wrong:
A problem occurred evaluating project ':app'.
> java.lang.UnsupportedClassVersionError: com/android/build/gradle/AppPlugin : Unsupported major.minor version 52.0
```

切换jdk就行了,具体需要哪个版本的jdk,自行搜索上面的"Unsupported major.minor version 52.0"

### 无法下载gradle依赖包

执行gradlew时提示

```shell
* What went wrong:
A problem occurred configuring root project 'bt-hidd'.
> Could not resolve all artifacts for configuration ':classpath'.
   > Could not download kotlin-stdlib-jdk8.jar (org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.2.71)
      > Could not get resource 'https://jcenter.bintray.com/org/jetbrains/kotlin/kotlin-stdlib-jdk8/1.2.71/kotlin-stdlib-jdk8-1.2.71.jar'.
         > Could not HEAD 'https://jcenter.bintray.com/org/jetbrains/kotlin/kotlin-stdlib-jdk8/1.2.71/kotlin-stdlib-jdk8-1.2.71.jar'.
            > Connect to jcenter.bintray.com:443 [jcenter.bintray.com/52.89.158.216, jcenter.bintray.com/35.165.178.167] failed: connect timed out
```

因为被墙了,无法连到外网下载依赖包

改源码下的build.gradle文件,将

```shell
buildscript {
    repositories {
        google()
        jcenter()
    }
..................
allprojects {
    repositories {
        google()
        jcenter()
    }
..................
```

改成

```shell
buildscript {
    repositories {
        google()
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
        maven{ url'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
    }
..................
allprojects {
    repositories {
        google()
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
        maven{ url'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
    }
..................
```

查找资料,发现可以一劳永逸地改变所有编译时的仓库地址,但未亲自测试

在~/.gradle下创建init.gradle文件,添加

```shell
allprojects{
    repositories {
        def REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public/'
            all { ArtifactRepository repo ->
                if(repo instanceof MavenArtifactRepository){
                    def url = repo.url.toString()
                        if (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('https://jcenter.bintray.com/')) {
                            remove repo
                }
                } 
            }
        maven {
            url REPOSITORY_URL
        }
    }
}
```





## 参考链接

[jcenter下载不了时，用国内镜像下载解决](https://www.jianshu.com/p/73c37d4d97dc)

[maven常见问题处理（3-3）Gradle编译时下载依赖失败解决方法](https://www.cnblogs.com/lexiaofei/p/6979887.html)

[Linux -- 安装配置Android SDK](https://blog.csdn.net/u011974797/article/details/78973012)



