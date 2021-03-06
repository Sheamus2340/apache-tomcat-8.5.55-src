# Tomcat源码分析

## Tomcat架构

### Tomcat功能

- HTTP服务器

	- Socket通信（TCP/IP）、解析HTTP报文

- servlet容器

	- 管理Servlet（自带的servlet + 自定义的servlet）,servlet 处理具体的业务逻辑

### Tomcat核心组件

- Connector（连接器）

	- 作用

		- 实现HTTP服务器

	- 实现名称

		- coyote

	- 子组件

		- ProtocolHandler（协议处理器）

			- EndPoint（端点）

				- 协议

					- TCP/IP

				- 作用

					- 进行socket通信，使用TCP/IP

				- 支持的IO模型

					- BIO
					- NIO

						- Tomcat 8默认的模型

					- NIO2（AIO）
					- APR

						- Apache可移植扩展包，需要安装在操作系统上，然后添加配置指定这个IO模型

			- Processor（处理器）

				- 协议

					- HTTP/AJP

				- 作用

					- 解析处理HTTP报文，使用HTTP协议；AJP是用于与Apache服务器通信处理静态资源的协议

				- 输出结果

					- 将请求封装成Request对象

		- Adapter（适配器）

			- 作用

				- 将Request对象转换成ServletRequest对象，便于Catalina容器使用
				- 将ServletResponse对象转换成Response对象便于ProtocolHandler的Processor使用

			- 子主题 2

- Container

	- 作用

		- 实现Servlet容器

	- 实现名称

		- Catalina

### 整体架构

- Server（1）

	- 含义

		- 代表Tomcat实例，又因为Catalina容器（Servlet容器）是Tomcat核心，所以通常也说Tomcat实例是一个Catalina实例

	- 子组件

		- Service（n）

			- 子组件

				- Connector（n）

					- 子组件

						- ProtocolHandler（1）

							- 子组件

								- EndPoint（1）

									- 一个EndPoint只能监听一个端口

								- Processor（1）

						- Adapter（1）

				- Container（1）

					- 子组件

						- Engine

							- 引擎是Catalina的核心
							- Host（n）

								- 虚拟主机
								- Context（n）

									- 一个虚拟主机支持多个Web应用部署，上下文是使用由Servlet规范中指定的Web应用程序表示，不论是压缩过的war包的文件形式还是未压缩的目录形式
									- 对应server.xml配置的<Host>节点属性 appBase
									- Wrapper（n）

										- Wrapper是Servlet的包装类
										- 一个Wrapper对应一个Servlet

## 分析源码

### 原则

- 定焦原则

	- 抓主线（抓住一条核心流程进行分析，不要漫无目的的看源码）

- 宏观原则

	- 站在上帝的视角，先脉络后枝叶（切忌试图搞明白看到的每一行代码）

### 方法和技巧

- 断点

	- 观察调用栈

- 反调

	- 右键 -> Find Usages

- 经验之谈

	- 比如一些doXXX()，service() ....一般是具体的实现方法

- 见名思意

	- 通过方法名称知道方法的作用

- 多实际动手操练，灵活应用上述方法技巧

## 源码构建

### 步骤

- 下载源码

	- 官网

- 解压zip文件
- 进入apache-tomcat-8.5.55-src文件夹，创建pom.xml文件，进行maven管理项目
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>Tomcat8.5</artifactId>
    <name>Tomcat8.5</name>
    <version>8.5</version>
 
    <build>
        <finalName>Tomcat8.5</finalName>
        <sourceDirectory>java</sourceDirectory>
        <testSourceDirectory>test</testSourceDirectory>
        <resources>
            <resource>
                <directory>java</directory>
            </resource>
        </resources>
        <testResources>
            <testResource>
                <directory>test</directory>
            </testResource>
        </testResources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
 
    <dependencies>
        <!--添加jaxrpc依赖 远程调用工具包-->
        <dependency>
            <groupId>javax.xml</groupId>
            <artifactId>jaxrpc</artifactId>
            <version>1.1</version>
        </dependency>
        <!--添加soap依赖 soap协议处理工具包-->
        <dependency>
            <groupId>javax.xml.soap</groupId>
            <artifactId>javax.xml.soap-api</artifactId>
            <version>1.4.0</version>
        </dependency>
        <!--解析WSDL文件 webservice的wsdl文件工具-->
        <dependency>
            <groupId>wsdl4j</groupId>
            <artifactId>wsdl4j</artifactId>
            <version>1.6.2</version>
        </dependency>
        <!--jsp编译器依赖 Eclipse Java编译器-->
        <dependency>
            <groupId>org.eclipse.jdt.core.compiler</groupId>
            <artifactId>ecj</artifactId>
            <version>4.5.1</version>
        </dependency>
        <!--  ant管理工具      -->
        <dependency>
            <groupId>ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.7.0</version>
        </dependency>
        <!--easymock用于模拟HttpServletRequest对象从而使测试顺利进行 辅助单元测试-->
        <dependency>
            <groupId>org.easymock</groupId>
            <artifactId>easymock</artifactId>
            <version>3.4</version>
        </dependency>
        <!--添加单元测试框架-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
</project>
```
- 在apache-tomcat-8.5.55-src目录下创建source文件夹
- 将conf和webapps文件夹移动到source目录下
- 将源码导入IDEA中
- 给Tomcat源码程序启动类Bootstrap配置VM参数，因为Tomcat源码运行也需要加载配置文件等
``` properties
-Dcatalina.home=apache-tomcat-8.5.55-src/source/ 
-Dcatalina.base=apache-tomcat-8.5.55-src/source/ 
-Djava.endorsed.dirs=apache-tomcat-8.5.55-src/source/endorsed 
-Djava.io.tmpdir=apache-tomcat-8.5.55-src/source/temp 
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager 
-Djava.util.logging.config.file=apache-tomcat-8.5.55-src/source/conf/logging.properties
```

### 注意点

- 需要手动添加Jsper的编译器，这样才能请求Jsp页面不出错
