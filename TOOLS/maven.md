

项目管理工具
* 项目对象模型 (POM：Project Object Model)，
* 标准集合，
* 项目生命周期(Project Lifecycle)，
* 依赖管理系统(Dependency Management System)，
* 运行定义在生命周期阶段(phase)中插件(plugin)目标(goal)的逻辑。




依赖范围
* compile：编译范围会被打包。
* provided：provided依赖只有在当JDK或者一个容器已提供该依赖之后才使用， provided依赖在编译和测试时需要，在运行时不需要，比如：servlet api被tomcat容器提供。
* runtime：runtime依赖在运行和测试系统的时候需要，但在编译的时候不需要。会被打包。
* test：test范围依赖 不会被打包。
* system：system范围依赖与provided类似，必须显式的提供一个对于本地系统中JAR文件的路径，需要指定systemPath磁盘路径，system依赖不推荐使用



