---
title: Maven 基本使用
date: 2019-08-15 10:02:47
tags: [Web,Java,Maven]
categories: Java
---
# 概念
Maven是基于项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。Maven是跨平台的项目管理工具，主要服务于基于Java平台的项目构建，依赖管理和项目信息管理，方便jar包的管理且不需要上传至代码托管服务器。类似于iOS平台的 cocoapods 和 android 平台的 gradle。

# 安装 Maven for mac
## 下载 Maven
从 Maven 官方地址：http://maven.apache.org/download.cgi 下载最新版本 apache-maven-3.6.1-bin.tar.gz。
bin:存放了 maven 的命令，比如我们前面用到的 mvn tomcat:run
boot:存放了一些 maven 本身的引导程序，如类加载器等
conf:存放了 maven 的一些配置文件，如 setting.xml 文件 
lib:存放了 maven 本身运行所需的一些 jar 包
## 将 Maven 添加到环境变量
Maven 下载完毕后，解压到环境变量集合的位置，将其解压在 /usr/local/maven 目录下。 
然后在终端中，执行如下命令
```bash
open ~/.bash_profile
```
打开.bash_profile后，在里面添加如下的 maven 配置：
```bash
# Maven config
export M3_HOME=/usr/local/maven/apache-maven-3.6.1
export PATH=$M3_HOME/bin:$PATH
```
保存并关闭文件，然后执行以下命令使最新的环境变量生效：
```bash
source ~/.bash_profile
```
## 测试 Maven 是否安装成功
```bash
 echo $M3_HOME
 echo $PATH
```
若输出结果是类似以下的值则表明配置没有问题
```bash
/usr/local/maven/maven3.3.9
/usr/local/maven/maven3.3.9/bin:/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home/bin:/Users/qiudengqiang/.nvm/versions/node/v9.11.1/bin:/usr/local/opt/openssl/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/VMware Fusion.app/Contents/Public:/Users/qiudengqiang/.rvm/bin::/usr/local/mysql/bin
```
接下来用maven 的命令查看 maven 版本，鉴定Maven 环境是否安装成功。
```bash
mvn -version
```
成功时输入以下日志
```bash
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /usr/local/maven/apache-maven-3.6.1
Java version: 1.8.0_66, vendor: Oracle Corporation, runtime: /Library/Java/JavaVirtualMachines/jdk1.8.0_66.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.14.4", arch: "x86_64", family: "mac"
```
至此，maven for mac 环境就配置好了。

# Maven 仓库
maven 的工作需要从仓库下载一些 jar 包，假如本地的项目 A、项目 B 等都会通过 maven 软件从远程仓库(可以理解为互联网上的仓库)下载 jar 包并存在本地仓库，本地仓库 就是本地文件夹，当第二次需要此 jar 包时则不再从远程仓库下载，因为本地仓库已经存在了，可以将本地仓库理解为缓存，有了本地仓库就不用每次从远程仓库下载了。
## 本地仓库
本地仓库 :用来存储从远程仓库或中央仓库下载的插件和 jar 包，项目使用一些插件或 jar 包， 优先从本地仓库查找
仓库默认的目录在用户目录下：/Users/qiudengqiang/.m2/repository
## 远程仓库
远程仓库:如果本地需要插件或者 jar 包，本地仓库没有，默认去远程仓库下载。 远程仓库可以在互联网内也可以在公司的局域网内
## 中央仓库
在 maven 软件中内置一个远程仓库地址 http://repo1.maven.org/maven2 ，它是中央仓库，服务于整个互联网，它是由 Maven 团队自己维护，里面存储了非常全的 jar 包，它包含了世界上大部分流行的开源项目构件。

## 配置仓库路径
在与bin同级目录的conf/settings.xml
```config
 <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
```
在终端中输入:
```bash
mvn help:system
```
maven默认会从上面的服务器(中央仓库) 下载 jar包到本地。

## 修改 settings.xml
在mirrors中添加下面的内容，使用阿里云服务器下载jar包，因为国外的下载太慢了，把国外的注释不用
```bash
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
     <!-- 阿里云maven-->
    <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
    </mirror>
  </mirrors>
```
## Maven 项目结构
```bash
ProjectName
  |-src
  |   |-main
  |   |  |-java        —— 存放项目的.java文件
  |   |  |-resources   —— 存放项目资源文件，如spring, hibernate配置文件
  |   |-test
  |      |-java        ——存放所有测试.java文件，如JUnit测试类
  |      |-resources   —— 测试资源文件
  |-target             —— 目标文件输出位置例如.class、.jar、.war文件
  |-pom.xml           ——maven项目核心配置文件
```
「注」：如果是普通的 java 项目，那么就没有 webapp 目录。
# Maven常用命令
## mvn compile
完成编译操作
执行完毕后，会生成target目录，该目录中存放了编译后的字节码文件。
## mvn clean
执行完毕后，会将target目录删除。
## mvn test
完成单元测试操作
执行完毕后，会在target目录中生成三个文件夹：surefire、surefire-reports（测试报告）、test-classes（测试的字节码文件）
## mvn package
完成打包操作
执行完毕后，会在target目录中生成一个文件，该文件可能是jar、war（可以在配置文件中制定目标格式）
## mvn install 
执行 mvn install命令，完成将打好的jar包安装到本地仓库的操作
执行完毕后，会在本地仓库中出现安装后的jar包，方便其他工程引用
## mvn 组合命令
mvn clean compile
mvn clean test
mvn clean package
mvn clean install

