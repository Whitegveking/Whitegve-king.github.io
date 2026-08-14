# Maven

## 概述

Maven是一款用于管理和构建Java项目的工具，是apache旗下的一个开源软件，它基于项目对象模型（POM）的概念，通过一小段描述信息（pom.xml）来管理项目的构建

### Maven的模型

<img width="1311" height="555" alt="Image" src="https://github.com/user-attachments/assets/350db351-9920-43c2-9a7b-2bd890472343" />

### Maven的仓库

Maven的仓库是用来存储和管理jar包的

Maven中有哪几类仓库？查找依赖（jar）的顺序是什么样的？

1. 本地仓库
2. 远程仓库
3. 中央仓库

### Maven的标准目录

<img width="350" height="330" alt="Image" src="https://github.com/user-attachments/assets/64e653f7-079f-4efe-9463-4857890076f2" />

### Maven坐标

- 是资源（jar）的**唯一标识**，通过该坐标可以**唯一定位资源位置**
- 使用坐标来**定义项目**或**引入项目中需要的依赖**

**由三个部分组成**：

- **groupId**：组织名称
- **artifactId**：模块名称
- **version**：版本号

---



## Maven的作用

### 依赖管理

方便快捷地管理项目依赖的资源（jar包）

<img width="438" height="201" alt="Image" src="https://github.com/user-attachments/assets/8ae87705-3184-4fa2-845e-447f19db22c7" />

描述jar包的信息，如版本，Maven就会自动联网下载该资源

### 项目构建

提供了一套标准化的跨平台自动化项目构建方式：

​	编译--->测试--->打包--->发布

### 统一项目结构

提供标准、统一的项目结构

## 依赖管理

### 依赖配置

依赖:指当前项目运行所需要的jar包，一个项目中可以引入多个依赖。
配置:

1. 在 pom.xml 中编写<dependencies>标签
2. 在<dependencies>标签中 使用<dependency>引入坐标
3. 定义坐标的 groupId, artifactId, version
4. 点击刷新按钮，引入最新加入的坐标

### 依赖排除

<exclusions>...</exclusions>

### 注意事项

- 一旦依赖配置变更，需要重新加载pom.xml
- 如果引入的依赖在本地仓库不存在，需要联网下载

## 生命周期

Maven的生命周期就是为了对所有的maven项目构建过程进行抽象和统一

Maven中有三套**相互独立**的生命周期

- clean：清理工作
- default：核心工作，如编译、测试、打包、安装、部署等
- site：生成报告、发布站点等

每套生命周期包含一些阶段（phase），阶段是有顺序的，后面的阶段依赖于前面的阶段

<img width="1077" height="507" alt="Image" src="https://github.com/user-attachments/assets/ea567533-086e-4838-a2cf-bbeefafe4db2" />

**重点**：

- clean：移除上一次构建生成的文件
- compile：编译项目源代码，将项目源文件编译为字节码文件
- test：使用合适的单元测试框架运行测试（junit）
- package：将编译后的文件打包，如jar、war等
- install：安装项目到本地仓库

> 在**同一套**生命周期中，当运行后面的阶段时，前面的阶段都会运行

生命周期基于插件完成，Maven的底层更相当于是一个插件执行框架，底层功能都是基于插件完成的

### 