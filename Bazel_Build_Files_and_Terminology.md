###Introduction
Bazel在一个叫做`workspace`的目录下组织的排列代码来构建软件.在`workspace`中的源文件一嵌套的`package`层级结构组成,每个`package`都包含足足关联的源文件和一个`BUILD`文件.这个`BUILD`文件明确了资源最后生成的软件输出.

###WorkSpace, Package and Targets
####WorkSpace
`workspace`是一个在你文件系统上的目录,他包含了你想要构建的原件的源文件,并且包括那些构建输出的文件夹符号链接.每个`workspace`有一个叫做`WORKSPACE`的文本文件,它可能是空的或者包括构建数据所必要得外部依赖引用.请查看[`Workspace Rules`](https://bazel.build/docs/be/workspace.html)

####Packages
在`workspace`中,代码组织的基础单元叫做`package`.`package`是相关文件的集合以及一个对他们依赖的说明.

`package`被定义为包含一个名为`BUILD`的文件,位于`workspace`顶级目录下.一个`package`包括这个目录的所有文件,以及所有的子目录,除了那些自己有`BUILD`文件的目录.

比如,下面的目录有两个`package`,`my/app`, 和`subpackage` `my/app/tests`.注意`my/app/data`不是一个`package`,但是他是一个属于`my/app`的`package`的目录.

####Target
`package`是一个容器.`package`的元素被叫做`targets`.大多数`targets`是两种类型之一,`files`和`rules`.另外,还有一种其他的`target`,`package groups`,但是他们很少很少.

进一步,`files`被分为两种.`Source files`通常由人编写,并且录入仓库.`Generated files`,有时被称作`derived files`,并不会被录入,但是他们由构建工具将源文件根据特定的规则生成而来.

`target`第二种类型是`rule`.`rule`明确了输入集合和输出集合的关系,包括了从输入到输出的必要步骤.一个规则的输出往往是`generated files`.对于规则的输入有可能是`source file`,但是他们叶铿是`generate files`.因此,一个规则的输出有可能是一个规则的输入,允许构建长链规则.

无论规则的输入是一个`source file`或者是一个`generated file`在大多数情况是无关紧要的;重要的是文件的内容.这个事实让他能够轻松替换以`rule`生成的`generated file`复杂的`source file`,比如手动维护高度结构化文件负担加剧,而且有人写程序生成它.不需要对该文件的消费者进行修改.相反的一个`generated file`可以被一个源文件的本地更改简单替代.

生成规则也可能包含其他的`rules`.这些关系的精准意义通常十分复杂,而且`language-`或者`rule-`dependent,但是直觉上来说它是简单的:一个C++库的`rule`A可能有其他C++库的`rule`B来作为输入.这样一来的作用是B的头文件在A编译时是可用的,B的符号在A的链接期是可用的,B运行时的数据在A执行时是可用的.

所有`rules`的不变是这样,由`rule`生成的文件通常属于`rule`自身所属的的`package`中.`Package`group是被`package_group`方法定义的.他们有两个属性:他们所包含的`package`列表以及他们的名字.唯一允许的方法来引入他们是通过`visibility`规则属性或者使用`package`的`default_visibility`方法属性;他们不会生成或消费文件.更多信息,参考`Build Encyclopedia`的相关区域.

####Label
所有的`targets`属于一个明确的`package`.`target`的名字被叫做他的`label`,并且一个典型的`label`的权威形式应该像这样
>//my/app/main:app_binary

每个`label`有两部分,一个`package`名称(`my/app/main`)和一个`target`名称(`app_binary`).每个`label`唯一标识一个`target`.`label`有事以其他形式展现;当冒号被省略,`target`的名字将会被认为和`package`的最后一个组件相同,所以这两个是相同的:
>//my/app 
>
>//my/app:app
短格式`label`比如像`//my/app`是不回和`package`name混淆的.`label`以`//`卡头,但是`package`却不是,因此`my/app`是包含`//my/app`的package.(一个通常误区是`//my/app`是指`package`,或者代表在一个`package`中所有的`targets`;然而他们都不是正确的.