# Maven 的核心概念
在平面几何中坐标（x,y）可以标识平面中唯一的一点。在maven中坐标就是为了定位一个唯一确定的jar包。我们需要找一个用来唯一标识一个构建的统一规范，拥有了统一规范，就可以把查找工作交给机器，Maven坐标主要组成(GAV) 确定一个jar在互联网位置。
groupId：定义当前Maven组织名称
artifactId：定义实际项目名称
version：定义当前项目的当前版本

## 坐标的查找
访问http://www.mvnrepository.com或者http://search.maven.org/网站
假设搜索所spring core,如图然后点击sping,接点选择所需要的版本，就能看到所需要的jar包了
![spring core](/images/tech/maven-spring-core.png)

## 依赖管理
### scope 依赖范围
![spring core](/images/tech/maven-dependency-scope.png)

其中依赖范围scope 用来控制依赖和编译，测试，运行的 classpath 的关系. 主要的是三种依赖关系如下：
- `compile` ： 默认编译依赖范围。对于编译，测试，运行三种classpath都有效
- `test` ：测试依赖范围。只对于测试classpath有效
- `provided` ：已提供依赖范围。对于编译，测试的classpath都有效，但对于运行无效。因为由容器已经提供，例如servlet-api
- `runtime` :运行时提供。例如:jdbc驱动

### 依赖传递
直接依赖和间接依赖
test2 依赖 test1，test3依赖test2
test2 直接依赖 test1，test3间接依赖test1
![transport dependency](/images/tech/maven-transport-dependency.png)
当第二依赖的范围是compile的时候，依赖可以传递
当第二直接依赖的范围是test的时候，依赖不会传递
provided和runtime一般很少用

### 依赖冲突
假如test1使用junit4.10依赖,并且scope是compile,那test2,test3都可以使用test1的junit4.10，因为传递下来了
假如test2使用junit4.9依赖，那test3会使用【就近的一个依赖】，也就是使用junit4.9

### 可选依赖
`<optional> true/false<optional>` 是否可选，也可以理解为是否向下传递。
在依赖中添加optional选项决定此依赖是否向下传递，如果是true则不传递，如果是false就传递，默认为false
![optional dependency](/images/tech/maven-optional-dependency.png)

### 排除依赖
exclusions可用于排除依赖,注意exclusions是写在dependency中
![optional dependency](/images/tech/maven-exclusions-dependency.png)

## 生命周期
Maven生命周期就是为了对所有的构建过程进行抽象和统一。
包括项目清理、初始化、编译、打包、测试、部署等几乎所有构建步骤。
生命周期可以理解为构建工程的步骤。

在Maven中有三套相互独立的生命周期，请注意这里说的是“三套”，而且“相互独立”，这三套生命周期分别是： 
Clean Lifecycle： 在进行真正的构建之前进行一些清理工作。 
Default Lifecycle： 构建的核心部分，编译，测试，打包，部署等等。 
Site Lifecycle： 生成项目报告，站点，发布站点。 

### Clean生命周期：清理项目
pre-clean 执行一些需要在clean之前完成的工作 
clean 移除所有上一次构建生成的文件 
post-clean 执行一些需要在clean之后立刻完成的工作 

也就是说，mvn clean 等同于 mvn pre-clean clean 
如果我们运行 mvn post-clean ，那么 pre-clean，clean 都会被运行。
这是Maven很重要的一个规则，可以大大简化命令行的输入。

### Default生命周期：构造项目
Default生命周期是Maven生命周期中最重要的一个，绝大部分工作都发生在这个生命周期中。这里，只解释一些比较重要和常用的阶段
```bash
validate 
generate-sources 
process-sources 
generate-resources 
process-resources 复制并处理资源文件，至目标目录，准备打包。 
compile 编译项目的源代码。 
process-classes 
generate-test-sources 
process-test-sources 
generate-test-resources 
process-test-resources 复制并处理资源文件，至目标测试目录。 
test-compile 编译测试源代码。 
process-test-classes 
test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。 
prepare-package 
package 接受编译好的代码，打包成可发布的格式，如 JAR 。 
pre-integration-test 
integration-test 
post-integration-test 
verify 
install 将包安装至本地仓库，以让其它项目依赖。 
deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享
```
运行任何一个阶段的时候，它前面的所有阶段都会被运行
这也就是为什么我们运行mvn install 的时候，代码会被编译，测试，打包，安装到本地仓库
此外，Maven的插件机制是完全依赖Maven的生命周期的，因此理解生命周期至关重要。

