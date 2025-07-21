---
title: 'Maven、Gradle国内源配置'
author: ['kyle']
date: '2025-07-21T22:30:46+08:00'
#hidemeta: true
tags:
- Maven
- Gradle
- Java

keywords:
- Maven
- Gradle
- Java
- 国内源
---

# Maven国内源配置

## 修改Maven中的配置文件。
进入Mavne的安装路径也就是解压路径，例如`D:\apache-maven-3.5.4\conf`。修改`D:\apache-maven-3.5.4\conf`下的`settings.xml`文件

## 找到`<mirrors></mirrors>`标签，在其中配置阿里云镜像。
```xml
 <mirrors>
     <!--下面是配置内容-->
     <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
    </mirror>
  </mirrors>
```

```xml
<mirrors>
 
    <mirror>
        <id>aliyun-public</id>
        <mirrorOf>*</mirrorOf>
        <name>aliyun public</name>
        <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
```

```xml
  </mirrors>
    <profile>
      <repositories>
        <repository>
            <id>google</id>
            <url>https://maven.aliyun.com/repository/google</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
 
        <repository>
            <id>gradle-plugin</id>
            <url>https://maven.aliyun.com/repository/gradle-plugin</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
 
        <repository>
            <id>spring</id>
            <url>https://maven.aliyun.com/repository/spring</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
 
        <repository>
            <id>spring-plugin</id>
            <url>https://maven.aliyun.com/repository/spring-plugin</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
 
        <repository>
            <id>grails-core</id>
            <url>https://maven.aliyun.com/repository/grails-core</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
 
 
      </repositories>
    </profile>
```

# Gradle国内源配置
## 针对单个项目比较简单：
```gradle
repositories {
    maven {
        url 'https://maven.aliyun.com/repository/public/'
    }
    maven {
        url 'https://maven.aliyun.com/repository/spring/'
    }
    mavenLocal()
    mavenCentral()
}
```

## 针对全局项目
在GRADLE_HOME/init.d/目录下新建文件：init.gradle
```gradle
allprojects {
    repositories {
 
        mavenLocal()
 
        maven { url 'https://maven.aliyun.com/repository/public/' }
        maven { url 'https://maven.aliyun.com/repository/spring/'}
      maven { url 'https://maven.aliyun.com/repository/google/'}
      maven { url 'https://maven.aliyun.com/repository/gradle-plugin/'}
      maven { url 'https://maven.aliyun.com/repository/spring-plugin/'}
      maven { url 'https://maven.aliyun.com/repository/grails-core/'}
      maven { url 'https://maven.aliyun.com/repository/apache-snapshots/'}
        
        mavenCentral()
    }
}
```