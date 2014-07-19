---
layout: post
title: Making Your Mac App’s Data Scriptable
category: "14"
date: "2014-07-11 11:00:00"
author: "<a href=\"http://inessential.com/\">Brent Simmons</a>"
tags: article
---


When adding AppleScript support — which is also JavaScript support, as of OS X 10.10 — it’s best to start with your app’s data. Scripting isn’t a matter of automating button clicks; it’s about exposing the model layer to people who could use your app in their workflows.
截至 OS X 10.10，当添加 AppleScript 支持，也可以说是 JavaScript 支持时，最好以你的应用数据作为开始。脚本并不是自动按钮点击；而是关于向可以在他们的工作流中使用你的应用的人暴露 model layer (模型层)。 

While that’s usually a small minority of users, they’re power users — the kind of people who recommend apps to friends and family. They blog and tweet about apps, and people listen to them. They can be your app’s biggest evangelists.
虽然通常这样的用户极少，他们是高级用户，会像朋友和家人推荐应用。他们的博客和 twitter 上有关于应用的内容，而且人们关注了他们。他们会成为你的应用的最大传播者。

Overall, the best reason to add scripting support is that it’s a matter of professionalism. But it doesn’t hurt that the effort is worth the reward.
总体而言，添加脚本支持的最大原因是它的的专业性因素。但是它不违背努力值得回报。

## Noteland

