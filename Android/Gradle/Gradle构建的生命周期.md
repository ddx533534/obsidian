## 1.构建阶段
1.**Initialization - 初始化阶段**

Gradle 支持单项目和多项目构建。在初始化阶段，Gradle会决定哪些项目参与构建，然后为每一个项目创建 Project 实例，即创建项目的层次结构。

2.**Configuration - 配置阶段**

在这个阶段主要是配置初始化阶段生成的每个 Project 实例对象，所有项目的build构建脚本在这个阶段执行。

3.**Execution - 执行阶段**

Gradle决定将要运行的任务子集，这些任务是在配置阶段创建和配置的。这些子集也由运行gradle命令时传递进去的任务参数决定的，然后开始执行。

# 2.初始化阶段
初始化阶段的任务是创建项目的层次结构，通过setting.gradle文件决定参与构建的所有项目，一个setting.gradle 对应一个`Setting`对象，最常用的`include`API就是Setting类下的一个方法。
对于单项目来说，setting.gradle 是可选的；但对于多项目来说，setting.gradle 是必须的，通过调用include方法决定需要参与构建的子项目。

```Groovy
include 'app'
```

为了更清楚项目的层次结构，可以执行下述命令：

``` groovy
> gradle -q projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'pay-demo'
+--- Project ':cashier'
+--- Project ':cashier-common'
+--- Project ':cashier-elderly'
+--- Project ':cashier-hybridwrapper'
+--- Project ':cashier-oneclick'
+--- Project ':cashier-web'
+--- Project ':meituanpay'
+--- Project ':meituanpay-common'
+--- Project ':meituanpay-desk'
+--- Project ':pay-common'
+--- Project ':payment-channel'
+--- Project ':samples'
\--- Project ':wallet'


To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :api:tasks
```

# 3.配置阶段
配置阶段的任务是执行各项目下的`build.gradle`脚本，完成对`Project`的配置，并且构造任务依赖关系图以便在执行阶段按照依赖关系执行Task。 每个`build.gralde`脚本文件对应一个`Project`对象，在初始化阶段创建。 配置阶段执行的代码包括`build.gralde`中的各种语句、闭包以及`Task`中的配置段语句.
注意： 无论执行Gradle的任何命令，**初始化阶段和配置阶段的代码都会被执行**。 因此我们在排查构建速度问题的时候可以留意，是否部分代码可以不在配置阶段中执行，或者可以写成任务Task从而减少配置阶段消耗的时间。


## 4.Hook
![[Pasted image 20220424161344.png]]