## Maven 工程的运行
进入 maven 工程目录(当前目录有 pom.xml 文件)，运行 tomcat:run 命令。
![maven-tomcat-run](/images/tech/maven-tomcat-run.png)
根据上边的提示信息，通过浏览器访问:http://localhost:8080/maven-helloworld/

## Maven 的概念模型
![maven-tomcat-run](/images/tech/maven-concept-model.png)
### 项目对象模型 (Project Object Model)
一个 maven 工程都有一个 pom.xml 文件，通过 pom.xml 文件定义项目的坐标、项目依赖、项目信息、 插件目标等。
### 依赖管理系统(Dependency Management System)
通过maven的依赖管理对项目所依赖的jar 包进行统一管理。
比如:项目依赖 junit4.9，通过在 pom.xml 中定义 junit4.9 的依赖即使用 junit4.9，如下所示是 junit4.9 的依赖定义:
```xml
<!-- 依赖关系 --> 
<dependencies>
    <!-- 此项目运行使用 junit，所以此项目依赖 junit --> 
    <dependency>
        <!-- junit 的项目名称 --> 
        <groupId>junit</groupId> 
        <!-- junit 的模块名称 --> 
        <artifactId>junit</artifactId> 
        <!-- junit 版本 --> 
        <version>4.9</version>
        <!-- 依赖范围:单元测试时使用 junit -->
        <scope>test</scope> 
    </dependency>
</dependencies>
```
### 一个项目生命周期(Project Lifecycle)
使用 maven 完成项目的构建，项目构建包括:清理、编译、测试、部署等过程，maven 将这些
过程规范为一个生命周期，如下所示是生命周期的各个阶段:
![maven-project-life-cycle](/images/tech/maven-project-lifecycle.png)
maven 通过执行一些简单命令即可实现上边生命周期的各各过程，比如执行 mvn compile 执行编译、
执行 mvn clean 执行清理。
### 一组标准集合
maven 将整个项目管理过程定义一组标准，比如:通过 maven 构建工程有标准的目录结构，有
标准的生命周期阶段、依赖管理有标准的坐标定义等。

### 插件(plugin)目标(goal)
maven 管理项目生命周期过程都是基于插件完成的。

# IDEA 开发 Maven 项目
在通常在开发的环境中，我们都会使用流行的工具来开发项目。
## IDEA 的 Maven 配置
![maven-idea-setting](/images/tech/maven-idea-setting.png)

## IDEA 中创建一个 Maven 的 web 工程
打开 idea，选择创建一个新工程，选择 idea 提供好的 maven 的 web 工程模板
![maven-idea-project](/images/tech/maven-idea-create-web-project.png)
点击 Next 填写项目信息
![maven-idea-project-gav](/images/tech/maven-idea-create-project-gav.png)
点击 Next，此处不做改动。
![maven-idea-mul](/images/tech/maven-idea-project-mul.png)
点击 Next 选择项目所在目录
![maven-idea-pp](/images/tech/maven-idea-create-pp.png)
点击 Finish 后开始创建工程，耐心等待，直到出现如下界面。
![maven-idea-project-catalogue](/images/tech/maven-idea-project-catalogue.png)
手动添加 src/main/java 目录，如下图右键 main 文件夹-New-Directory,创建一个新的文件夹命名为 java
![maven-idea-project-adddir](/images/tech/maven-idea-project-adddir.png)
点击 OK 后，在新的文件夹 java 上右键 Make Directory as Sources Root
![maven-idea-project-source-root](/images/tech/maven-idea-make-as-source-root.png)

### 创建一个 Servlet
src/java/main 创建了一个 Servlet，但报错
![maven-idea-project-servlet-error](/images/tech/maven-idea-servlet-error.png)
要解决问题，就是要将 servlet-api-xxx.jar 包放进来，作为 maven 工程应当添加 servlet 的坐标，从而 导入它的 jar

### 在 pom.xml 文件添加坐标
直接打开 hello_maven 工程的 pom.xml 文件，再添加坐标
![maven-idea-pom-add-gav](/images/tech/maven-idea-pom-add-gav.png)