Noteland is an app without any UI except for a blank window — but it has a model layer, and it’s scriptable. You can [find it on GitHub](https://github.com/objcio/issue-14-scriptable-apps) and follow along.
Noteland 是一个没有任何除了空白窗口之外的 UI 的应用，但是它有 model layer，并且是脚本化的。你可以在 [GitHub](https://github.com/objcio/issue-14-scriptable-apps) 上关注它。

It supports AppleScript (and JavaScript on 10.10). It’s written in Objective-C in Xcode 5.1.1. We initially tried to use Swift and Xcode 6 Beta 2, but ran into snags, though it’s entirely likely they were our own fault, since we’re still learning Swift.
它支持 AppleScript（10.10上也支持JavaScript）。它在 Xcode 5.1.1 中用 Objective-C 完成。我们最初试图使用 Swift 和 Xcode 6 Beta 2，但是它出现了困难，虽然这完全可能是我们自己的错误，因为我们仍然在学习 Swift。

### Noteland’s Object Model
### Noteland 的对象模型

There are two classes: notes and tags. There may be multiple notes, and a note may have multiple tags.
有两个类，notes 和 tags。可能有多个笔记，而且一个笔记也许有多个标签。

NLNote.h declares several properties: `uniqueID`, `text`, `creationDate`, `archived`, `tags`, and a read-only `title` property.
NLNote.h 声明了几个属性: `uniqueID`, `text`, `creationDate`, `archived`, `tags`, 和一个只读的 `title` 属性。

Tags are simpler. NLTag.h declares two scriptable properties: `uniqueID` and `name`.
Tags 类更加简单。NLTag.h 声明了两个脚本化属性: `uniqueID` 和 `name`.

We want users to be able to create, edit, and delete notes and tags, and to be able to access and change all of their properties, with the exception of any that are read-only.
我们希望用户能够创建，编辑和删除笔记和标签，并且能够访问和改变除了只读以外的属性。

### Scripting Definition File (.sdef)
### 脚本定义文件(.sdef)

The first step is to define the scripting interface — it’s conceptually like creating a .h file for scripters, but in a format that AppleScript understands.
第一个步骤是定义脚本接口，概念上可以理解为为脚本创建一个 .h 文件，但是以 AppleScript 能够识别的格式。

In the past, we’d create and edit an aete resource (“aete” stands for Apple Event Terminology.) These days it’s much easier: we create and edit an sdef (scripting definition) XML file.
过去，我们需要创建和编辑 aete 资源（“aete” 代表 Apple Event Terminology）。现在这将更加容易：我们可以创建一个 sdef （脚本定义）XML 文件。

You might think you’d prefer JSON or a plist, but XML is a decent match for this — beats an aete resource hands-down, at least. In fact, there was a plist version for a while, but it required *two* different plists that you had to keep in sync. It was a pain.
你可能更倾向于使用 JSON 或者 plist，但是 XML 在这里会更加合适，至少它毫无疑问战胜了 aete 资源。事实上，曾有一段时间有 plist 版本，但是它要求你保持 *两个* 不同的 plist 同步。这非常痛苦。

The original name of the resource points to a matter worth noting. An Apple event is the low-level message that AppleScript generates, sends, and receives. It’s an interesting technology on its own, and has uses outside of scripting support. Additionally, it’s been around since System 7 in the early ’90s, and has survived the transition to OS X.
资源点的原始名称值得注意。Apple event 是 AppleScript 生成，发送和接受低级别的消息。 这本身是一种很有趣的技术，而且有脚本支持以外的用途。除此之外，在 90 年代初从 System 7 开始一直围绕它，并且一直存在到过渡到 OS X。

(Speculation: Apple events survived because so many print publishers relied on AppleScript, and publishers were among Apple’s most loyal customers during the 'dark days,' in the middle and late ’90s.)
（猜测：Apple event 的存活是由于很多印刷出版商依赖 AppleScript，在 90 年代中后期的 '黑暗日子' 中，出版商们是 Apple 最忠诚的用户。）

An sdef file always starts with the same header:
sdef 文件总是以同样的头部作为开始：

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE dictionary SYSTEM "file://localhost/System/Library/DTDs/sdef.dtd">

The top-level item is a dictionary — “dictionary” is AppleScript’s word for a scripting interface. Inside the dictionary you'll find one or more suites.
顶级的元素是字典，“dictionary” 是 AppleScript 的一个脚本接口。在字典中你会发现一个或多个套件。

(Tip: open AppleScript Editor and choose File > Open Dictionary… You’ll see a list of apps with scripting dictionaries. If you choose one — iTunes, for instance — you’ll see the classes, properties, and commands that iTunes understands.)
（提示：打开 AppleScript Editor，然后选择 File > Open Dictionary...你会看到有脚本字典的应用列表。如果你选择 iTunes 作为例子，你会看到类，属性和 iTunes 能识别的命令。）

    <dictionary title="Noteland Terminology">

#### Standard Suite
#### 标准套件

The standard suite defines classes and commands all applications should support. It includes quitting, closing windows, making and deleting objects, querying objects, and so on.
标准套件定义了应用应该支持的所有类和操作。其中包括退出，关闭窗口，创建和删除对象，查询对象等等。

To add it to your sdef file, copy and paste from the canonical copy of the standard suite at `/System/Library/ScriptingDefinitions/CocoaStandard.sdef`.
将它添加到你的 sdef 文件，从位于 `/System/Library/ScriptingDefinitions/CocoaStandard.sdef` 的标准套件中复制和粘贴。

Copy everything from `<suite name="Standard Suite"`, through and including the closing `</suite>`.
从 `<suite name="Standard Suite"`, 从头到尾且包括结尾 `</suite>` 复制所有东西。

Paste it right below the `dictionary` element in your sdef.
将它粘贴到你的 sdef 文件中 `dictionary` 元素的正下方。

Then, in your sdef file, go through and delete everything that doesn’t apply. Noteland isn’t document-based and doesn’t print, so we removed the open and save commands, the document class, and everything to do with printing.
然后，在你的 sdef 文件中，遍历并删除没有应用到的所有东西。Noteland 不基于文档且无需打印，所以我们去掉了打开和保存命令，文件类，以及与打印有关的一切。

(Tip: Xcode does a good job indenting XML. To re-indent, select all the text and choose the Editor > Structure > Re-Indent command.)
（建议：Xcode 在 XML 的缩进方面做得很好，为了重新缩进，选中所有文本并且选择 Editor > Structure > Re-Indent。）

Once you’ve finished editing, use the command-line xmllint program — `xmllint path/to/noteland.sdef` — to make sure your XML is okay. If it just displays the XML, without errors or warnings, then it’s fine. (Remember that you can drag the document proxy icon from a window title in Xcode into Terminal and it will paste in the path to the file.)
当你完成编辑后，使用命令行 xmllint 程序 `xmllint path/to/noteland.sdef` 以确保 XML 是正常的。如果它只显示了 XML，没有错误和警告，那么就是正确的。（记住你可以在 Xcode 的窗口标题栏拖拽文件的代理图标到终端，然后会粘贴文件的路径。）

#### Noteland Suite
#### Noteland 套件

A single app-defined suite is usually best, though not mandated: you could have more than one when it makes sense. Noteland defines just one, the Noteland Suite:
一个单一的应用定义套件通常是最好的，虽然并不是强制的：当它起作用时你可以有超过一个。Noteland 只定义一个，下面是 Noteland 套件：

    <suite name="Noteland Suite" code="Note" description="Noteland-specific classes.">

A scripting dictionary expects things to be contained by other things. The top-level container is the application object itself.//here ,,,,
脚本字典预期的东西被包含在其他东西中。顶级容器是应用程序对象本身。

In Noteland, its class name is `NLApplication`. You should always use the code `capp` for the application class: it’s a standard Apple event code. (Note that it’s also present in the standard suite.)
在 Noteland 中，它的类名是 `NLApplication`。对于应用的类你应该总是使用代码 `capp` ：这是一个标准的 Apple event 代码。（注意它也存在于标准套件中。）

    <class name="application" code="capp" description="Noteland’s top level scripting object." plural="applications" inherits="application">
        <cocoa class="NLApplication"/>

The application contains an array of notes. It’s important to differentiate elements (items there can be more than one of) and properties. In other words, an array in code should be an element in your dictionary:
该应用包含一个笔记的数组。区分元素（这里可以有不止一个项目）和属性非常重要。换句话说，代码中的数据应该作为你的字典中的一个元素。

    <element type="note" access="rw">
        <cocoa key="notes"/>
    </element>`

Cocoa scripting uses Key-Value Coding (KVC), and the dictionary specifies the key names.
Cocoa 脚本使用 KVC，和字典指定的键值名。

#### Note Class
#### Note 类

    <class name="note" code="NOTE" description="A note" inherits="item" plural="notes">
        <cocoa class="NLNote"/>`

The code is `NOTE`. It could be almost anything, but note that Apple reserves all lowercase codes for its own use, so `note` wouldn’t be allowed. It could be `NOT*`, or `NoTe`, or `XYzy`, or whatever you want. (Ideally the code wouldn’t collide with codes used by other apps. But there’s no way that we know of to ensure that, so we just, well, *guess*. That said, `NOTE` may not be all that great of a guess.)
上面的编码是 `NOTE`。这几乎可以是任何东西，但是请注意，Apple 保留所有的小写编码供自己使用，所以 `note` 是不被允许的。它可以是 `NOT*`, 或 `NoTe`, 或 `XYzy`，或者任何你想要的。（理想情况下自己的编码不会与其他应用的编码冲突。但是我们没有办法确保这一点，所以我们只能够 *猜测*。也就是说， 猜想 `NOTE` 可能并不是一个很好的选择。）

Your classes should inherit from `item`. (In theory you could have a class the inherits from another of your classes, but we’ve never tried this.)
你的类应该继承自 `item`。（理论上，你可以让一个类继承自你的另一个类，但是我们没有做过这个尝试。）

The note class has several properties:
note 类有多个属性：

    <property name="id" code="ID  " type="text" access="r" description="The unique identifier of the note.">
        <cocoa key="uniqueID"/>
    </property>
    <property name="name" code="pnam" type="text" description="The name of the note — the first line of the text." access="r">
        <cocoa key="title"/>
    </property>
    <property name="body" code="body" description="The plain text content of the note, including first line and subsequent lines." type="text" access="rw">
        <cocoa key="text"/>
    </property>
    <property name="creationDate" code="CRdt" description="The date the note was created." type="date" access="r"/>
    <property name="archived" code="ARcv" description="Whether or not the note has been archived." type="boolean" access="rw"/>

Whenever possible, it’s best to provide unique IDs for your objects. Otherwise, scripters have to rely on names and positions, which may change. Use the code 'ID  ' for unique IDs. (Note the two spaces; codes are four-character codes.) The name of the unique ID should always be `id`.
如果可能，最好为你的对象提供独一无二的 ID。否则，脚本不得不依赖于名字和位置，这可能会改变，对唯一的 ID 使用编码 'ID  '。（注意前面的两个空格；编码应该是四个字符。）

It’s also standard to provide a `name` property, whenever it makes sense, and the code should be `pnam`. Noteland makes this a read-only property, since the name is just the first line of the text of a note, and the text of the note is edited via the read-write `body` property.
当它有意义时，也标准的提供了 `name` 属性，编码应该是 `pnam`。Noteland 使得它是一个只读属性，因为名称只是笔记中文本的第一行，而且笔记的文本通过可读写属性 `body` 编辑。 

For `creationDate` and `archived`, we don’t need to provide a Cocoa key element, since the key is the same as the property name.
对于 `creationDate` 和 `archived`，我们并不需要提供 Cocoa 的键元素，因为键和属性名字相同。

Note the types: text, date, and boolean. AppleScript supports these and several more, as [listed in the documentation](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ScriptableCocoaApplications/SApps_about_apps/SAppsAboutApps.html#//apple_ref/doc/uid/TP40001976-SW12).
注意类型： text, date, 和 boolean。AppleScript 支持它们和其它几个，如 [listed in the documentation](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ScriptableCocoaApplications/SApps_about_apps/SAppsAboutApps.html#//apple_ref/doc/uid/TP40001976-SW12).

Notes can also have tags, and so there’s a tags element:
笔记可以有标签，所以下面是一个标签元素：

    <element type="tag" access="rw">
        <cocoa key="tags"/>
    </element>
    </class>`

#### Tag Class
#### Tag 类

Tags are `NLTag` objects:
Tags 是 `NLTap` 对象：

    <class name="tag" code="TAG*" description="A tag" inherits="item" plural="tags">
        <cocoa class="NLTag"/>`

Tags have just two properties, `id` and `name`:
Tags 只有两个属性，`id` 和 `name`：

    <property name="id" code="ID  " type="text" access="r" description="The unique identifier of the tag.">
        <cocoa key="uniqueID"/>
    </property>
    <property name="name" code="pnam" type="text" access="rw">
        <cocoa key="name"/>
    </property>
    </class>

That ends the Noteland suite and the entire dictionary:
下面的代码是 Noteland 套件和整个字典的结束：

        </suite>
    </dictionary>

### App Configuration
### 应用程序配置

Apps aren’t scriptable out of the box. In Xcode, edit the app’s Info.plist.
应用在沙盒外不是脚本化的。在 Xcode 中，编辑应用的 Info.plist。

Since the app uses a custom `NSApplication` subclass — in order to provide the top-level container — we edit Principal Class (`NSPrincipalClass`) to say `NLApplication` (the name of Noteland’s `NSApplication` subclass).
因为应用使用了一个自定义的 `NSApplication` 子类，为了提供顶级容器，我们编辑主体类 (`NSPrincipalClass`) 声明 `NLApplication` (Noteland 的 `NSApplication` 子类名字).

We also add a Scriptable (`NSAppleScriptEnabled`) key and set it to YES. And finally, we add a Scripting definition file name (`OSAScriptingDefinition`) key and give it the name of the sdef file: noteland.sdef.
我们还添加了一个脚本化的键（`OSAScriptingDefinition`）并且设置它为 YES。最后，我们添加一个名为（`OSAScriptingDefinition`） 键的脚本定义文件，以及将它 sdef 文件命名为：noteland.sdef。

### Code
### 代码

#### NSApplication subclass
#### NSApplication 子类
You may be surprised by how little code there is to write.
你可能会惊讶竟然只需要写那么少的代码。

See NLApplication.m in the Noteland project. It lazily creates a notes array and provides some dummy data. Lazily just because. It has no connection to scripting support.
见 Noteland 工程中的 NLApplication.m 文件。它懒惰地创建了一个笔记数组且提供了一些虚拟的数据。说懒惰只是因为，它没有连接脚本支持。

(Note that there’s no object persistence, since I want Noteland to be as free as possible from things other than scripting support. You’d use Core Data or an archiver or something to persist data.)
（注意这里没有对象持久化，因为我想让 Noteland 尽可能自由，不仅仅是脚本支持。你可以使用 Core Data 或者 archiever（归档）或者其它东西来保存数据。）

It could have just skipped the dummy data and provided an array.
它可能只跳过了虚拟数据和提供的数组。

In this case, the array is an `NSMutableArray`. It wouldn’t have to be — if it’s an `NSArray`, then Cocoa scripting will just replace the notes array when changes are made. But if we make it an `NSMutableArray` *and* we provide the following two methods, then the array won’t be replaced. Instead, objects will be added and removed from the mutable array:
在本例中，数组是 `NSMutableArray` 类型的。如果是 `NSArray` ，就不需要那样了，然后 Cocoa 脚本只会在发生改变时替换笔记数组。但是如果我们让它作为 `NSMutableArray` 数组 *且* 提供下面两个方法，那么这个数组就不会被替换。取而代之，对象将会被添加，以及从可变数组中移除。 

    - (void)insertObject:(NLNote *)object inNotesAtIndex:(NSUInteger)index {
        [self.notes insertObject:object atIndex:index];
    }

    - (void)removeObjectFromNotesAtIndex:(NSUInteger)index {
        [self.notes removeObjectAtIndex:index];
    }

Also note that the notes array property is declared in the .m file in the class extension. There’s no need to put it in the .h file. Since Cocoa scripting uses KVC and doesn’t care about your headers, it will find it.
另外需要注意，笔记数组在类扩展的 .m 文件中被声明。不需要将它放到 .h 文件中。因为 Cocoa 脚本使用 KVC ,而且不关心你的header部分，它会找到它的。

#### NLNote Class
#### NLNote 类

NLNote.h declares the various properties of a note: `uniqueID`, `text`, `creationDate`, `archived`, `title`, and `tags`.
NLNote.h 声明了笔记的各个属性： `uniqueID`, `text`, `creationDate`, `archived`, `title`, 和 `tags`。

The `init` method sets the `uniqueID` and `creationDate` and sets the tags array to an empty `NSArray`. We're using an `NSArray` this time, rather than an `NSMutableArray`, just to show it can be done.)
在 `init` 方法中设置 `uniqueID` 和 `creationDate`，还有将标签数组设为空的 `NSArray`。这次我们使用 `NSArray` 而不是 `NSMutableArray`，仅仅为了说明它也可以达到目的。
 
The `title` method returns a calculated value: the first line of the text of the note. (Recall that this becomes the `name` to the scripting dictionary.)
`tilte` 方法返回一个计算后的值：笔记中文本的第一行。（回想一下，则会成为了脚本字典的 `name`。）

The method to note is the `objectSpecifier` method. This is critical to your classes; scripting support needs this so it understands your objects.
要注意 `objectSpecifier` 方法。这是类的关键；脚本支持需要这个使其能够理解你的对象。

Luckily this method is easy to write. Though there are different types of object specifiers, it’s usually best to use `NSUniqueIDSpecifier`, since it’s stable. (Other options include `NSNameSpecifier`, `NSPositionalSpecifier`, and so on.)
幸运的是，这个方法很容易实现。虽然对象说明符有不同类型，通常最好使用 `NSUniqueIDSpecifier`，因为它很稳定。（其它选项包括：`NSNameSpecifier`, `NSPositionalSpecifier` 等。）

The object specifier needs to know about the container, and the container is the top-level Application object.
对象说明符需要了解容器相关的东西，而且容器是顶级应用的对象。

The code looks like this:
代码如下所示：

    NSScriptClassDescription *appDescription = (NSScriptClassDescription *)[NSApp classDescription];
    return [[NSUniqueIDSpecifier alloc] initWithContainerClassDescription:appDescription containerSpecifier:nil key:@"notes" uniqueID:self.uniqueID];

`NSApp` is the global application object; we get its `classDescription`. The key is `@"notes"`, a nil `containerSpecifier` refers to the top-level (app) container, and the `uniqueID` is the note’s `uniqueID`.
`NSApp` 是全局应用的对象；我们获取它的 `classDescription`。键为 `@"notes"`，`containerSpecifier` 为 nil 指的是顶级（应用）的容器， `uniqueID` 是笔记的 `uniqueID`。

#### Note as Container
#### Note 作为容器

We have to think ahead a little bit. Tags will need an `objectSpecifier` also, and tags are contained by notes — so a tag needs a reference to its containing note.
我们需要超前考虑一点。标签也会需要 `objectSpecifier`，而且笔记是标签的容器，所以标签需要引用它的容器--笔记。

Cocoa scripting handles creating tags, but there’s a method we can override that lets us customize the behavior.
Cocoa 脚本处理创建标签，但是我们可以重写让自己自定义行为的方法。

NSObjectScripting.h defines `-newScriptingObjectOfClass:forValueForKey: withContentsValue:properties:`. That’s what we need. In NLNote.m, it looks like this:
NSObjectScripting.h 定义了 `-newScriptingObjectOfClass:forValueForKey: withContentsValue:properties:`。这正是我们需要的。在 NLNote.m 中，它是下面这样的：

    NLTag *tag = (NLTag *)[super newScriptingObjectOfClass:objectClass forValueForKey:key withContentsValue:contentsValue properties:properties];
    tag.note = self;
    return tag;

We create the tag using super’s implementation, then set the tag’s `note` property to the note. To avoid a possible retain cycle, NLTag.h makes the note a weak property.
我们使用父类的接口创建标签，然后设置标签的 `note` 属性为该笔记。为了避免可能的 retain 周期，NLTag.h 的 note 是 weak 属性。

(You might think this is a bit inelegant, and we’d agree. We wish instead that containers were asked for the `objectSpecifiers` for their children. Something like `objectSpecifierForScriptingObject:` would be better. We filed a bug: [rdar://17473124](rdar://17473124).)
（你可能认为这并不太不优雅，我们同意这么说。我们希望取代那种为了子类的 `objectSpecifiers` 而需要 `objectSpecifiers` 的容器。像是 `objectSpecifierForScriptingObject:` 这样可能会更好。我们提出了一个 bug[rdar://17473124](rdar://17473124)。）

#### NLTag Class
#### NLTag 类

`NLTag` has `uniqueID`, `name`, and `note` properties.
`NLTag` 有 `uniqueID`, `name`, 和 `note` 属性。

`NLTag`’s `objectSpecifier` is conceptually the same as the code in `NLNote`, except that the container is a note rather than the top-level application class.
`NLTag` 的 `objectSpecifier` 在概念上和 `NLNote`中的代码相同，除了容器是一个笔记而不是顶级应用类。

It looks like this:
如下所示：

    NSScriptClassDescription *noteClassDescription = (NSScriptClassDescription *)[self.note classDescription];
    NSUniqueIDSpecifier *noteSpecifier = (NSUniqueIDSpecifier *)[self.note objectSpecifier];
    return [[NSUniqueIDSpecifier alloc] initWithContainerClassDescription:noteClassDescription containerSpecifier:noteSpecifier key:@"tags" uniqueID:self.uniqueID];

That’s it. Done. That’s not much code — most of the work is in designing the interface and editing the sdef file.
就是这样。完成了。并没有太多代码，大量的工作都是设计 interface 和编辑 sdef 文件。

In the old days, you’d still be writing Apple event handlers and working with Apple event descriptors and all kinds of crazy jazz. In other words, you’d be a long way from done. Thankfully, these aren’t the old days.
在过去，你需要编写 Apple event 处理程序，并与 Apple event 描述符和各种疯狂的爵士乐一起工作。换句话说，要完成这些你需要走很长的路。值得庆幸的是，已经不是过去的日子了。 

The fun part is next.
下一个部分会很有趣。

### AppleScript Editor
### AppleScript Editor

Launch Noteland. Launch /Applications/Utilities/AppleScript Editor.app.
启动 Noteland。启动 /Applications/Utilities/AppleScript Editor.app。

Run the following script:
运行下面的脚本：

    tell application "Noteland"
        every note
    end tell

In the Result pane at the bottom, you’ll see something like this:
在底部的结果窗口中，你会看到下面这样的信息：

    {note id "0B0A6DAD-A4C8-42A0-9CB9-FC95F9CB2D53" of application "Noteland", note id "F138AE98-14B0-4469-8A8E-D328B23C67A9" of application "Noteland"}

The IDs will be different, of course, but this is an indication that it’s working.
当然，ID 会有所不同，但是这是迹象表明，它在工作。

Try this script:
试一试这个脚本：

    tell application "Noteland"
        name of every note
    end tell

You’ll see `{"Note 0", "Note 1"}` in the Result pane.
你会在结果窗中看到 `{"Note 0", "Note 1"}`。

Try this script:
再试一下这个脚本：

    tell application "Noteland"
        name of every tag of note 2
    end tell

Result: `{"Tiger Swallowtails", "Steak-frites"}`.
结果：`{"Tiger Swallowtails", "Steak-frites"}`。

(Note that AppleScript arrays are 1-based, so note 2 refers to the second note. Which doesn’t sound so crazy when we put it that way.)
（请注意 AppleScript 数组是基于 1 的，所以 2 指的是第二个笔记。当我们这样说时，听起来并不疯狂）

You can also create notes:
你也可以创建一个笔记：

    tell application "Noteland"
        set newNote to make new note with properties {body:"New Note" & linefeed & "Some text.", archived:true}
        properties of newNote
    end tell

The result will be something like this (with appropriate details changed):
结果将会是类似这样的（详细信息有相应改变）：

    {creationDate:date "Thursday, June 26, 2014 at 1:42:08 PM", archived:true, name:"New Note", class:note, id:"49D5EE93-655A-446C-BB52-88774925FC62", body:"New Note\nSome text."}`

And you can create new tags:
你还可以创建新的标签：

    tell application "Noteland"
        set newNote to make new note with properties {body:"New Note" & linefeed & "Some text.", archived:true}
        set newTag to make new tag with properties {name:"New Tag"} at end of tags of newNote
        name of every tag of newNote
    end tell

The result will be: `{"New Tag"}`.
结果会是：`{"New Tag"}`。

It works!
这是可行的！

### More to Learn
### 扩展

Scripting the object model is just part of adding scripting support; you can add support for commands, too. For instance, Noteland could have an export command that writes notes to files on disk. An RSS reader might have a refresh command, a Mail app might have a download mail command, and so on.
将对象模型脚本化是是添加脚本支持的一部分；你也可以为命令添加支持。例如，Noteland 可以有一个将笔记写到硬盘文件的导出命令。RSS 阅读器可能有一个刷新命令，邮件应用可能有下载邮件命令，等等。

Matt Neuburg’s [AppleScript: The Definitive Guide](http://www.amazon.com/AppleScript-Definitive-Guide-Matt-Neuburg/dp/0596102119/ref=la_B001H6OITU_1_1?s=books&ie=UTF8&qid=1403816403&sr=1-1) is worth checking out even though it was published in 2006, as things haven’t changed much since then. Matt also has a [tutorial on adding scripting support to Cocoa apps](http://www.apeth.net/matt/scriptability/scriptabilityTutorial.html). It's definitely worth reading, and it goes into more detail than this article.
Matt Neuburg 的 [AppleScript 权威指南](http://www.amazon.com/AppleScript-Definitive-Guide-Matt-Neuburg/dp/0596102119/ref=la_B001H6OITU_1_1?s=books&ie=UTF8&qid=1403816403&sr=1-1) 值得一读，尽管它是 2006 年出版的，因为从那以后并没有发生太大的改变。Matt 还写有一篇 [Cocoa 应用添加脚本支持的教程](http://www.apeth.net/matt/scriptability/scriptabilityTutorial.html)。该教程绝对值得一读，它比这篇文章更加详细。

There’s a session in the [WWDC 2014 videos](https://developer.apple.com/videos/wwdc/2014/) on JavaScript for Automation, which talks about the new JavaScript OSA language. (Years ago, Apple suggested that one day there would be a programmer’s dialect of AppleScript, since the natural language thing is a bit weird for people who write in C and C-like languages. JavaScript could be considered the programmer’s dialect.)
这有一个 [WWDC 2014 Session 的视频](https://developer.apple.com/videos/wwdc/2014/),关于 JavaScript 的自动化，其中谈到了新的 JavaScript OSA 语言。（多年以前 Apple 曾提出，总有一天会出现 AppleScript 的程序员的特有语言，因为自然语言对写 C 和 C 类语言的人说略有一点怪。JavaScript 可以被认为是程序员的特有语言。）

And of course, Apple has documentation on the various technologies:
当然，Apple 有关于这些技术的文档：

- [Cocoa Scripting Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ScriptableCocoaApplications/SApps_intro/SAppsIntro.html#//apple_ref/doc/uid/TP40002164)
- [Cocoa 脚本指南](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ScriptableCocoaApplications/SApps_intro/SAppsIntro.html#//apple_ref/doc/uid/TP40002164)
- [AppleScript Overview](https://developer.apple.com/library/mac/documentation/applescript/conceptual/applescriptx/AppleScriptX.html#//apple_ref/doc/uid/10000156-BCICHGIE)
- [AppleScript 概览](https://developer.apple.com/library/mac/documentation/applescript/conceptual/applescriptx/AppleScriptX.html#//apple_ref/doc/uid/10000156-BCICHGIE)

Also, see Apple’s Sketch app for an example of an app that implements scripting.
此外，请参阅 Apple 的一个素描应用，它实现了脚本化。
