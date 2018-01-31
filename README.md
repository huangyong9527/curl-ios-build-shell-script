# curl-ios-build-shell-script
Build libcurl with shell script - only support to use on iOS, and bitcode is supported.

# Usage
1.Download curl source code(gz, zip and bz2 fromat are supported), e.g. curl-7.46.0.tar.bz2 

2.Move curl-build.sh into the folder where the curl-7.46.0.tar.bz2 is cotained.

3.Edit curl-build.sh, set the value of CURL_COMPRESSED_FN and -miphoneos-version-min( supported minimal iOS version ).
If you want to support the bitcode, find "export CFLAGS=-arch ${ARCH} -isysroot ${IOS_SDK_PATH} -miphoneos-version-min=6.0", add the -fembed-bitcode option at end of line. You must change the value of "--with-ssl" where the openssl libraries are installed, or disable ssl with "--without-ssl".

4.cd the the folder where the curl-7.46.0.tar.bz2 is cotained.

5.Execute curl-build.sh, libraries are created at "curl-version-build/universal/".

# 备注
查看openssl-ios-build-shell-script和curl-ios-build-shell-script这两个项目，
下载openssl-build.sh和curl-build.sh，并准备openssl, curl源码压缩包，下面正式开讲。

  iOS跨平台编译无非是要配置好编译器，SDK路径和最小支持的iOS版本，
还有开源项目本身要设置的一些选项，编译完了之后调用lipo合成universal的库；
所谓自动编译，无非是在shell脚本里面调用这些里面用到一些命令，这就是基本原理。

 说几个注意点：
1>. xcrun -sdk iphoneos --show-sdk-path ／ xcrun -sdk iphonesimulator --show-sdk-path用来查找真机和模拟器SDK的路径；

2>. xcrun --find gcc / xcrun --find clang用来查找编译器的路径；

3>. -miphoneos-version-min选项指定最小支持的iOS版本；

4>. -fembed-bitcode选项开启bitcode的支持，去掉就不支持bitcode；

5>. 编译curl，暂时只支持gcc编译，虽然内部是调用clang，
我试了用clang编译，会报错：error "We can't compile without socket() support!"，若有人知道怎么解决，请告诉我；

6>. 编译完各平台的curl，直接使用同一份头文件，XCode会报错，说curlbuild.h里面的"[CurlchkszEQ(long, CURL_SIZEOF_LONG)]"有问题，
这是因为32位和64位的curlbuild.h内容不一样，我们可以把32位和64位的curlbuild.h放在同一个目录下，重命名为curlbuild-32.h和curlbuild-64.h，
然后机智地建一个curlbuild.h，里面通过"__LP64__"宏来确定是否是64位的平台来包含不同的头文件；

7>. 编译openssl，需要导出CROSS_TOP和CROSS_SDK两个变量，这两个变量合起来就是SDK的绝对路径；

8>.编译x86_64平台的openssl，Configure时需要指定no-asm选项，否则会报错；

9>. openssl-build.sh可能只支持较新版本的openssl，因为新的版本内建支持iOS，要支持老版的，请自己修改脚本。
