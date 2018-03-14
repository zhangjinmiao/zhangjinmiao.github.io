##1依赖配置
groupId,必选，实际隶属项目 (基本坐标)
artifactId,必选，其中的模块 (基本坐标)
version必选，版本号 (基本坐标)
type可选，依赖类型，默认jar
scope可选，依赖范围，默认compile
optional可选，标记依赖是否可选，默认false
exclusion可选，排除传递依赖性，默认空
大部分依赖声明只包含基本坐标。

##2依赖范围
maven项目有三种classpath（编译，测试，运行）
scope用来表示与classpath的关系，总共有五种
compile:编译，测试，运行
test:测试
provided:编译，测试
runtime:运行
system:编译，测试，同provided，但必须指定systemPath，慎用

依赖范围有效运行期：
![图片1.png](http://upload-images.jianshu.io/upload_images/626005-1638642b10d2d3d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###Maven依赖jar包冲突解决
####1、判断jar是否正确的被引用
1）、在项目启动时加上VM参数：-verbose:class
项目启动的时候会把所有加载的jar都打印出来 类似如下的信息：
classpath加载的jar

具体load的类

我们可以通过上面的信息查找对应的jar是否正确的被依赖，具体类加载情况，同时可以看到版本号，确定是否由于依赖冲突造成的jar引用不正确；

2）、 通过maven自带的工具：‍‍mvn dependency:tree

具体后面可以加 -Dverbose 参数 ，详细参数可以去自己搜，这里不详细介绍。

比如分析如下POM

运行： mvn dependency:tree -Dverbose

####2、冲突的解决

1）在pom.xml中引用的包中加入exclusion，排除依赖

```
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>dubbo</artifactId>  
    <version>2.5.3</version>  
    <exclusions>  
        <exclusion>  
            <artifactId>spring</artifactId>  
            <groupId>org.springframework</groupId>  
         </exclusion>  
     </exclusions>  
</dependency>  
```
去除全部依赖
```
<dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>2.5.3</version>
        <exclusions>
            <exclusion>
                <artifactId>*</artifactId>
                <groupId>*</groupId>
            </exclusion>
        </exclusions>
    </dependency>
```

2）在ide中右击进行处理，处理完后在pom.xml中也会添加exclusion元素
##3传递性依赖
Maven传递依赖的关系
![2.png](http://upload-images.jianshu.io/upload_images/626005-f2dce7666e190b83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##4可选依赖
有时候我们不想让依赖传递，那么可配置该依赖为可选依赖，将元素optional设置为true即可,例如：

```
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
    <optional>true</optional>
</dependency>
```
那么依赖该项目的另一项目将不会得到此依赖的传递。

##5排除依赖
当我们引入第三方jar包的时候，难免会引入传递性依赖，有些时候这是好事，然而有些时候我们不需要其中的一些传递性依赖。如当我们引入spring包时，我们不想引入传递性依赖commons-logging，我们可以使用exclusions元素声明排除依赖，exclusions可以包含一个或者多个exclusion子元素，因此可以排除一个或者多个传递性依赖。
```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.3.8.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

##6依赖归类
如果我们项目中用到很多关于Spring Framework的依赖，它们分别是org.springframework:spring-core:2.5.6, org.springframework:spring-beans:2.5.6,org.springframework:spring-context:2.5.6,它们都是来自同一项目的不同模块。因此，所有这些依赖的版本都是相同的，而且可以预见，如果将来需要升级Spring Framework，这些依赖的版本会一起升级。因此，我们应该在一个唯一的地方定义版本，并且在dependency声明引用这一版本，这一在Spring Framework升级的时候只需要修改一处即可。

```
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<springframework.version>4.3.8.RELEASE</springframework.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-beans</artifactId>
			<version>${springframework.version}</version>
		</dependency>
  </dependencies>
```

##7管理依赖
**mvn dependency:list**
表示依赖列表

**mvn dependency:tree**
表示依赖列表

**mvn dependency:analyze**
查找出在编译和测试中未使用但显示声明的依赖
Unused declared dependencies found:

##8仓库
###何为Maven仓库
在Maven世界中，任何一个依赖、插件或者项目构建的输出，都可以称为构件，任何一个构件都有一组坐标唯一标识。
以前传统的开放模式中，每个项目都有lib目录，各个项目lib目录下的内容存在大量的重复，这样不仅造成磁盘空间的浪费，而且也难 于管理。
Maven在某个统一的位置存储所有项目的共享的构件，这个统一的位置，我们就称之为仓库。（仓库就是存放依赖和插件的地方）

###仓库的布局
任何一个构件都有其唯一的坐标，根据这个坐标可以定义其在仓库中的唯一存储路径，这便是Maven的仓库布局方式。
该路径与坐标的大致对应关系为：
groupId/artifactId/version/artifactId-version.packaging

###仓库的分类
对于Maven来说，仓库只分为两类：本地仓库和远程仓库。
当Maven根据坐标寻找构件的时候，它首先会查看本地仓库，如果本地仓库存在此构件，则直接使用；如果本地仓库不存在此构件，Maven就会去远程仓库查找，发现需要的构件，下载到本地仓库再使用。如果本地仓库和远程仓库都没有，Maven就会报错。(孝明上次那个问题)
私服是一种特殊的远程仓库，为了节省带宽和时间，应该在局域网内架设一个私有的仓库服务器，用其代理所有外部的远程仓库。内部的项目还能部署到私服上供其他项目使用。

![图片2.png](http://upload-images.jianshu.io/upload_images/626005-ab1ef215a72227b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####本地仓库
一般来说，在Maven项目目录下，没有诸如lib/这样用来存放依赖文件的目录。当Maven在执行编译或者测试时，如果需要使用依赖文件，它总是基于坐标使用本地仓库的依赖文件。
默认情况下，本地仓库地址为C:\Users\用户名\.m2\repository\，有时候，因为某些原因(C盘空间不够，系统盘)，需要自定义本地仓库的目录地址，这时，可以编辑Maven目录下settings.xml，设置localRepository元素的值为想要的仓库地址。例如：
![image.png](http://upload-images.jianshu.io/upload_images/626005-41a56a38082ac9bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一个构件只有在本地仓库中之后，才能由其他Maven项目使用。首先必须得以构件并安装到本地仓库中。在项目中执行mvn clean install 命令，Install插件的install目标将项目的构件输出文件安装到本地仓库。

####远程仓库
当用户执行maven命令之后，Maven会根据配置和需要从远程仓库下载构件到本地仓库。

这好比藏书，例如我要读<红楼梦>，会先检查自己的书房是否藏了这本书，如果发现没有这本书，于是就跑去书店买一本回来，放在书房里。

本地仓库就好比书房，我需要读书的时候先从书房找，相应的，Maven需要构件的时候，先从本地仓库找。当无法从本地仓库找到需要的构件的时候，就会从远程仓库下载构件到本地仓库。

####私服
私服是一种特殊的远程仓库，它是架设在局域网内的仓库服务，私服代理广域网上的远程仓库，供局域网内的用户使用。当Maven需要下载构件的时候，它从私服请求，如果 私服上不存在该构件，则从外部远程仓库下载，缓存在私服上之后，再为Maven的下载请求提供服务。

私服的好处：
1、节省自己的外网带宽
2、加速Maven构建
3、部署自己内部的第三方构件
4、提高稳定性，增强控制
5、降低中央仓库的负荷。

####镜像
- 如果仓库X可以提供仓库Y存储的所有内容，那么就可以认为X是Y的一个镜像。如果配置X的镜像，那么对于仓库Y的任何请求都会转至X仓库。
- 镜像常见的用法是结合私服，由于私服可以代理任何外部的公共仓库，因此，对于组织内部的Maven用户来说，使用一个私服地址就等于使用了所有需要的外部仓库。任何需要的构件都可以从私服获得，私服就是所有仓库的镜像。这时可以配置这样的一个镜像，见下图。
![image.png](http://upload-images.jianshu.io/upload_images/626005-831bb1a4de25398b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 该例中<mirrorOf>的值为星号，表示该配置是所有Maven仓库的镜像，任何对于远程仓库的请求都会转至http://172.168.10.100:8081/nexus/content/groups/public/ 如果镜像仓库需要认证，则配置一个ID为nexus的<server>即可。

- 需要注意：由于镜像仓库完全屏蔽了被镜像的仓库，当镜像仓库不稳定或者停止服务的时候，Maven将无法访问镜像仓库，因此将无法下载构件。

##9生命周期

###何为生命周期
Maven的生命周期就是为了对所有的构建过程进行抽象和统一，这个生命周期包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有的构建步骤。

###Maven的三大生命周期
Maven有三套相互独立的生命周期，请注意这里说的是"三套"，而且"相互独立"，这三套生命周期分别是：
1、Clean Lifecycle 在进行真正的构建之前进行一些清理工作。
2、Default Lifecycle 构建的核心部分，编译，测试，打包，部署等等。
3、Site Lifecycle 生成项目报告，站点，发布站点。
每个生命周期包含一些阶段(phase)，这些阶段是有顺序的，并且后面的阶段依赖于前面的阶段。以clean生命周期为例，它包含的阶段有pre-clean，clean和post-clean。当用户调用pre-clean的时候，只有pre-clean阶段得以执行。当用户调用clean时候，pre-clean和clean阶段会得以顺序执行。

####clean生命周期
Clean生命周期的目的是清理项目，它包含三个阶段：
1、pre-clean 执行一些清理前需要准备完成的工作
2、clean 清理上一次构建生成的文件
3、post-clean 执行一些清理后需要完成的工作。

####default生命周期
Default生命周期定义了真正构建时所需要执行的所有步骤，它是所有生命周期中最核心的部分，这里，只解释一些比较重要和常用的阶段：
Validate 验证项目是否正确且所有必要的信息都可用。
generate-sources 为包含在编译范围内的代码生成源代码
process-sources 处理源代码，如过滤值。
generate-resources 生成所有需要在打包过程中的资源文件。
**process-resources** 复制并处理资源文件，至目标目录，准备打包。
**compile **编译项目的源代码。
process-classes 为编译生成的文件做后期工作。
generate-test-sources 为编译内容生成测试源代码
process-test-sources 处理测试源代码。
generate-test-resources  生成测试需要的资源文件。
**process-test-resources **复制并处理资源文件，至目标测试目录。
**test-compile** 编译测试源代码。
process-test-classes 对测试编译生成的文件做后期处理。
**test **使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
prepare-package 在真正打包之前，执行一些打包的必要操作。
**package** 接受编译好的代码，打包成可发布的格式，如 JAR 。
pre-integration-test 执行一些在集成测试运行之前需要的动作。
integration-test 如果有必要，处理包并发布至集成测试环境。
post-integration-test 执行一些在集成测试运行之后需要的动作。
Verify 执行所有的检查，验证包是否有效，质量是否合格。
**install** 将包安装至本地仓库，以让其它项目依赖。
**deploy **将最终的包复制到远程的仓库，以让其它开发人员与项目共享。

####site生命周期
Site生命周期的目的是建立和发布项目站点，Maven能够基于POM所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息。该生命周期包含如下阶段：
1. Pre-site 执行一些在生成项目站点发布到服务器上。
2. Site 生成项目站点文档。
3. Post-site 执行一些在生成项目站点之后需要完成的工作。
4. Site-deploy 将生命的项目

