https://maven.apache.org/index.html

项目管理工具
* 项目对象模型 (POM：Project Object Model)，
* 标准集合，
* 项目生命周期(Project Lifecycle)，
* 依赖管理系统(Dependency Management System)，
* 运行定义在生命周期阶段(phase)中插件(plugin)目标(goal)的逻辑。


全局 setting 与用户 setting

conf/setting.xml 全局配置。
${user.dir}/.m2/settings.xml 用户配置
maven 会先找用户配置，如果找到则以用户配置文件为准，否则使用全局配置文件。

依赖范围
* compile：编译范围会被打包。
* provided：provided依赖只有在当JDK或者一个容器已提供该依赖之后才使用， provided依赖在编译和测试时需要，在运行时不需要，比如：servlet api被tomcat容器提供。
* runtime：runtime依赖在运行和测试系统的时候需要，但在编译的时候不需要。会被打包。
* test：test范围依赖 不会被打包。
* system：system范围依赖与provided类似，必须显式的提供一个对于本地系统中JAR文件的路径，需要指定systemPath磁盘路径，system依赖不推荐使用

# pom
* super pom 所有pom的父pom
* 父pom  单继承
* 当前pom
* effective pom 有效pom   mvn help:effective-pom



```xml



```





```xml
<project > ：文件的根节点
<modelversion > ： pom.xml 使用的对象模型版本
<groupId > ：项目名称，一般写项目的域名
<artifactId > ：模块名称，子项目名或模块名称
<version > ：产品的版本号 .
<packaging > ：打包类型，一般有 jar、war、pom 等
<name > ：项目的显示名，常用于 Maven 生成的文档。
<description > ：项目描述，常用于 Maven 生成的文档
<dependencies> ：项目依赖构件配置，配置项目依赖构件的坐标
<build> ：项目构建配置，配置编译、运行插件等。
```





# Plugin
https://maven.apache.org/plugins/index.html










# Lifecycle
Lifecycle 基于插件完成

* Clean
	pre-clean
	clean
	post-clean
* site     生成项目报告，站点，发布站点
	pre-site 
	site 
	post-site 
	deploy-site
* default        构建的核心部分，编译，测试，打包，部署等等
	validate  
	generate-sources  
	process-sources  
	generate-resources  
	process-resources复制并处理资源文件，至目标目录，准备打包。 
	compile编译项目main目录下的源代码。 
	process-classes  
	generate-test-sources  
	process-test-sources 
	generate-test-resources  
	process-test-resources复制并处理资源文件，至目标测试目录。 
	test-compile编译测试源代码。 
	process-test-classes
	test使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
	prepare-package  
	package接受编译好的代码，打包成可发布的格式，如JAR。 
	pre-integration-test  
	integration-test 
	post-integration-test  
	verify 
	instal将包安装至本地仓库，以让其它项目依赖。 
	deploy将最终的包复制到远程的仓库，以让其它开发人员共享;或部署到服务器













配置阿里云镜像和jdk版本
```xml
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
</mirrors>
 
<profiles>
    <profile>
        <id>jdk-1.8</id>
        <activation>
        <activeByDefault>true</activeByDefault>
            <jdk>1.8</jdk>
        </activation>
        <properties>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
            <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        </properties>
    </profile>
</profiles>
```