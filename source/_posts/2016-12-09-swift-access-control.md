---
layout: post
title:  "Swift3 访问控制（Access Control）"
date:   2016-12-079
author: Jumpingfrog0 (黄东鸿）
comments: true
tags: iOS
---

### 访问控制（Access Control）

`Swift`中有非常完备的代码访问控制，代码写的好不好，从你的代码访问控制上就可以一眼看出。特别是在开源项目中，接口的设计及其重要，要简单易用，不暴露实现细节，做到可扩展性强，同时又要保证某些私有属性和方法不被使用者调用以避免Bug，这一切都是建立在访问控制的基础上的。设计模式中的`开闭原则: 对扩展开发，对修改关闭`就很好的诠释了`访问控制`的意思。

> `访问控制`可以限定你在源代码和模块中访问代码的级别，这个可以让你隐藏代码的实现细节，让你更好的设计一个接口，哪些代码是可以被访问和使用的。

其实这个东西说起来很简单，但在写代码的过程中却很容易被忽略，因为我们的代码经常是只写在一个工程中，工程内的所有代码都可以任意调用，运行起来也没有问题，但如果我们想写成一个开源库提供给别人使用时，这就需要特别重视了，接口设计的好不好，意味着你的开源库是否会被更多的人使用，在`GitHub`上骗的赞多不多了，手动滑稽~~ :)

