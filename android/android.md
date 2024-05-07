


log 范围由小到大
* Log.e: 表示错误信
* Log.w: 表示警告信息。
* Log.i: 表示一般消息。
* Log.d: 表示调试信息
* Log.v: 表示沉余信息。


原生开发 高性能 java kotlin
混合开发 跨平台 flutter react native


项目结构
app目录下
* manifests子目录， 下面只有一个XML文件AndroidManifest.xml, 运行配置文件。
* java子录，下面有3个com.example.myapp包，第一个包存放当前模块的Java源代码，后面两个包存放测试用的Java代码。
* res子目录，存放资源文件。res下面又有4个子目录:
	* drawable 目录存放图形描述文件与图片文件。
	* layout 布局文件。
	* mipmap 启动图标。
	* values 常量定义文件，例如字符串常量strings.xml、像素常量dimens.xml、颜色常量colors.xml、样式风格定义styles.xml等。
Gradle Scripts下面主要是工程的编译配置文件，主要有:
* build.gradle, 该文件分为项目级与模块级两种，用于描述App工程的编译规则。
* proguard-rules.pro, 该文件用于描述Java代码的混淆规则。
* gradle.properties, 该文件用于配置编译工程的命令行参数，-般无须改动。
* settings.gradle, 该文件配置了需要编译哪些模块。初始内容为include ':app',表示只编译app模块。
* local.properties, 项目的本地配置文件，它在工程编译时自动生成，用于描述开发者电脑的环境配置，包括SDK的本地路径、NDK的本地路径等。