添加 jar 包的坐标时，还可以指定这个 jar 包将来的作用范围。
每个 maven 工程都需要定义本工程的坐标，坐标是 maven 对 jar 包的身份定义，比如:入门程序的 坐标定义如下:
```xml
<!--项目名称，定义为组织名+项目名，类似包名--> 
<groupId>com.itheima</groupId>
<!-- 模块名称 --> 
<artifactId>hello_maven</artifactId>
<!-- 当前项目版本号，snapshot为快照版本即非正式版本，release为正式发布版本 -->
<version>0.0.1-SNAPSHOT</version> 

<packaging > :打包类型
    jar:执行 package 会打成 jar 包
    war:执行 package 会打成 war 包
pom :用于 maven 工程的继承，通常父工程设置为 pom
```

## 依赖范围
A 依赖 B，需要在 A 的 pom.xml 文件中添加 B 的坐标，添加坐标时需要指定依赖范围，依赖范围包括:

`compile`:编译范围，指 A 在编译时依赖 B，此范围为默认依赖范围。编译范围的依赖会用在 编译、测试、运行，由于运行时需要所以编译范围的依赖会被打包。
`provided`:provided依赖只有在当JDK或者一个容器已提供该依赖之后才使用， provided依 赖在编译和测试时需要，在运行时不需要，比如:servlet api 被 tomcat 容器提供。
runtime:runtime 依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如:jdbc 的驱动包。由于运行时需要所以 runtime 范围的依赖会被打包。
`test`:test 范围依赖 在编译和运行时都不需要，它们只有在测试编译和测试运行阶段可用， 比如:junit。由于运行时不需要所以 test 范围依赖不会被打包。
`system`:system 范围依赖与 provided 类似，但是你必须显式的提供一个对于本地系统中 JAR 文件的路径，需要指定 systemPath 磁盘路径，system 依赖不推荐使用。
![maven-dependency-scope](/images/tech/maven-depedency-scope02.png)

在 maven-web 工程中测试各个 scop。

### 测试总结
 默认引入 的 jar 包 ------- compile 【默认范围 可以不写】(编译、测试、运行 都有效 )
 servlet-api 、jsp-api ------- provided (编译、测试 有效， 运行时无效 防止和 tomcat 下 jar 冲突)  jdbc 驱动 jar 包 ---- runtime (测试、运行 有效 )
 junit ----- test (测试有效)
依赖范围由强到弱的顺序是:compile>provided>runtime>test


## 设置 JDK 编译版本
本教程使用 jdk1.8，需要设置编译版本为 1.8，这里需要使用 maven 的插件来设置:
在 pom.xml 中加入:
```xml
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
## 添加 tomcat7 插件
在 pom 文件中添加如下内容
![maven-pom-tomcat7](/images/tech/maven-pom-tomcat7.png)
此时点击 idea 最右侧 Maven Projects， 就可以看到我们新添加的 tomcat7 插件
双击 tomcat7 插件下 tomcat7:run 命令直接运行项目
![maven-pom-tomcat7](/images/tech/maven-idea-tomcat7-run.png)
也可以直接点击如图按钮，手动输入 tomc7:run 命令运行项目
![maven-pom-tomcat7](/images/tech/maven-idea-type-tomcat7-run.png)
点击后弹出如下图窗口
![maven-pom-tomcat7](/images/tech/maven-idea-tomcat7-run02.png)

# Maven 工程运行调试

## 端口占用处理
重新执行 tomcat:run 命令重启工程，重启之前需手动停止 tomcat，否则报下边的错误:
![maven-pom-tomcat7](/images/tech/maven-port-conflict.png)

## 断点调试
点击如图所示选项
![maven-idea-debug-edit](/images/tech/maven-idea-debug-edit.png)
在弹出框中点击如图加号按钮找到 maven 选项
![maven-idea-debug-option](/images/tech/maven-idea-debug-option.png)
在弹出窗口中填写如下信息
![maven-idea-debug-edit-fill](/images/tech/maven-idea-debug-edit-fill.png)
完成后先 Apply 再 OK 结束配置后，可以在主界面找到我们刚才配置的操作名称。
![maven-idea-debug-window-btn](/images/tech/maven-idea-debug-window-btn.png)
如上图红框选中的两个按钮，左侧是正常启动，右侧是 debug 启动。
## pom 文件描述

pom.xml 是 Maven 项目的核心配置文件，位于每个工程的根目录，基本配置如下:

`<project >` :文件的根节点 .
`<modelversion >` : pom.xml 使用的对象模型版本
`<groupId >` :项目名称，一般写项目的域名 
`<artifactId >` :模块名称，子项目名或模块名称 
`<version >` :产品的版本号 .
`<packaging >` :打包类型，一般有 jar、war、pom 等 
`<name >` :项目的显示名，常用于 Maven 生成的文档。
`<description >` :项目描述，常用于 Maven 生成的文档 
`<dependencies>` :项目依赖构件配置，配置项目依赖构件的坐标
`<build>` :项目构建配置，配置编译、运行插件等。