本来想翻译苹果的官方文档的[Access Control](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html)，
翻译到一半，觉得内容实在是太多，懒得翻译，后来发现网上已经有人翻译好了，所以就拷贝了很多过来 - _ -  ：[http://www.devtalking.com/articles/swift-access-control/](http://www.devtalking.com/articles/swift-access-control/)

### Modules 和 源文件

`Swift`的访问控制是基于模块（module）和源文件（source file）这两个概念的。

* 模块：指的是`framework`或`app bundle`，可以用`import`关键字引入到其他模块中。
* 源文件：就是一个的`swift`文件，属于某一个模块。

### 访问级别

`Swift`中有5种不同的访问级别，这些访问级别是与被定义在源文件中的实体有关的，也与那些源文件所属的模块有关。

* `open`和`public`: 可以在所属模块的任意源文件中被访问，也可以在其他模块中被访问，两者的区别下文会介绍。
* `internal`：可以在所属的模块的任意源文件中被访问，但不能在其他模块中被访问。
* `fileprivate`：只能在定义自己的源文件中被访问。
* `private`：只能在封闭的声明块中被访问。

`open`是最高（限制最少）的访问级别，`private`是最低（限制最多）的访问级别。

`open`只能应用于类和类成员，它和`public`的区别如下：

* `public`级别的类，只能在所属模块中被子类化（被子类继承）
* `public`级别的类成员，只能在所属模块中被重载
* `open`级别的类可以在所属模块和其他模块中被子类化（被子类继承）
* `open`级别的类成员可以在所属模块和其他模块中被重载

#### 指导原则

总的来说，`访问级别`在`Swift`中遵循以下的指导原则：

	没有实体可以根据具有较低（更严格）访问级别的另一实体来定义。
	
举例：

* 一个`public`的变量，其类型不能为`internal`, `fileprivate`, `private`访问级别的，因为这个类型在使用该`public`变量的任何地方都可能不可用，这样会报错。
* 一个函数的访问级别不能高于它的参数和返回类型的访问级别，因为这样就可能会出现，函数可以被使用，但它的参数和返回类型却不能被访问，同样会报错。

在`Swift`中，如果不显式地指定访问级别，它就会有一个默认的访问级别 `internal`。

大多数情况下，我们都不需要显式地指定访问级别，因为我们大多数情况下都是在开发一个应用程序，那些代码不需要提供给其他应用或模块使用，当然，你也可以使用`fileprivate`或`private`来隐藏一些功能的实现细节。

而当你在开发开源库或`framework`时，就需要把一些实体定义为`public`或`open`了。比如你在设计一个类或接口，如果你希望它可以在其他模块中被使用，但在其他模块中又不能被继承时，就定义为`public`，这时这个类或接口仍可以在你自己的库中被继承，；如果你希望它可以在其他模块中被使用，同时在其他模块中也可以被继承时，就使用`open`。

明确地声明一个类为`open`，意味着你已经充分考虑了这个类在其他模块中被作为父类对代码的影响，并且你已经相应地设计了这个类的代码。

#### 单元测试（Unit Test）的访问级别

单元测试（Unit Test）在`Swift`中会被当成其他模块，一般情况下，只有被声明为`public`或`open`的实体才能在其他模块中被访问和使用，然而，如果你引用模块时使用了`@testable`，`单元测试`可以访问任何`internal`的实体了。

```swift
@testable import ProductModuleName
```

### 访问控制语法

通过修饰符`open`, `public`, `internal`, `fileprivate`, `private`来声明实体的访问级别。

```swift
public class SomePublicClass {}
internal class SomeInternalClass {}
fileprivate class SomeFilePrivateClass {}
private class SomePrivateClass {}
 
public var somePublicVariable = 0
internal let someInternalConstant = 0
fileprivate func someFilePrivateFunction() {}
private func somePrivateFunction() {}
```

除非显式地指定，否则默认的访问级别是`internal`。这意味着`SomeInternalClass`和`someInternalConstant`即使没有明确地使用修饰符声明访问级别，仍然有隐式的访问级别`internal`。

```swift
class SomeInternalClass {}              // implicitly internal
let someInternalConstant = 0            // implicitly internal
```

### 访问控制细节

#### 类（Types）

类的访问级别影响着类成员（属性、方法、构造函数、下标索引）的默认访问级别。

* 如果你定义了一个类的访问级别是`private`或`fileprivate`，那么它成员的默认访问级别就是`private`或`fileprivate`
* 如果你定义了一个类的访问级别是`internal`（或不显式地指定访问级别）或`public`，那么它成员的默认访问级别就是`internal`

> `public`类型的成员默认是`internal`访问级别，而不是`public`访问级别。如果你想要声明一个`public`的类成员，就要显式地使用`public`修饰符。这能够保证这个类的公共接口API是你明确指定哪些属性和方法是暴露出来的，可以避免将内部使用的属性和方法错误地公开成公共API。

```swift
public class SomePublicClass {                  // explicitly public class
    public var somePublicProperty = 0            // explicitly public class member
    var someInternalProperty = 0                 // implicitly internal class member
    fileprivate func someFilePrivateMethod() {}  // explicitly file-private class member
    private func somePrivateMethod() {}          // explicitly private class member
}
 
class SomeInternalClass {                       // implicitly internal class
    var someInternalProperty = 0                 // implicitly internal class member
    fileprivate func someFilePrivateMethod() {}  // explicitly file-private class member
    private func somePrivateMethod() {}          // explicitly private class member
}
 
fileprivate class SomeFilePrivateClass {        // explicitly file-private class
    func someFilePrivateMethod() {}              // implicitly file-private class member
    private func somePrivateMethod() {}          // explicitly private class member
}
 
private class SomePrivateClass {                // explicitly private class
    func somePrivateMethod() {}                  // implicitly private class member
}
```

#### 元祖（Tuple Types）

元祖的访问级别取决于你在该元祖中定义的所有类型的限制最严的那个访问级别。比如，你定义了一个拥有两种不同类型元素的元祖，一个是`internal`级别的，另一个是`private`级别的，那么这个元祖的访问级别就是`private`。

#### 函数（Function Types）

函数的访问级别需要根据参数类型和返回值类型中最严的访问级别计算得出。如果计算得出的访问级别不符合上下文，就必须显式地指定访问级别。

下面的例子中定义了一个全局函数`someFunction`,并且没有使用修饰符声明访问级别。你也许会认为它的访问级别是`internal`，但并非如此，事实上将会报错。

```swift
func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // 报错: Function must be declared fileprivate because its result uses a fileprivate type
}
```

因为函数的返回值类型的级别是`private`，所以你必须显式地使用修饰符声明它的访问级别为`private`：

```swift
private func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // function implementation goes here
}
```

将该函数声明为`public`或`internal`或者使用默认的访问级别`internal`都是错误的，因为如果把该函数当作`public`或`internal`级别来使用的话，是无法得到`private`级别的返回值的。

#### 枚举（Enumeration Types）

枚举中成员的访问级别与枚举类型的访问级别一致，你不能单独为枚举成员指定访问级别。
举个例子，`CompassPoint`枚举明确地声明为`public`，因此它的成员`north`, `south`, `east`和`west`的级别都是`public`：

```swift
public enum CompassPoint {
    case north
    case south
    case east
    case west
}
```

#### 嵌套类型（Nested Types）

* 如果在`private`级别的类型中定义嵌套类型，那么该嵌套类型就自动拥有`private`访问级别。
* 如果在`public`或者`internal`级别的类型中定义嵌套类型，那么该嵌套类型自动拥有`internal`访问级别。
* 如果想让嵌套类型拥有`public`访问级别，那么需要对该嵌套类型进行明确的访问级别声明。

#### 子类（Subclassing）

子类的访问级别不能高于父类的访问级别。
比如，父类的访问级别是`internal`，你就不能定义一个访问级别为`public`的子类。

此外，在满足子类不高于父类访问级别以及遵循各访问级别作用域（即模块或源文件）的前提下，你可以重写任意类成员（方法、属性、初始化方法、下标索引等）。

如果我们无法直接访问某个类中的属性或方法，那么可以继承该类，并重载相应地属性或方法，这样可以使我们更容易地访问该类的类成员。比如，`public`级别的类A有一个`fileprivate`级别的方法`someMethod`，类B是A的子类，访问级别是`internal`，但是类B重载了`someMethod`方法，并声明为`inernal`级别，这样`someMethod`就拥有比原来的实现更高的访问级别了。

```swift
public class A {
    fileprivate func someMethod() {}
}
 
internal class B: A {
    override internal func someMethod() {}
}
```

只要满足子类不高于父类访问级别以及遵循各访问级别作用域的前提下（即父类的`fileprivate`成员的作用域在同一个源文件中，子类的`internal`成员的作用域在同一个模块下），我们甚至可以在子类中，用子类成员访问父类成员，哪怕父类成员的访问级别比子类成员的要低：

```swift
public class A {
    fileprivate func someMethod() {}
}
 
internal class B: A {
    override internal func someMethod() {
        super.someMethod()
    }
}
```

因为父类A和子类B定义在同一个源文件中，所以在类B中可以在重写的`someMethod`方法中调用`super.someMethod()`。

#### Getters 和 Setters

常量、变量、属性、下标索引的`Getters`和`Setters`的访问级别继承自它们所属成员的访问级别。

`Setter`的访问级别可以低于对应的`Getter`的访问级别，这样就可以控制变量、属性或下标索引的读写权限。在*var*或*subscript*定义作用域之前，你可以通过`fileprivate `, `private(set)`或`internal(set)`先为它门的写权限申明一个较低的访问级别。

> 注意：这个规定适用于用作存储的属性或用作计算的属性。即使你不明确的申明存储属性的`Getter`, `Setter`，`Swift`也会隐式的为其创建`Getter`和`Setter`，用于对该属性进行读取操作。使用`fileprivate(set)`, `private(set)`和`internal(set)`可以改变`Swift`隐式创建的`Setter`的访问级别。在计算属性中也是同样的。

下面的例子中定义了一个结构体名为`TrackedString`，它记录了*value*属性被修改的次数：

```swift
struct TrackedString {
    private(set) var numberOfEdits = 0
    var value: String = "" {
    didSet {
        numberOfEdits++
    }
    }
}
```

`TrackedString`结构体定义了一个用于存储的属性名为*value*，类型为`String`，并将初始化值设为""（即一个空字符串）。该结构体同时也定义了另一个用于存储的属性名为*numberOfEdits*，类型为`Int`，它用于记录属性*value*被修改的次数。这个功能的实现通过属性*value*的`didSet`方法实现，每当给*value*赋新值时就会调用`didSet`方法，给*numberOfEdits*加一。

结构体`TrackedString`和它的属性*value*均没有明确的申明访问级别，所以它们都拥有默认的访问级别`internal`。但是该结构体的*numberOfEdits*属性使用`private(set)`修饰符进行申明，这意味着*numberOfEdits*属性只能在定义该结构体的源文件中赋值。*numberOfEdits*属性的`Getter`依然是默认的访问级别`internal`，但是`Setter`的访问级别是`private`，这表示该属性只有在当前的源文件中是可读可写的，在当前源文件所属的模块中它只是一个可读的属性。

如果你实例化`TrackedString`结构体，并且多次对*value*属性的值进行修改，你就会看到*numberOfEdits*的值会随着修改次数更改：

```swift
var stringToEdit = TrackedString()
stringToEdit.value = "This string will be tracked."
stringToEdit.value += " This edit will increment numberOfEdits."
stringToEdit.value += " So will this one."
println("The number of edits is \(stringToEdit.numberOfEdits)")
// prints "The number of edits is 3"
```

虽然你可以在其他的源文件中实例化该结构体并且获取到*numberOfEdits*属性的值，但是你不能对其进行赋值。这样就能很好的告诉使用者，你只管使用，而不需要知道其实现细节。

#### 初始化（Initializers）

我们可以给自定义的初始化方法指定访问级别，但是必须要低于或等于它所属类的访问级别。但如果该初始化方法是必须要使用的话，那它的访问级别就必须和所属类的访问级别相同。

如同函数或方法参数，初始化方法参数的访问级别也不能低于初始化方法的访问级别。

##### 默认初始化方法

`Swift`为结构体、类都提供了一个默认的无参初始化方法，用于给它们的所有属性提供赋值操作，但不会给出具体值。默认初始化方法的访问级别与所属类型的访问级别相同。

> 注意：如果一个类型被声明为`public`级别，那么默认的初始化方法的访问级别为`internal`。如果你想让无参的初始化方法在其他模块中可以被使用，那么你必须提供一个具有`public`访问级别的无参初始化方法。

##### 结构体的默认成员初始化方法

如果结构体中的任一存储属性的访问级别为`private`，那么它的默认成员初始化方法访问级别就是`private`。尽管如此，结构体的初始化方法的访问级别依然是`internal`。

如果你想在其他模块中使用该结构体的默认成员初始化方法，那么你需要提供一个访问级别为`public`的默认成员初始化方法。

#### 协议（Protocols）

如果你想为一个协议明确的声明访问级别，那么有一点需要注意，就是你要确保该协议只在你声明的访问级别作用域中使用。

协议中的每一个必须要实现的函数都具有和该协议相同的访问级别。这样才能确保该协议的使用者可以实现它所提供的函数。

> 注意：如果你定义了一个`public`访问级别的协议，那么实现该协议提供的必要函数也会是`public`的访问级别。这一点不同于其他类型，比如，`public`访问级别的其他类型，他们成员的访问级别为`internal`。

##### 协议继承

如果定义了一个新的协议，并且该协议继承了一个已知的协议，那么新协议拥有的访问级别最高也只和被继承协议的访问级别相同。比如说，你不能定义一个public的协议而去继承一个internal的协议。

##### 协议一致性

类可以采用比自身访问级别低的协议。比如说，你可以定义一个`public`级别的类，可以让它在其他模块中使用，同时它也可以采用一个`internal`级别的协议，并且只能在定义了该协议的模块中使用。

采用了协议的类的访问级别遵循它本身和采用协议中最低的访问级别。也就是说如果一个类是`public`级别，采用的协议是`internal`级别，那个采用了这个协议后，该类的访问级别也是`internal`。

如果你采用了协议，那么实现了协议必须的方法后，该方法的访问级别遵循协议的访问级别。比如说，一个`public`级别的类，采用了`internal`级别的协议，那么该类实现协议的方法至少也得是`internal`。

#### 扩展（Extensions）

扩展成员的默认级别和被你扩展的类型中原始成员的默认访问级别一致。

* 如果你扩展了一个`public`或`internal`的类型，任何新扩展成员的默认访问级别都是`internal`
* 如果你扩展了一个`fileprivate`的类型，任何新扩展成员的默认访问级别都是`fileprivate`
* 如果你扩展了一个`private`的类型，任何新扩展成员的默认访问级别都是`private`

当然，你也可以明确地声明扩展的访问级别（比如, `private extension`）给该扩展内所有成员指定一个新的默认访问级别。这个新的默认访问级别仍然可以被单独成员所指定的访问级别所覆盖。

##### 协议的扩展

如果一个扩展采用了某个协议，那么你就不能对该扩展使用访问级别修饰符来声明了。该扩展中实现协议的方法都会遵循该协议的访问级别。

#### 泛型（Generics）

泛型类型或泛型函数的访问级别遵循泛型类型、函数本身、泛型类型参数三者中访问级别最低的级别。

#### 别名（Type Aliases）

任何被你定义的类型别名都会被当作不同的类型来进行访问控制。一个类型别名的访问级别低于或等于这个类型的访问级别。比如说，一个`private`级别的类型别名可以设定一个`private`、`fileprivate`、`inernal`、`public`、或`open`的类型，但是一个`public`级别的类型别名不能设定`internal`、`fileprivate`或`private`的类型。

> 注意：这条规则也适用于为满足协议一致性而给相关类型命名别名。