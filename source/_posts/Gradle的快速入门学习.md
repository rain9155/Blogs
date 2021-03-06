---
title: Gradle的快速入门学习
tags: gradle
Categories: grade
date: 2020-06-26 22:36:22
categories:
---


## 前言

Gradle是一个灵活和高效自动化构建工具，它的构建脚本采用Groovy或kotlin语言编写，Groovy或Kotlin都是基于JVM的语言，它们的语法和java的语法有很多的类似并且兼容java的语法，所以对于java开发者，只需很少的学习成本就能快速上手Gradle开发，同时Gradle也是Android官方的构建工具，学习Gradle，能够帮助我们更好的了解Android项目的构建过程，当项目构建出现问题时，我们也能更好的排查问题，所以Gradle的学习能帮助我们更好的管理Android项目，Gradle的官方地址如下：

[Gradle官网](https://docs.gradle.org/current/userguide/userguide.html)

[Github地址](https://github.com/gradle/gradle)

## Gradle的特点

1、Gradle构建脚本采用Groovy或Kotlin语言编写，如果采用Groovy编写，构建脚本后缀为.gradle，在里面可以使用Groovy语法，如果采用Kotlin编写，构建脚本后缀为.gradle.kts，在里面可以使用Kotlin语法；

2、因为Groovy或Kotlin都是面向对象语言，所以在Gradle中处处皆对象，Gradle的.gradle或.gradle.kts脚本本质上是一个Project对象，在脚本中一些带名字的配置项如buildscript、allprojects等本质上就是对象中的方法，而配置项后面的闭包{}就是参数，所以我们在使用这个配置项时本质上是在调用对象中的一个方法；

3、在Groovy或Kotlin中，函数和类一样都是一等公民，它们都提供了很好的闭包{}支持，所以它们很容易的编写出具有[DSL](https://zh.wikipedia.org/wiki/%E9%A2%86%E5%9F%9F%E7%89%B9%E5%AE%9A%E8%AF%AD%E8%A8%80)风格的代码，用DSL编写构建脚本的Gradle比其他采用xml编写构建脚本的构建工具如maven、Ant等的可读性更强，动态性更好，整体更简洁；

4、Gradle中主要有Project和Task对象，Project是Gradle中构建脚本的表示，一个构建脚本对应一个Project对象，Task是Gradle中最小的执行单元，它表示一个独立的任务，Project为Task提供了执行的上下文。

## Groovy基础入门

本文的所有示例都是采用Groovy语言编写，在阅读本文前先简单的入门Groovy：

[Groovy 使用完全解析](https://blog.csdn.net/zhaoyanjun6/article/details/70313790)

下面主要讲Groovy与java的主要区别：

1、Groovy语句后面的分号可以忽略

```groovy
int num1 = 1
int num2 = 2
int result = add(num1, num2)

int add(int a, int b){
    return a + b
}
```

2、Groovy支持动态类型推导，使用def来定义变量和方法时可以不指定变量的类型和方法的返回值类型，同时定义方法时参数可以不指定类型

```groovy
def num1 = 1
def num2 = 2
def result = add(num1, num2)

def add(a, b){
    return a + b
}
```

3、Groovy的方法调用传参时可以不添加括号，方法不指定return语句时，最后一行默认为返回值

```groovy
def result = add 1, 2

def add(a, b){
  a + b
}
```

4、Groovy可以用单、双、三引号来表示字符串，其中单引号表示普通字符串，双引号表示的字符串可以使用取值运算符**${}**，而**$**在单引号只只是表示一个普通的字符，三引号表示的字符串又称为模版字符串，它可以保留文本的换行和缩进格式，三引号同样不支持**$**

```groovy
def world = 'world'

def str1 = 'hello ${world}'
def str2 = "hello ${world}"
def str3 = '''hello
&{world}'''

//打印输出：
//hello ${world}
//hello world
//hello
//&{world}
```

5、Groovy会为类中每个没有可见性修饰符的字段生成get/set方法，我们访问这个字段其实是调用它的get/set方法，同时如果类中的方法以get/set开头，我们也可以像普通字段一样访问这个方法

```groovy
class Person{
    def name
    private def country
      
    def setNation(nation){
        this.country = nation
    }
    def getNation(){
        return country
    }
}

def person = new Person()
//访问字段
person.name = 'rain9155'
println person.name
//像字段一样访问这个以get/set开头的方法
person.nation = "china"
println person.nation
```

6、Groovy中的闭包是用**{参数列表 -> 代码体}**表示的代码块，当参数 <= 1个时，**->**箭头可以省略，同时当只有一个参数时，如果不指定参数名默认以**it**为参数名，在Groovy中闭包对应的类型是[Closure](https://groovy-lang.org/closures.html)，所以闭包可以作为参数传递，同时闭包有一个delegate字段,  通过delegate可以把闭包中的执行代码委托给任意对象来执行

```groovy
class Person{
    def name
    def age
}

def person = new Person()
//定义一个闭包
def closure = {
    name = 'rain9155'
    age = 21
}
//把闭包委托给person对象执行
closure.delegate = person
//执行闭包，或者调用closure.call()也可以执行闭包
closure()

//以上代码把闭包委托给person对象执行，所以闭包就执行时就处于person对象上下文
//所以闭包就可以访问到person对象中的name和age字段，完成赋值
println person.name//输出：rain9155
println person.age//输出：21
```

在Gradle中很多地方都使用了闭包的委托机制，通过闭包完成一些特定对象的配置，在Gradle中，如果你没有指定闭包的delegate，delegate默认为当前项目的Project对象。

以上Groovy知识，认为对于有java基础的人来说，用于学习Gradle足够了，当然对于一些集合操作、文件操作等，等以后使用到时可以到[Groovy官网](http://www.groovy-lang.org/single-page-documentation.html)来查漏补缺。

## Gradle的安装配置

官方教程：[Installing Gradle](https://gradle.org/install/)

安装Gradle前需要确保你的电脑已经配置好JDK，JDK的版本要求是8或更高，可以通过包管理器自动安装Gradle或手动下载Gradle安装两种方式，在Window平台下我推荐使用手动安装，在Mac平台下我推荐使用Homebrew包管理器自动安装：

### 1、Window平台

* 1、在Gradle的[安装页面](https://gradle.org/releases/)选择一个Gradle版本，下载它的binary-only或complete版本，binary-only版本表示下载的Gradle压缩包只包含Gradle的源码，complete版本表示下载的Gradle压缩包包含Gradle的源码和源码文档说明，这里我下载了gradle-6.5-bin版本；
* 2、下载好Gradle后，把它解压到特定目录，如我这里解压到：D:/gradle，然后像配置java环境一样把D:/gradle/gradle-6.5/bin路径添加到系统的PATH变量下；
* 3、打开cmd，输入`gradle -v`校验是否配置成功，输出以下类似信息则配置成功.

```groovy
C:\Users\HY>gradle -v

------------------------------------------------------------
Gradle 6.5
------------------------------------------------------------

Build time:   2020-06-02 20:46:21 UTC
Revision:     a27f41e4ae5e8a41ab9b19f8dd6d86d7b384dad4

Kotlin:       1.3.72
Groovy:       2.5.11
Ant:          Apache Ant(TM) version 1.10.7 compiled on September 1 2019
JVM:          10.0.2 ("Oracle Corporation" 10.0.2+13)
OS:           Windows 10 10.0 amd64
```

### 2、Mac平台

* 1、安装[Homebrew](https://brew.sh/index_zh-cn)；
* 2、打开终端，输入`brew install gradle`，它默认会下载安装binary-only版本;
* 3、当Homebrew安装Gradle完成后，在终端输入`gradle -v`校验是否安装成功.

## Gradle的项目结构

Gradle项目可以使用Android Studio、IntelliJ IDEA等IDE工具或文本编辑器来编写，这里我以Mac平台为例采用文本编辑器配合命令行来编写，Window平台类似。

新建一个目录，如我这里为：～/GradleDemo，打开命令行，输入`cd ~/GradleDemo`切换到这个目录，然后输入`gradle init`，接着gradle会执行`init`这个task任务，它会让你选择生成的项目模版、编写脚本的语言、项目名称等，我选择了basic模版(即原始的Gradle项目)、Groovy语言、项目名称为GradleDemo，如下：

{% asset_img gradle1.png gradle1 %}

这样会`init`任务就会自动替你生成相应的项目模版，如下：

{% asset_img gradle2.png gradle1 %}

忽略掉以**.**开头的隐藏文件或目录，gradle init为我们自动生成了以下文件或目录：

### 1、build.gradle

它表示Gradle的项目构建脚本，在里面我们可以通过Groovy来编写脚本，在Gradle中，一个build.gradle就对应一个项目，build.gradle放在Gradle项目的根目录下，表示它对应的是根项目，build.gradle放在Gradle项目的其他子目录下，表示它对应的是子项目，Gradle构建时会把build.gradle解析成[Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project)对象，你在里面编写的DSL，其实就是Project接口中的方法。

### 2、settings.gradle

它表示Gradle的多项目配置脚本，存放在Gradle项目的根目录下，在里面可以通过include来决定哪些子项目会参与构建，Gradle构建时会把settings.gradle解析成[Settings](https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html#org.gradle.api.initialization.Settings)对象，include也只是Settings接口中的一个方法。

### 3、Gradle Wrapper

`gradle init`执行时会同时执行`wrapper`任务，`wrapper`任务会创建gradle/wrapper目录，并创建gradle/wrapper目录下的gradle-wrapper.jar、gradle-wrapper.properties这两个文件，还同时创建gradlew、gradlew.bat这两个脚本，它们统称为Gradle Wrapper，是对Gradle的一层包装。

Gradle Wrapper的作用就是可以让你的电脑在**不安装配置Gradle环境**的前提下运行Gradle项目，例如当你的Gradle项目被用户A clone下来时，而用户A的电脑上没有安装配置Gradle环境，用户A通过Gradle构建项目时，Gradle Wrapper就会从指定下载位置下载Gradle，并解压到电脑的指定位置，然后用户A就可以在不配置Gradle系统变量的前提下在Gradle项目的命令行中运行gradlew或gradlew.bat脚本来使用gradle命令，假设用户A要运行`gradle -v`命令，在linux平台下只需要运行`./gradle -v`，在window平台下只需要运行`gradlew -v`，只是把`gradle`替换成`gradlew`。

Gradle Wrapper的每个文件含义如下：

**1、gradlew**：用于在linux平台下执行gradle命令的脚本；

**2、gradlew.bat**：用于在window平台下执行gradle命令的脚本；

**3、gradle-wrapper.jar**：包含Gradle Wrapper运行时的逻辑代码；

**4、gradle-wrapper.properties**：用于指定Gradle的下载位置和解压位置；

gradle-wrapper.properties中各个字段解释如下：

| 字段名           | 解释                                                         |
| ---------------- | ------------------------------------------------------------ |
| distributionBase | 下载的Gradle的压缩包解压后的主目录，为GRADLE_USER_HOME，在window中它表示**C:/用户/你电脑登录的用户名/.gradle/**，在mac中它表示**～/.gradle/** |
| distributionPath | 相对于distributionBase的解压后的Gradle的路径，为wrapper/dists |
| distributionUrl  | Grade压缩包的下载地址，在这里可以修改下载的Gradle的版本和版本类型(binary或complete)，例如gradle-6.5-all.zip表示Gradle 6.5的complete版本，gradle-6.5-bin.zip表示Gradle 6.5的binary版本 |
| zipStoreBase     | 同distributionBase，不过是表示存放下载的Gradle的压缩包的主目录 |
| zipStorePath     | 同distributionPath，不过是表示存放下载的Gradle的压缩包的路径 |

使用Gradle Wrapper后，就可以统一项目在不同用户电脑上的Gradle版本，同时不必让运行这个Gradle项目的人安装配置Gradle环境，提高了开发效率。

## Gradle的多项目配置

现在我们创建的Gradle项目默认已经有一个根项目了，它的build.gradle文件就处于Gradle项目的根目录下，如果我们想要添加多个子项目，这时就需要通过settings.gradle进行配置。

首先我们在GradleDemo中创建多个文件夹，这里我创建了4个文件夹，分别为：subproject_1、subproject_2，subproject_3，subproject_4，然后在每个新建的文件夹下创建build.gradle文件，如下：

{% asset_img gradle3.png gradle1 %}

接着打开settings.gradle，添加如下：

```groovy
include ':subproject_1', ':subproject_2', ':subproject_3', ':subproject_4'
```

这样就完成了子项目的添加，打开命令行，切换到GradleDemo目录处，输入`gradle projects`，执行projects任务展示所有项目之间的依赖关系，如下：

```groovy
# chenjianyu @ FVFCG2HJL414 in ~/GradleDemo 
$ gradle projects     

> Task :projects
Root project 'GradleDemo'
+--- Project ':subproject_1'
+--- Project ':subproject_2'
+--- Project ':subproject_3'
\--- Project ':subproject_4'

BUILD SUCCESSFUL in 540ms
1 actionable task: 1 executed
```

可以看到，4个子项目依赖于根项目，接下来我们来配置项目，配置项目一般在当前项目的build.gradle中进行，可以通过buildscript方法配置，但是如果有多个项目时，而每个项目的某些配置又一样，那么在每个项目的build.gradle进行相同的配置是很浪费时间，而Gradle的Project接口为我们提供了allprojects和subprojects方法，在根项目的build.gradle中使用这两个方法可以全局的为所有子项目进行配置，allprojects和subprojects的区别是：**allprojects的配置包括根项目而subprojects的配置不包括根项目**，例如:

```groovy
//根项目的build.gradle

//为所有项目添加maven repo地址
allprojects {
    repositories {
        mavenCentral()
    }
}
//为所有子项目添加groovy插件
subprojects {
    apply plugin: 'groovy'
}
```

## Gradle构建的生命周期

当在命令行输入`gradle build`构建整个项目或`gradle task名称`执行某个任务时就会进行Gradle的构建，它的构建过程分为3个阶段：

**init(初始化阶段) -> configure(配置阶段) -> execute(执行阶段)**

* **init**：初始化阶段主要是解析settings.gradle，生成Settings对象，确定哪些项目需要参与构建，为需要构建的项目创建Project对象；
* **configure**：配置阶段主要是解析build.gradle，配置init阶段生成的Project对象，构建根项目和所有子项目，同时生成和配置在build.gradle中定义的Task对象，构造Task的关系依赖图，关系依赖图是一个有向无环图；
* **execute**：根据configure阶段的关系依赖图执行Task.

Gradle在上面3个阶段中每一个阶段的开始和结束都会hook一些监听，暴露给开发者使用，方便开发者在Gradle的不同生命周期阶段做一些事情。

settings.gradle和build.gradle分别代表Settings对象和Project对象，它们都有一个[Gradle](https://docs.gradle.org/current/dsl/org.gradle.api.invocation.Gradle.html#org.gradle.api.invocation.Gradle)对象，我们可以在Gradle项目根目录的settings.gradle或build.gradle中获取到Gradle对象，然后进行生命周期监听，如下：

```groovy
//build.gradle或settings.gradle

this.gradle.buildStarted {
    println "Gradle构建开始"
}

//---------------------init开始--------------------------------
this.gradle.settingsEvaluated {
    println "settings.gradle解析完成"
}
this.gradle.projectsLoaded {
    println "所有项目从settings加载完成"
}
//---------------------init结束--------------------------------
  
//-------------------configure开始-----------------------------
this.gradle.beforeProject {project ->
    //每一个项目构建之前被调用
    println "${project.name}项目开始构建"
}
this.gradle.afterProject {project ->
    //每一个项目构建完成被调用
    println "${project.name}项目构建完成"
}
this.gradle.projectsEvaluated {
    println "所有项目构建完成"
}
this.gradle.taskGraph.whenReady {
    println("task图构建完成")
}
//-------------------configure结束-----------------------------

//-------------------execute开始-----------------------------
this.gradle.taskGraph.beforeTask {task ->
    //每个task开始执行时会调用这个方法
    println("${task.name}task开始执行")
}
this.gradle.taskGraph.afterTask {task ->
    //每个task执行结束时会调用这个方法
    println("${task.name}task执行完成")
}
//-------------------execute结束-----------------------------

this.gradle.buildFinished {
    println "Gradle构建结束"
}
```

上面监听方法的放置顺序就是整个Gradle构建的顺序，但是要注意的是Gradle的buildStarted方法永远不会被回调，因为我们注册监听的时机太晚了，当解析settings.gradle或build.gradle时，Gradle就已经构建开始了，所以这个方法也被Gradle标记为废弃的了，因为我们没有机会监听到Gradle构建开始，同时如果你是在build.gradle中添加上面的所有监听，那么Gradle的settingsEvaluated和projectsLoaded方法也不会被回调，因为settings.gradle的解析是在build.gradle之前，在build.gradle中监听这两个方法的时机也太晚了。

> 也可以通过Gradle对象的addBuildListener方法添加[BuildListener](https://docs.gradle.org/current/javadoc/org/gradle/BuildListener.html)来监听Gradle的整个生命周期回调

上面是监听整个Gradle构建的生命周期回调监听，那么我怎么监听我的当前单个项目的构建开始和结束呢？只需要在你当前项目的build.gradle中添加：

```groovy
//build.gradle

this.beforeEvaluate {
    println '项目开始构建'
}
this.afterEvaluate {
    println '项目构建结束'
}
```

但是要注意的是在根项目的build.gradle添加上述方法，其beforeEvaluate方法是无法被回调的，因为注册时机太晚，解析根项目的的build.gradle时根项目已经开始构建了，但是子项目的build.gradle添加上述方法是可以监听到项目构建的开始和结束，因为根项目构建完成后才会轮到子项目的构建。

## Task

### 1、Task的创建

[Task](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html#org.gradle.api.Task)是Gradle中最小执行单元，它是一个接口，默认实现类为[DefaultTask](https://docs.gradle.org/current/dsl/org.gradle.api.DefaultTask.html#org.gradle.api.DefaultTask)，在Project中提供了task方法来创建Task，所以Task的创建必须要处于Project上下文中，这里我在subproject_1/build.gradle中创建Task，如下：

```groovy
//subproject_1/build.gradle

//通过Project的task方法创建一个Task
task task1{
	doFirst{
		println 'one'
	}
	doLast{
 		println 'two'
 	}
}
```

上述代码通过task方法创建了一个名为task1的Task，在创建Task的同时可以通过闭包配置它的doFirst和doLast动作，doFirst和doLast都是Task中的方法，其中doFirst方法会在Task的action执行前执行，doLast方法会在Task的action执行后执行，而action就是Task的执行单元，在后面自定义Task会介绍到，除此之外还可以在创建Task之后再指定它的doFirst和doLast动作，如下：

```groovy
//subproject_1/build.gradle

//通过Project的task方法创建一个Task
def t = task task2
t.doFirst {
	println 'one'
}
t.doLast{
	println 'two'
}
```

上面通过Project的task方法创建的Task默认被放在Project的[TaskContainer](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.TaskContainer.html#org.gradle.api.tasks.TaskContainer)类型的容器中，我们可以通过Project的getTasks方法获取到这个容器，而TaskContainer提供了create方法来创建Task，如下：

```groovy
//subproject_1/build.gradle

//通过TaskContainer的create方法创建一个Task
tasks.create(name: 'task3'){
	doFirst{
		println 'one'
	}
	doLast{
 		println 'two'
 	}
}
```

以上就是创建Task最常用的几种方式，创建Task之后，就可以执行它，执行一个Task只需要把task名称接在`gradle`命令后，如下我在命令行输入`gradle task1`执行了task1:

```groovy
# chenjianyu @ FVFCG2HJL414 in ~/GradleDemo 
$ gradle task1   

> Task :subproject_1:task1
one
two

BUILD SUCCESSFUL in 463ms
1 actionable task: 1 executed
```

如果你想精简输出的信息，只需要添加`-q`参数，如：`gradle -q task1`，这样输出就只包含task1的输出：

```groovy
# chenjianyu @ FVFCG2HJL414 in ~/GradleDemo 
$ gradle -q task1
one
two
```

如果要执行多个Task，多个task名称接在`gradle`命令后用空格隔开就行，如：`gradle task1 task2 task3`。

### 2、Task的属性配置

Gradle为每个Task定义了默认的属性(Property)， 比如description、group、dependsOn、inputs、outputs等, 我们可以配置这些Property，如下配置Task的描述和分组：

```groovy
//subproject_2/build.gradle

//我们可以在定义Task时对这些Property进行赋值
task task1{
 	group = 'MyGroup'
 	description = 'Hello World'

 	doLast{
 		println "task分组：${group}"
 		println "task描述：${description}"
 	}
}
```

Gradle在执行一个Task之前，会先配置这个Task的Property，然后再执行这个Task的执行代码块，所以配置Task的代码块放在哪里都无所谓，如下：

```groovy
//subproject_2/build.gradle

//在定义Task之后才对Task进行配置
task task2{
	doLast{
		println "task分组：${group}"
 		println "task描述：${description}" 
	}
}
task2{
	group = 'MyGroup'
 	description = 'Hello World'
}

//等效于task2
task task3{
	doLast{
		println "task分组：${group}"
 		println "task描述：${description}" 
	}
}
task3.description = 'Hello World!'
task3.group = "MyGroup"

//等效于task3
task task4{
	doLast{
		println "task分组：${group}"
 		println "task描述：${description}" 
	}
}
task4.configure{
	group = 'MyGroup'
 	description = 'Hello World'
}
```

我们可以通过Task的**dependsOn**属性指定Task之间的依赖关系，如下：

```groovy
//subproject_2/build.gradle

//创建Task时通过dependsOn声明Task之间的依赖关系
task task5(dependsOn: task4){
	doLast{
		println 'Hello World'
	}
}

//或者在创建Task之后再声明task之间的依赖关系
task4.dependsOn task3
```

上述的依赖关系是task3 -> task4 -> task5，依赖的Task先执行，所以当我在命令行输入`gradle task5`执行task5时，输出：

```groovy
# chenjianyu @ FVFCG2HJL414 in ~/GradleDemo 
$ gradle task5  

> Task :subproject_2:task3
task分组：MyGroup
task描述：Hello World!

> Task :subproject_2:task4
task分组：MyGroup
task描述：Hello World

> Task :subproject_2:task5
Hello World
task5task执行完成

BUILD SUCCESSFUL in 2s
3 actionable tasks: 3 executed
```

依次执行task3、 task4 、task5。

### 3、自定义Task

前面创建的Task默认都是[DefaultTask](https://docs.gradle.org/current/dsl/org.gradle.api.DefaultTask.html#org.gradle.api.DefaultTask)类型，我们可以通过继承DefaultTask来自定义Task类型，Gradle中也内置了很多具有特定功能的Task，它们都间接继承自DefaultTask，如Copy(复制文件)、Delete(文件清理)等，我们可以直接在build.gradle中自定义Task，如下：

```groovy
//subproject_3/build.gradle

class MyTask extends DefaultTask{

  	def message = 'hello world from myCustomTask'

    @TaskAction
    def println1(){
      println "println1: $message"
    }

    @TaskAction
    def println2(){
      println "println2: $message"
    }
}
```

在MyTask中，通过**@TaskAction**注解的方法就是该Task的action，action是Task最主要的组成，它表示Task的一个执行动作，当Task中有多个action时，多个action的执行顺序按照**@TaskAction**注解的方法的放置顺序，所以执行一个Task的过程就是：doFirst方法 -> action方法 -> doLast方法，在MyTask中定义了两个action，接下来我们使用这个Task，如下：

```groovy
//subproject_3/build.gradle

//在定义Task时通过type指定Task的类型
task myTask(type: MyTask){
      message = 'custom message'
}
```

在定义Task时通过**type**指定Task的类型，同时还可以通过闭包配置MyTask中的message参数，在命令行输入`gradle myTask`执行这个Task，如下：

```groovy
# chenjianyu @ FVFCG2HJL414 in ~
$ gradle myTask        

> Task :subproject_3:myTask
println2: custom message
println1: custom message

BUILD SUCCESSFUL in 611ms
1 actionable task: 1 executed
```

我们自定义的Task本质上就是一个类，除了直接在build.gradle文件中编写自定义Task，还可以在Gradle项目的根目录下新建一个buildSrc目录，在buildSrc/src/main/[java/kotlin/groovy]目录中定义编写自定义Task，可以采用java、kotlin、groovy三种语句之一，或者在一个独立的项目中编写自定义Task，在后面自定义Plugin时会讲到这几种方式。

### 4、Task的输出



## 自定义Plugin

Plugin可以理解为一系列Task的集合，通过实现**Plugin<T>**接口的**apply**方法就可以自定义Plugin，自定义的Plugin本质上也是一个类，所以和Task类似，在Gradle中也提供了3种方式来编写自定义Plugin：

- **1、在build.gradle中直接编写**：可以在任何一个build.gradle文件中编写自定义Plugin，此方式自定义的Plugin只对该build.gradle对应的项目可见；
- **2、在buildSrc目录下编写**：可以在Gradle项目根目录的buildSrc/src/main/[java/kotlin/groovy]目录中编写自定义Plugin，可以采用java、kotlin、groovy三种语句之一，Gradle在构建时会自动的编译buildSrc/src/main/[java/kotlin/groovy]目录下的所有类文件为class文件，供本项目所有的build.gradle引用，所以此方式自定义的Plugin只对本Gradle项目可见；
- **3、在独立项目中编写**：可以新建一个Gradle项目，在该Gradle项目中编写自定义Plugin，然后把Plugin源码打包成jar，发布到maven、lvy等托管平台上，这样其他项目就可以引用该插件，所以此方式自定义的Plugin对所有Gradle项目可见.

由于在上面自定义Task的介绍中已经讲过了如何在build.gradle中直接编写，自定义Plugin也类似，所以下面就主要介绍**2、3**两种方式，而且这两种方式也是平时开发中自定义Plugin最常用的方式。

### 1、在buildSrc目录下编写

在GradleDemo中新建一个buildSrc目录，然后在buildSrc目录新建src/main/groovy目录，如果你要使用java或kotlin，则新建src/main/java或src/main/kotlin，src/main/groovy目录下你还可以继续创建package，这里我的package为com.example.plugin，然后在该package下新建一个类MyPlugin.groovy，该类继承自Plugin接口，如下：

{% asset_img gradle4.png gradle1 %}

```groovy
class MyPlugin implements Plugin<Project>{
	@Override
  void apply(Project project){}
}
```

现在MyPlugin中没有任何逻辑，我们平时是在build.gradle中通过**apply plugin: 'Plugin名'**来引用一个Plugin，而apply plugin中的apply就是指apply方法中的逻辑，而apply方法的参数project指的就是引用该Plugin的build.gradle对应的Project对象，接下来我们让我们在apply方法中编写逻辑，如下：

```groovy
package com.example.plugin

import org.gradle.api.*

class MyPlugin implements Plugin<Project>{
  
	@Override
  void apply(Project project){
    //通过project的ExtensionContainer的create方法创建一个名为outerExt的扩展，扩展对应的类为OuterExt
    //create方法返回OuterExt实例，我们可以在apply方法中使用OuterExt实例
    def outerExt = project.extensions.create('outerExt', OuterExt.class)
		
    //通过project的task方法创建一个名为showExt的Task
		project.task('showExt'){
			doLast{
        //使用OuterExt实例
				println "outerExt = $outerExt"
			}
		}
  }
  
  /**
   * 自定义插件的扩展对应的类
   */
  static class OuterExt{
		def name
		def message
    
    @Override
		String toString(){
			return "[name = $name, message = $message]"
		}
	}
}
```

上述我在apply方法中创建了一个扩展和一个Task，其中Task好理解，那么扩展是什么？我们平时引用android插件时，一定见过这样类似于android这样的命名空间，如下：

```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.3"
  	//...
}
```

它并不是一个名为android的方法，它而是android插件中名为android的扩展，该扩展对应一个bean类，该bean类中有compileSdkVersion、buildToolsVersion等方法，所以配置android就是在配置andorid对应的bean类，现在回到我们的MyPlugin中，MyPlugin也定义了一个bean类：OuterExt，该bean类有name和messag两个字段，Groovy会自动为我们生成get/set方法，而apply方法中通过project实例的ExtensionContainer的create方法创建一个名为outerExt的扩展，扩展对应的bean类为OuterExt，扩展的名字可以随便起，其中[ExtensionContainer](https://docs.gradle.org/current/javadoc/org/gradle/api/plugins/ExtensionContainer.html)类似于TaskContainer，它也是Project中的一个容器，这个容器存放Project中所有的扩展，通过ExtensionContainer的create方法可以创建一个扩展，create方法返回的是扩展对应的类的实例，这样我们使用MyPlugin就可以这样使用，如下：

```groovy
//subproject_4/build.gradle

apply plugin: com.example.plugin.MyPlugin

outerExt {
    name 'rain9155'
    message 'hello'
}

//执行gradle showExt, 输出:
//outerExt = [name = rain9155, message = hello]
```

扩展的特点就是可以通过闭包来配置扩展对应的类，这样就可以通过扩展outerExt来配置我们的Plugin，很多自定义Plugin都是都通过添加扩展这种方式来配置自定义的Plugin，很多人就问了，那么类似于android的嵌套DSL如何实现，如下：

```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.3"

    defaultConfig {
        applicationId "com.example.myapplication"
        minSdkVersion 16
        targetSdkVersion 29
        //...
    }
  //...
}
```

android{}中有嵌套了一个defaultConfig{}，但是defaultConfig并不是一个扩展，而是一个名为defaultConfig的方法，参数为[Action](https://docs.gradle.org/current/javadoc/org/gradle/api/Action.html)类型，它是一个接口，里面只有一个execute方法，这里就我参考android 插件的内部实现实现了嵌套DSL，原理就不深入探究了，嵌套DSL可以简单的理解为**扩展对应的类中再定义一个类**，MyPlugin的实现如下：

```groovy
package com.example.plugin

import org.gradle.api.*
import org.gradle.api.model.*//新引入

class MyPlugin implements Plugin<Project>{

	@Override
	void apply(Project project){
    //创建名为outerExt的扩展，扩展对应的类为OuterExt，create方法返回的是OuterExt实例
		def outerExt = project.extensions.create('outerExt', OuterExt.class)

		project.task('showExt'){
			doLast{
        //使用OuterExt实例和InnerExt实例
				println "outerExt = $outerExt, innerExt = ${outerExt.innerExt}"
			}
		}
	}

  static class OuterExt{
    def name
    def message
    //再定义一个类
    InnerExt innerExt

    //使用@Inject注解带ObjectFactory类型参数的构造
    @javax.inject.Inject
    public OuterExt(ObjectFactory objectFactory){
      //通过objectFactory的newInstance方法创建InnerExt类实例
      innerExt = objectFactory.newInstance(InnerExt.class)
    }

    //定义一个方法，该方法的参数类型为Action，泛型类型为InnerExt
    //其中方法名为可以随意起，它会在build.gradle中使用到
    void inner(Action<InnerExt> action){
      //调用Action的execute方法，传入InnerExt实例
      action.execute(innerExt)
    }

    @Override
    String toString(){
      return "[name = $name, message = $message]"
    }

    static class InnerExt{
      def name
      def message

      @Override
      String toString(){
        return "[name = $name, message = $message]"
      }
    }
  }
}
```

使用MyPlugin就可以这样使用，如下：

```groovy
//subproject_4/build.gradle

apply plugin: com.example.plugin.MyPlugin

//嵌套DSL
outerExt {
  name 'rain'
  message 'hello'

  //使用inner方法
  inner{
    name '9155'
    message 'word'
  }
}

//执行gradle showExt, 输出:
//outerExt = [name = rain, message = hello], innerExt = [name = 9155, message = word]
```

outerExt {}中嵌套了inner{}，其中inner是一个方法，参数类型为Action，Gradle内部会把inner方法后面的闭包配置给InnerExt类，总的来说，定义嵌套DSL的大概步骤如下：

- 1、定义嵌套的DSL对应的bean类，如这里为InnerExt；
- 2、定义一个带ObjectFactory类型参数的构造，并使用@Inject注解（@Inject是javax包下的，ObjectFactory是属于Gradle的model包下的类，通过@Inject注解的构造会被Gradle调用来实例化该类，并注入ObjectFactory实例）；
- 3、在构造中通过ObjectFactory对象的newInstance方法来创建bean类实例（通过ObjectFactory实例化的对象可以被闭包配置）；
- 4、定义一个方法，该方法的参数类型为Action，泛型类型为嵌套的DSL对应的bean类，方法名随便起，如这里为inner，然后在方法中调用Action的execute方法，传入bean类实例.

上面4步就是嵌套DSL时需要在自定义Plugin中做的事，Gradle还为我们提供了更灵活的命名嵌套DSL，通过[NamedDomainObjectContainer](https://docs.gradle.org/current/dsl/org.gradle.api.NamedDomainObjectContainer.html#org.gradle.api.NamedDomainObjectContainer)实现，它类似于android中buildType{}, 如下：

```groovy
android {
  //...
  buildTypes {
    release {
      //..
    }
    debug {
      //...
    }
  }
}
```

上面buildTypes中定义了2个命名空间，分别为：release、debug，每个命名空间都会生成一个 BuildType 配置，在不同的场景下使用，并且我还可以根据使用场景定义更多的命名空间如：test、testDebug等，buildTypes{}中的命名空间**数量不定**，命名空间的**名字不定**，这是因为buildTypes内部是通过NamedDomainObjectContainer容器实现的，大家有兴趣可以自己查阅相关实现，这里就不展开了。

> 还有一点要注意的是，对于扩展对应的bean类，如果你把它定义在自定义的Plugin的类文件中，一定要用**static**修饰，如这里的OuterExt类、InnerExt类使用了static修饰，或者把它们定义在单独的类文件中。

### 2、在独立项目中编写

在独立项目中编写和在buildSrc目录下编写是一样的，只是多了一个发布过程，这里我为了方便就不新建一个独立项目了，而是在GradleDemo中新增一个名为gradle_plugin的子项目，然后在gradle_plugin下新建一个src/main/groovy和src/main/resources目录，接着把刚刚在buildSrc编写的com.example.plugin.MyPlugin复制到src/main/groovy下，最后在GradleDemo新建一个repo目录，当作待会发布插件时的仓库，此时GradleDemo结构如下：

{% asset_img gradle5.png gradle1 %}

因为gradle_plugin项目中的MyPlugin.groovy使用了Gradle的相关api，如Project等，所以你要在gradle_plugin/build.gradle中引进Gradle api，打开gradle_plugin/build.gradle，添加如下：

```groovy
//gradle_plugin/build.gradle

//应用groovy插件
apply plugin: 'groovy'

dependencies {
    //这样gradle_plugin/src/main/groovy/中就可以使用Gradle和Groovy语法
    implementation gradleApi()
}
```

现在MyPlugin已经有了，我们需要给插件起一个名字，在gradle_plugin/src/main/resources目录下新建**META-INF/gradle-plugins**目录，然后在该目录下新建一个**XX.properties**文件，XX就是你想要给插件起的名字，就是apply plugin后填写的插件名字，例如andorid 插件名叫com.android.application，所以它的properties文件为com.android.application.properties，这里我给我的MyPlugin插件起名为**myplugin**，所以我新建myplugin.properties，如下：

{% asset_img gradle6.png gradle1 %}

打开myplugin.properties文件，添加如下：

```groovy
#在这里定义自定义插件的实现类，而插件的名字为properties文件的名字，如这里为：myplugin, 则引用插件时为apply plugin：'myplugin'
implementation-class=com.example.plugin.MyPlugin
```

通过implementation-class指明你要发布的插件的实现类，如这里为com.example.plugin.MyPlugin，接下来我们就来发布插件。

发布插件你可以选择你要发布到的仓库，如maven、lvy，我们最常用的就是maven了，所以这里我选择maven，Gradle提供了[maven插件](https://docs.gradle.org/current/userguide/maven_plugin.html#sec:maven_tasks)来帮助我们发布到maven，而maven又有本地仓库和远端仓库这两种类型的仓库，所以maven插件提供了install和uploadArchives这两种任务来帮助我们发布插件到maven的本地仓库和maven的远端仓库，在你的gradle/build.gradle添加如下：

```groovy
//gradle_plugin/build.gradle

//应用maven插件
apply plugin: 'maven'

//通过install任务上传自定义插件的jar文件和pom文件到maven本地repo, maven本地repo路径在mac下为～/.m/repository/，在window下为：C:/用户/用户名/.m/repository/
install {
    repositories.mavenInstaller {
        //通过pom配置自定义插件的group、artifact和version，通过classpath引用自定义插件时为：groupId:artifactId:version
        pom{
            groupId = 'com.example.customplugin'
            artifactId = 'myplugin'
            version = '1.0'
        }
    }
}

//通过uploadArchives任务上传自定义插件的jar文件和pom文件, 可以上传到本地指定目录地址或远端repo地址
uploadArchives{
    repositories.mavenDeployer{
        pom{
            groupId = 'com.example.customplugin'
            artifactId = 'myplugin'
            version = '2.0'
        }

        //上传到本地
        repository(url: uri('../repo'))

        //上传到远端
        //repository(url: uri('http://remote/repo')){
        //    authentication(userName: "name", password: "***")
        //}
    }
}
```

其中pom{}通过groupId、artifactId和version配置的是插件的classpath路径，即引用插件的路径，我们会在dependencies{}使用**classpath 'com.example.customplugin:myplugin:1.0'**来引用我们发布的插件。

接着在Gradle项目所处目录的命令行输入`gradle install  `来执行install任务，任务执行成功后，在maven本地repo（mac下为～/.m/repository/，在window下为：C:/用户/用户名/.m/repository/）中会看到上传的插件jar和pom文件，如下：

{% asset_img gradle7.png gradle1 %}

接着在Gradle项目所处目录的命令行输入`gradle uploadArchives`来执行uploadArchives任务，这里我指定uploadArchives上传的地址为本地GradleDemo/repo目录处，你也可以替换为你的远端maven地址，uploadArchives任务执行成功后，在GradleDemo/repo/中会看到上传的插件jar和pom文件，如下：

{% asset_img gradle8.png gradle1 %}

现在MyPlugin插件已经发布成功了，我发布MyPlugin的1.0和2.0两个版本到两个不同的本地目录中，接下来让我们使用这个插件。

在subproject_4/build.gradle中添加如下：

```groovy
//subproject_4/build.gradle

buildscript {
    repositories { 
        //添加maven本地repo
        mavenLocal()
        
        //添加maven远端repo
	   //mavenCentral()
        
        //添加uploadArchives上传时指定的本地路径
        maven {
            url uri('../repo')
        }
    }

    dependencies {
        //定义插件jar的classpath路径，gradle编译时会扫描该classpath下的所有jar文件
        //classpath 'com.example.customplugin:myplugin:1.0'
        classpath 'com.example.customplugin:myplugin:2.0'
    }
}
```

这里使用了Project的buildscript方法，在该方法中可以使用repositories方法和dependencies方法指定当前项目构建时依赖的仓库和类路径，项目在构建时会去repositories定义的仓库地址寻找classpath指定路径下的所有jar文件，如我这里repositories方法中指定了mavenLocal()和maven {url uri('../repo')}，其中mavenLocal()对应maven的本地仓库即/.m/repository，如果你通过uploadArchives上传到了远端，则可以新增mavenCentral()，它代表maven的远端地址，而dependencies方法中则通过classpath指定了插件在仓库下的类路径，这里我指定了MyPlugin 2.0的类路径: com.example.customplugin:myplugin:2.0，它在maven {url uri('../repo')}仓库下。

现在我们可以通过apply plugin使用MyPlugin了，在subproject_4/build.gradle中添加如下：

```groovy
//引用插件
apply plugin: 'myplugin'

//使用DSL配置插件的属性
outerExt{
    name 'rain'
    message 'hello'

    inner{
        name '9155'
        message 'word'
    }
}

//执行gradle showExt，输出：
//outerExt = [name = rain, message = hello], innerExt = [name = 9155, message = word]
```

其中apply plugin后面的插件名就是我们在gradle_plugin/src/main/resources/META-INF/gradle-plugins目录下编写的properties文件的名称。

> 在最新版的Gradle中，本文所使用的[maven插件](https://docs.gradle.org/current/userguide/maven_plugin.html#sec:maven_tasks)已经被废弃了，Gradle提供了[maven-publish插件](https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven:tasks)来替代，但是它的整体发布过程是类似。

## 结语

本文通过Gradle的特点、项目结构、生命周期、Task、自定义Plugin等来全面的学习Gradle，掌握上面的知识已经足够让你入门Gradle，但是如果你想要更进一步的学习Gradle，掌握本文的知识点是完全不足的，你可能还需要熟悉掌握Project中各个api的使用、能独立的自定义一个有完整功能的Plugin、能熟练地编写Gradle脚本来管理项目等，下面有一份Gradle DSL Reference，你可以在需要来查找Gradle相关配置项的含义和用法：

[Gradle DSL Reference](https://docs.gradle.org/current/dsl/index.html)

对于android开发者，我们引入的android插件中也有很多配置项，下面的Android  Plugin DSL Reference，你可以在需要来查找android 插件相关配置项的含义和用法：

[Android Plugin DSL Reference](https://developer.android.com/reference/tools/gradle-api)

本文的源码位置：

[GradleDemo](https://github.com/rain9155/GradleDemo)

以上就是本文的全部内容，希望大家有所收获！

参考内容：

[Gradle官网](https://docs.gradle.org/current/userguide/userguide.html)

[Gradle学习系列](https://www.cnblogs.com/davenkin/p/gradle-learning-1.html)

[Gradle插件从入门到进阶](https://juejin.im/post/5ccf02e36fb9a0322e73a3db#heading-52)



























