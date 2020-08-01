# ServerConfig

> SpringCloud 组件，ServerConfig 是配置服务中心，组件，用于统一管理项目配置''



## 原理

### 读取配置文件内容到服务中心
通过启动一个springboot服务（server-config），配置好git仓库地址（也可以svn或者其他），通过配置好的文件地址，访问配置文件，将配置读取到服务中心，并且文件内容修改后，可以实时刷新。

### 客户端到配置中心读配置内容
其他微服务访问配置不需要访问git，只需要通过访问ServerConfig读取配置，能到达统一配置的目的。



## 搭建

> 前提概要，本搭建使用ide:Spring Tool Suite 4 
>
> Version: 4.3.0.RELEASE
> Build Id: 201906200901
>
> 安装好maven 3.6.3，并配置好网络通畅的maven仓库地址，有些国外仓库地址可能网速不好，下载依赖慢。
>
> 国内镜像仓库反而更快 https://www.cnblogs.com/Narule/p/12595960.html

## Server

file  -> new -> Spring Starter Project
![](https://img2020.cnblogs.com/blog/1436620/202003/1436620-20200330000626226-533036850.png)

### pom依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.5.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.wunanyu</groupId>
	<artifactId>Config</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Config</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Hoxton.SR3</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```



### 配置文件

#### application.yml

```yaml
server:
  port: 8888  #	服务端口号
spring:
  application:
    name: server-config  #服务名
  cloud:
    config:
      server:
        git:
          uri: /usr/data/git/cloud/config  #linux配置文件地址
          # 这里可以配置文件路径，也可以配置服务器路径，windows需要添加//  
          # 如uri: file://${user.home}/config-repo
          # github  uri：https：//github.com/repo

#
#logging:
#config: src/main/resources/logback-spring.xml
log:
  path: log  #日志路径
```



更详细可以见https://www.cnblogs.com/hellxz/p/9306507.html

#### logback-spring.xml

> 日志打印的配置文件已经配置好，可以直接用，这里不介绍logback细节，不是本次重点

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--  
配置说明

%6 表示字段最小长度为6，格式右对齐 		[	 右对齐 ]
%-64 表示最小长度64，并且左对齐		[左对齐	]
%6.32 表示最小长度6，最大长度32		[左对齐	]
-->
<configuration scan="true" scanPeriod="60 seconds"
	debug="false">
	<!-- 默认读取application.yml 配置 -->
	<springProperty scope="contex" name="app" source="spring.application.name"></springProperty>
	<springProperty scope="contex" name="log.path" source="log.path"></springProperty>
 	<!-- <property name="log.path" value="log" /> -->
	<property name="LOG_PATTERN"
		value="%d -%magenta(%6(${PID:- }))- [%-5(${app})] [%6thread] %-5level %cyan(%-64logger{128}) - %msg%n" />
		<!-- 时间	进程						 线程 					日志级别 				输出对象 			消息内容 -->
		<!-- 彩色 -->
		<!-- <property name="LOG_PATTERN" value="%d -%magenta(${PID:- })- [%16thread] %customcolor(%-5level) %cyan(%-64logger{128}) - %msg%n" /> -->
		<!-- 黑白日志 -->
		<!-- <property name="LOG_PATTERN" value="%d -%6(${PID:- })- [%16thread] %-5level %-64logger{128} - %msg%n" /> -->
	
	<!-- <contextName>${app}</contextName> -->
	<!--输出到控制台 -->
	<appender name="console"
		class="ch.qos.logback.core.ConsoleAppender">
		<!-- 级别过滤 -->
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<!-- 级别低于INFO 的日志不会被打印 -->
			<level>INFO</level>
		</filter>
		<!-- 打印模板 -->
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
			<!-- <pattern>%d -%magenta(${PID:- })- [%16thread] %customcolor(%-5level) 
				%cyan(%-64logger{128}) - %msg%n</pattern> -->
		</encoder>
	</appender>

	<!--输出到文件 -->
	<!-- all 日志文件，级别高于info 的全部打印 -->
	<appender name="all-file"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/${app}-all.log</file>
		<!-- ThresholdFilter级别过滤 级别大于 info 的才会输出 -->
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>INFO</level>
		</filter>
		<rollingPolicy
			class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/all/${app}-all-logging-%d{yyyy-MM-dd}.log
			</fileNamePattern>
			<!-- <fileNamePattern>${log.path}/spring-boot-${log.name}-logging.%d{yyyy-MM-dd}.all-log.zip</fileNamePattern> -->
			<!-- 日志保存周期 -->
			<maxHistory>30</maxHistory>
			<!-- 总大小 -->
			<totalSizeCap>1GB</totalSizeCap>
		</rollingPolicy>
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
		</encoder>
	</appender>

	<!-- info 日志文件 只打印info级别信息 -->
	<appender name="info-file"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/${app}-info.log</file>
		<!-- 只打印唯一种级别的日志 可以用 LevelFilter 配置 -->
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<level>INFO</level>
			<!--匹配到就允许 -->
			<onMatch>ACCEPT</onMatch>
			<!--没匹配到就禁止 -->
			<onMismatch>DENY</onMismatch>
		</filter>
		<rollingPolicy
			class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/info/${app}-info-logging-%d{yyyy-MM-dd}.log
			</fileNamePattern>
			<!-- 日志保存周期 -->
			<maxHistory>30</maxHistory>
			<!-- 总大小 -->
			<totalSizeCap>1GB</totalSizeCap>
		</rollingPolicy>
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
		</encoder>
	</appender>

	<!-- warn 日志文件 只打印warn级别信息 -->
	<appender name="warn-file"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/${app}-warn.log</file>
		<!-- 只打印唯一种级别的日志 可以用 LevelFilter 配置 -->
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<level>WARN</level>
			<!--匹配到就允许 -->
			<onMatch>ACCEPT</onMatch>
			<!--没匹配到就禁止 -->
			<onMismatch>DENY</onMismatch>
		</filter>
		<rollingPolicy
			class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/warn/${app}-warn-logging-%d{yyyy-MM-dd}.log
			</fileNamePattern>
			<!-- 日志保存周期 -->
			<maxHistory>30</maxHistory>
			<!-- 总大小 -->
			<totalSizeCap>1GB</totalSizeCap>
		</rollingPolicy>
		<!-- <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
			<fileNamePattern>${log.path}/spring-boot-${log.name}-logging.%d{yyyy-MM-dd}.warn-log.zip</fileNamePattern> 
			日志保存周期 <maxHistory>30</maxHistory> 总大小 <totalSizeCap>1GB</totalSizeCap> </rollingPolicy> -->
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
		</encoder>
	</appender>

	<!-- error 日志文件 只打印error级别信息 -->
	<appender name="error-file"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/${app}-error.log</file>
		<!-- <filter class="ch.qos.logback.classic.filter.LevelFilter"> <level>ERROR</level> 
			<onMatch>ACCEPT</onMatch> <onMismatch>DENY</onMismatch> </filter> -->
		<!--如果只是想要 Error 级别的日志，那么需要过滤一下，默认是 info 级别的，ThresholdFilter -->
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>Error</level>
		</filter>
		<rollingPolicy
			class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/error/${app}-error-logging-%d{yyyy-MM-dd}.log
			</fileNamePattern>
			<!-- 日志保存周期 -->
			<maxHistory>30</maxHistory>
			<!-- 总大小 -->
			<totalSizeCap>1GB</totalSizeCap>
		</rollingPolicy>
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
		</encoder>
	</appender>






	<!-- root 指定打印日志最低级别 -->
	<root level="info">
		<appender-ref ref="console" />
		<appender-ref ref="all-file" />
		<!-- <appender-ref ref="trace-file" /> -->
		<!-- <appender-ref ref="debug-file" /> -->
		<appender-ref ref="info-file" />
		<appender-ref ref="warn-file" />
		<appender-ref ref="error-file" />
	</root>

	<!-- logback为java中的包 -->
	<!-- <logger name="com.baiding"/> -->
	<!-- java中的包 -->
	<!-- <logger name="com.baiding" level="warn" addtivity="false"> <appender-ref 
		ref="console" /> </logger> -->


	<!-- <appender name="trace-file" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
		<file>${log.path}/spring-boot-${log.name}-trace.log</file> <filter class="ch.qos.logback.classic.filter.LevelFilter"> 
		<level>TRACE</level> <onMatch>ACCEPT</onMatch> <onMismatch>DENY</onMismatch> 
		</filter> <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
		<fileNamePattern>${log.path}/spring-boot-${log.name}-logging.%d{yyyy-MM-dd}.trace-log.zip</fileNamePattern> 
		日志保存周期 <maxHistory>30</maxHistory> 总大小 <totalSizeCap>1GB</totalSizeCap> </rollingPolicy> 
		<encoder> <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} 
		- %msg%n</pattern> </encoder> </appender> -->

	<!-- <appender name="debug-file" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
		<file>${log.path}/spring-boot-${log.name}-debug.log</file> <filter class="ch.qos.logback.classic.filter.LevelFilter"> 
		<level>DEBUG</level> <onMatch>ACCEPT</onMatch> <onMismatch>DENY</onMismatch> 
		</filter> <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
		<fileNamePattern>${log.path}/spring-boot-${log.name}-logging.%d{yyyy-MM-dd}.debug-log.zip</fileNamePattern> 
		日志保存周期 <maxHistory>30</maxHistory> 总大小 <totalSizeCap>1GB</totalSizeCap> </rollingPolicy> 
		<encoder> <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} 
		- %msg%n</pattern> </encoder> </appender> -->

</configuration>
```



### 启动类

```java
package com.wunanyu.cloud.config;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication  //springboot启动类注解
@EnableConfigServer  //SpringCloud-ServerConfig配置中心服务注解，表示这是配置服务中心
/**
 * @author Narule
 */
public class ConfigApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigApplication.class, args);
	}
	
}
```



启动这个类的main方法，看控制台console的打印日志是否启动成功，启动成功，则可以通过

localhost:8888/eureka-dev.yml 查看配置信息，只要配置地址有这个文件

### configserver 配置文件规则

```
/ { 应用名 } / { 环境名 } [ / { 分支名 } ]
/ { 应用名 } - { 环境名 }.yml
/ { 应用名 } - { 环境名 }.properties
/ { 分支名 } / { 应用名 } - { 环境名 }.yml
/ { 分支名 } / { 应用名 } - { 环境名 }.properties
```

### client 读取配置

如果一个springboot服务叫eureka,eureka配置如下

bootstrap.yml

```yml
spring:
  application:
    name: eureka #应用名
  cloud:
    config:
     uri: localhost:8888 #配置中心的访问地址 这不是配置文件的地址，是配置服务server-config的地址
     profile: dev #环境
     label: master #分支
management:
  endpoints:
    web:
      exposure:
        include:
        - "*"
        
eureka:
  server:
    renewal-percent-threshold: 0.45
    
log:
  path: log
```

那么他会读取的文件名称为`eureka-dev.yml`, git分支为master的文件(默认)



## Client

Client 其实只是在pom中添加 config依赖

`<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
</dependency>`

并且在bootstrap.yml 中配置server-config服务的url地址，Client就能自己读取到配置中心服务的数据

### pom依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.5.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.wunanyu</groupId>
	<artifactId>Eureka</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Eureka</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Hoxton.SR3</spring-cloud.version>
	</properties>

	<dependencies>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<!--Spring Boot Actuator，感应服务端变化  监控工具，可以刷新数据（需要自己主动调用接口）-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
		
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

### 配置文件

#### 	bootstrap.yml

```yml
spring:
  application:
    name: eureka
  cloud:
    config:
     uri: http://192.168.50.135:8888
     profile: dev
     label: master
management:
  endpoints:
    web:
      exposure:
        include:
        - "*"
        
eureka:
  server:
    renewal-percent-threshold: 0.45
    
log:
  path: log
```



#### 	logback-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--  
配置说明

%6 表示字段最小长度为6，格式右对齐 		[	 右对齐 ]
%-64 表示最小长度64，并且左对齐		[左对齐	]
%6.32 表示最小长度6，最大长度32		[左对齐	]
-->
<configuration scan="true" scanPeriod="60 seconds"
	debug="false">
	<!-- 默认读取application.yml 配置 -->
	<springProperty scope="contex" name="app" source="spring.application.name"></springProperty>
	<springProperty scope="contex" name="log.path" source="log.path"></springProperty>
 	<!-- <property name="log.path" value="log" /> -->
	<property name="LOG_PATTERN"
		value="%d -%magenta(%6(${PID:- }))- [%-5(${app})] [%6thread] %-5level %cyan(%-64logger{128}) - %msg%n" />
		<!-- 时间	进程						 线程 					日志级别 				输出对象 			消息内容 -->
		<!-- 彩色 -->
		<!-- <property name="LOG_PATTERN" value="%d -%magenta(${PID:- })- [%16thread] %customcolor(%-5level) %cyan(%-64logger{128}) - %msg%n" /> -->
		<!-- 黑白日志 -->
		<!-- <property name="LOG_PATTERN" value="%d -%6(${PID:- })- [%16thread] %-5level %-64logger{128} - %msg%n" /> -->
	
	<!-- <contextName>${app}</contextName> -->
	<!--输出到控制台 -->
	<appender name="console"
		class="ch.qos.logback.core.ConsoleAppender">
		<!-- 级别过滤 -->
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<!-- 级别低于INFO 的日志不会被打印 -->
			<level>INFO</level>
		</filter>
		<!-- 打印模板 -->
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
			<!-- <pattern>%d -%magenta(${PID:- })- [%16thread] %customcolor(%-5level) 
				%cyan(%-64logger{128}) - %msg%n</pattern> -->
		</encoder>
	</appender>

	<!--输出到文件 -->
	<!-- all 日志文件，级别高于info 的全部打印 -->
	<appender name="all-file"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/${app}-all.log</file>
		<!-- ThresholdFilter级别过滤 级别大于 info 的才会输出 -->
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>INFO</level>
		</filter>
		<rollingPolicy
			class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/all/${app}-all-logging-%d{yyyy-MM-dd}.log
			</fileNamePattern>
			<!-- <fileNamePattern>${log.path}/spring-boot-${log.name}-logging.%d{yyyy-MM-dd}.all-log.zip</fileNamePattern> -->
			<!-- 日志保存周期 -->
			<maxHistory>30</maxHistory>
			<!-- 总大小 -->
			<totalSizeCap>1GB</totalSizeCap>
		</rollingPolicy>
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
		</encoder>
	</appender>

	<!-- info 日志文件 只打印info级别信息 -->
	<appender name="info-file"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/${app}-info.log</file>
		<!-- 只打印唯一种级别的日志 可以用 LevelFilter 配置 -->
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<level>INFO</level>
			<!--匹配到就允许 -->
			<onMatch>ACCEPT</onMatch>
			<!--没匹配到就禁止 -->
			<onMismatch>DENY</onMismatch>
		</filter>
		<rollingPolicy
			class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/info/${app}-info-logging-%d{yyyy-MM-dd}.log
			</fileNamePattern>
			<!-- 日志保存周期 -->
			<maxHistory>30</maxHistory>
			<!-- 总大小 -->
			<totalSizeCap>1GB</totalSizeCap>
		</rollingPolicy>
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
		</encoder>
	</appender>

	<!-- warn 日志文件 只打印warn级别信息 -->
	<appender name="warn-file"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/${app}-warn.log</file>
		<!-- 只打印唯一种级别的日志 可以用 LevelFilter 配置 -->
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<level>WARN</level>
			<!--匹配到就允许 -->
			<onMatch>ACCEPT</onMatch>
			<!--没匹配到就禁止 -->
			<onMismatch>DENY</onMismatch>
		</filter>
		<rollingPolicy
			class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/warn/${app}-warn-logging-%d{yyyy-MM-dd}.log
			</fileNamePattern>
			<!-- 日志保存周期 -->
			<maxHistory>30</maxHistory>
			<!-- 总大小 -->
			<totalSizeCap>1GB</totalSizeCap>
		</rollingPolicy>
		<!-- <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
			<fileNamePattern>${log.path}/spring-boot-${log.name}-logging.%d{yyyy-MM-dd}.warn-log.zip</fileNamePattern> 
			日志保存周期 <maxHistory>30</maxHistory> 总大小 <totalSizeCap>1GB</totalSizeCap> </rollingPolicy> -->
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
		</encoder>
	</appender>

	<!-- error 日志文件 只打印error级别信息 -->
	<appender name="error-file"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/${app}-error.log</file>
		<!-- <filter class="ch.qos.logback.classic.filter.LevelFilter"> <level>ERROR</level> 
			<onMatch>ACCEPT</onMatch> <onMismatch>DENY</onMismatch> </filter> -->
		<!--如果只是想要 Error 级别的日志，那么需要过滤一下，默认是 info 级别的，ThresholdFilter -->
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>Error</level>
		</filter>
		<rollingPolicy
			class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/error/${app}-error-logging-%d{yyyy-MM-dd}.log
			</fileNamePattern>
			<!-- 日志保存周期 -->
			<maxHistory>30</maxHistory>
			<!-- 总大小 -->
			<totalSizeCap>1GB</totalSizeCap>
		</rollingPolicy>
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
		</encoder>
	</appender>






	<!-- root 指定打印日志最低级别 -->
	<root level="info">
		<appender-ref ref="console" />
		<appender-ref ref="all-file" />
		<!-- <appender-ref ref="trace-file" /> -->
		<!-- <appender-ref ref="debug-file" /> -->
		<appender-ref ref="info-file" />
		<appender-ref ref="warn-file" />
		<appender-ref ref="error-file" />
	</root>

	<!-- logback为java中的包 -->
	<!-- <logger name="com.baiding"/> -->
	<!-- java中的包 -->
	<!-- <logger name="com.baiding" level="warn" addtivity="false"> <appender-ref 
		ref="console" /> </logger> -->


	<!-- <appender name="trace-file" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
		<file>${log.path}/spring-boot-${log.name}-trace.log</file> <filter class="ch.qos.logback.classic.filter.LevelFilter"> 
		<level>TRACE</level> <onMatch>ACCEPT</onMatch> <onMismatch>DENY</onMismatch> 
		</filter> <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
		<fileNamePattern>${log.path}/spring-boot-${log.name}-logging.%d{yyyy-MM-dd}.trace-log.zip</fileNamePattern> 
		日志保存周期 <maxHistory>30</maxHistory> 总大小 <totalSizeCap>1GB</totalSizeCap> </rollingPolicy> 
		<encoder> <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} 
		- %msg%n</pattern> </encoder> </appender> -->

	<!-- <appender name="debug-file" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
		<file>${log.path}/spring-boot-${log.name}-debug.log</file> <filter class="ch.qos.logback.classic.filter.LevelFilter"> 
		<level>DEBUG</level> <onMatch>ACCEPT</onMatch> <onMismatch>DENY</onMismatch> 
		</filter> <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
		<fileNamePattern>${log.path}/spring-boot-${log.name}-logging.%d{yyyy-MM-dd}.debug-log.zip</fileNamePattern> 
		日志保存周期 <maxHistory>30</maxHistory> 总大小 <totalSizeCap>1GB</totalSizeCap> </rollingPolicy> 
		<encoder> <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} 
		- %msg%n</pattern> </encoder> </appender> -->

</configuration>
```

### 启动类

```java
package com.wunanyu.cloud.eureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class EurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaApplication.class, args);
	}

}

```
