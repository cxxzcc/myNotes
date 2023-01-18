https://maven.apache.org/index.html


![](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/maven概念模型图.png)

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

<!--项目名称，定义为组织名+项目名，类似包名-->
<groupId>com.itheima</groupId>
<!-- 模块名称 -->
<artifactId>hello_maven</artifactId>
<!-- 当前项目版本号，snapshot 为快照版本即非正式版本，release 为正式发布版本 -->
<version>0.0.1-SNAPSHOT</version>
<packaging > ：打包类型
    jar：执行 package 会打成 jar 包
    war：执行 package 会打成 war 包
    pom ：用于 maven 工程的继承，通常父工程设置为 pom

<project > ：文件的根节点
<modelversion > ： pom.xml 使用的对象模型版本
<name > ：项目的显示名，常用于 Maven 生成的文档。
<description > ：项目描述，常用于 Maven 生成的文档
<dependencies> ：项目依赖构件配置，配置项目依赖构件的坐标
<build> ：项目构建配置，配置编译、运行插件等。


<scope>  </scope>
<scope>system</scope>
<systemPath>
/${project.basedir}/../mes-biz/src/main/resources/lib/sapjco-3.0.6.jar
</systemPath>
<!--
 compile 默认范围 可以不写（编译、测试、运行 都有效 ）
 provided （编译、测试 有效，运行时无效 防止和 tomcat 下 jar 冲突）
 runtime （测试、运行 有效 ）
 test （测试有效）
 强到弱顺序：compile > provided > runtime > test

 system：system 范围依赖与 provided 类似，但是你必须显式的提供一个对于本地系统中 JAR文件的路径，需要指定 systemPath 磁盘路径，system依赖不推荐使用。

import依赖范围使用要求:
	打包类型必须是pom
	必须放在dependencyManagement中

-->

//排除jar包
<exclusions>  
    <exclusion>        
	    <groupId></groupId>        
	    <artifactId></artifactId>    
    </exclusion>
</exclusions>


<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>

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