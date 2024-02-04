---
title: "把React打包到Springboot的maven项目里"
description : "把React项目整合到Springboot项目里，使得最终只有一个jar包，包含前端和后端的所有构建"
date: 2023-12-10
tags : [
    "React",
    "Spring"
]
featuredImage: "https://s2.loli.net/2023/12/10/HA5NOofS8BcRxI7.png"
featuredImagePreview: "https://s2.loli.net/2023/12/10/HA5NOofS8BcRxI7.png"
draft: false
---
<!--more-->


> 我需要做一个简单的Spring Boot + React的前后端项目，考虑到前端的代码和内容不会过多，因此希望把react项目放到Spring Boot项目里，使得在构建项目的时候可以前后端一起打包。在网上找了几个教程，都不是特别满意，最后自己整理了一下，项目已经push到[Github](https://github.com/SroUngE/react-in-springboot)了。

## 环境准备  
* Maven 3.6+
* Java 17
* Node.js v18.17.0

## 初始化Spring Boot项目
### 1. 创建Spring Boot项目  
使用Spring Initializr来快速创建Spring Boot项目
![](https://s2.loli.net/2023/12/10/Cnbt7jUO8s2BaFz.png)

引入Spring Web的依赖，用于后端Web开发，引入Thymeleaf依赖，用于匹配前端页面
![](https://s2.loli.net/2023/12/10/PCYFmWy5AUl4ieH.png)

### 2. 安装并启动
```shell
mvn clean install
```
启动Spring Boot项目，此时访问`http://localhost:8080`，显示空白页面
![](https://s2.loli.net/2023/12/10/DbMprycFXZ3lvN4.png)

此时的目录结构如下：
```shell
├─src
│  ├─main
│  │  ├─java
```

## 初始化React项目
### 1. 创建React项目
在项目的`src/main/`目录下创建一个名为frontend的React项目
```shell
npx create-react-app frontend
```
### 2. 安装并启动
```shell
npm install
```
```shell
npm run build
```
```shell
npm run start
```
此时访问`http://localhost:3000`，显示React的主页
![](https://s2.loli.net/2023/12/10/Nn8DSmRh4xEHv9f.png)

此时的目录结构如下：
```shell
├─src
│  ├─main
│  │  ├─frontend
│  │  ├─java
```

## Maven插件打包React的build信息
### 1. 引入frontend-maven-plugin插件
这里需要使用到Github的插件：frontend-maven-plugin
```xml
<plugin>
    <groupId>com.github.eirslett</groupId>
    <artifactId>frontend-maven-plugin</artifactId>
    <version>1.15.0</version>
</plugin>
```

### 2. 打包React的build信息
1. 从远程仓库下载node.js的环境到本地的target文件夹下
2. 运行`npm install`指令安装React项目
3. 运行`npm run build`指令构建React项目
4. 将React项目的build文件夹复制到target/classes/static文件夹下

第1-3步由`frontend-maven-plugin`完成，配置如下：
```xml
<plugin>
    <groupId>com.github.eirslett</groupId>
    <artifactId>frontend-maven-plugin</artifactId>
    <version>1.15.0</version>
    <configuration>
        <nodeVersion>v18.17.0</nodeVersion>
        <!-- The working directory is where you've put package.json and your frontend configuration files -->
        <workingDirectory>src/main/frontend</workingDirectory>
        <!-- The installation directory is the folder where your node and npm are installed. -->
        <installDirectory>target</installDirectory>
    </configuration>

    <executions>
        <execution>
            <id>install node and npm</id>
            <goals>
                <goal>install-node-and-npm</goal>
            </goals>
        </execution>
        <execution>
            <id>running npm install</id>
            <goals>
                <goal>npm</goal>
            </goals>
        </execution>
        <execution>
            <id>running npm build</id>
            <goals>
                <goal>npm</goal>
            </goals>
            <configuration>
                <arguments>run build</arguments>
            </configuration>
        </execution>
    </executions>
</plugin>
```

第4步就是在maven的`process-resource`的生命周期执行的，配置如下：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <executions>
        <execution>
            <id>copy React builds into Spring Boot</id>
            <phase>process-resources</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <outputDirectory>target/classes/static</outputDirectory>
                <resources>
                    <resource>
                        <!-- the build folder to copy from -->
                        <directory>src/main/frontend/build</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

然后执行`mvn clean install`，首次运行会比较久，我跑了`3m27s`。结束后，`target/classes/static`目录有React的build文件，`target/node`目录有node环境。
![](https://s2.loli.net/2023/12/10/r4bxso6VPgzRq3J.png)


## 连接Spring Boot 和 React
### 1. 修改Spring Boot寻找页面的路径
默认情况下，thymeleaf会去`target/templates`下匹配页面，需要修改为`target/static`目录下，因此在`application.properties`文件里加上：
```properteis
spring.thymeleaf.prefix=classpath:/static/
```

### 2. 创建Controller
在Springboot项目里写一个接口来对接index页面：
```java
package com.example.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class WebController {

    @GetMapping("/")
    public String index() {
        return "index";
    }
}
```
### 3.修改React代理
因为最终这两个项目会被打包成一个jar包，在同一个server上启动，所以只需要把默认的react的代理设置为本地后端项目的端口即可。在`package.json`文件的末尾添加：
```json
"proxy": "http://locahost:8080"
```

## 大功告成
重新`mvn clean install`之后，启动Spring Boot，打开`http://locahost:8080`，显示：
![](https://s2.loli.net/2023/12/10/bwOWc4hdjC6kHNn.png)


## 进一步的优化
可以看到每次`mvn clean install`的时候都会需要安装一遍node，其实node只需要安装一次就可以了。因此可以创建一个property叫做`skipNodeInstall`，在clean阶段不clean node文件夹，然后跳过插件的`install node and npm` 和 `running npm install` 这两步。
