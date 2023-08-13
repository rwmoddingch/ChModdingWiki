# IL简易教程
## 1.  什么是IL

IL是.NET框架中中间语言（Intermediate Language）的缩写。使用.NET框架提供的编译器可以直接将源程序编译为.exe或.dll文件，但此时编译出来的程序代码并不是CPU能直接执行的机器代码，而是一种中间语言IL（Intermediate Language）的代码(来源百度)。

当然我觉得如果只是这么说的话可能还是会有很多人不理解IL到底是什么。

首先我们来看一小段IL代码的示例

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/1.png)


是不是还能看到很多熟悉的类和方法？上面这段代码的来源是 rw 的 Player.AddFood(int add) 方法。那么其实可以看出，IL代码并没有改变原本C#代码编写的类型或者方法。IL其实并不是一门独立的语言。它所作的是把你在C#内进行的大部分操作翻译为了更加适合机器阅读的格式。比如类似+=这样的语法糖，会被转换成更加原始的形式以便翻译成更加底层的语音。
___
## 2.如何看懂IL代码

IL代码除了上图在DnSpy中的形式外，还有种更普遍的形式，可以在ILSpy中看到![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/2.png)


大致观察一下就可以发现，虽然两者的格式有所区别，但本质是相同的，下面一段文本格式就是每行IL代码的基本格式。

```typescript
序号 : 操作码(指令) 操作符
```
在了解了IL代码的基本格式之后，或许你也注意到了，IL代码和C#代码并不是一 一对应的关系。实际上，绝大多数情况下，一行C#代码在翻译成IL代码后都会变成多行代码。在ILSpy中将显示模式切换为IL with C#，便可以在IL代码块上方看到注释形式的对应原始C#语句。之后展示IL代码都会在ILSpy中操作，如果是为了阅读IL代码相较于DnSpy也更推荐使用ILSpy。
___
### 简单的开始

首先我们从一个及其简单的方法和它的IL代码开始
```C#
public float Foo(float a)
{
    float b = 1f;
    b += a;
    return b;
}
```
![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/4.png)


首先看向IL代码的头部位置，它提供了两个基本信息：

函数的返回值类型是float32，第一个参数是float32类型的a。

再看向第二段代码。可以看到同样包含了两个基本信息：

.maxstack : 指定了这个函数分配到的最大堆栈大小。我们暂时不用管这个堆栈最大小是什么，但这里的**堆栈**是个很重要的概念，稍后会做介绍。

.locals init ：这里初始化了所有的局部变量。在图中可以看到一共有两个局部变量，第一个是

float32的b，第二个是float32类型的匿名变量。

至此我们就获取了一个函数在IL代码中的基本信息。
___
### IL的基本元素

得到基本信息后，我们就可以画出一个简单的结构示意图了（脑绘即可）

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/5.png)

!!! note "注意"
    这里用虚色的并不是指可以往后添加更多的参数，而是表示在别的代码里有可能会出现更多的参数。不过局部变量是可以增加的。

**参数**：包括了全部传入函数的形参。如果是非静态函数的话，那么0号参数会自动填充为该示例本身，其他参数往后顺延一位。

**局部变量**：就是，局部变量。

**堆栈**：这里的堆栈是非常重要的概念。在IL代码里，所有的变量是不能直接进行操作的，需要用专门的指令将参数或者局部变量压入堆栈，然后后续的指令才能对堆栈内的变量进行操作。完成操作后，同样需要用专门的指令将堆栈内的变量提取出来，然后存入参数\局部变量内。同时，指令只能对堆栈最顶端的变量进行操作。

在绝大多数情况下，堆栈是符合先进先出规则的。但在IL内有个例外情况。 当某个操作会一次消耗多个堆栈内的变量时（例如多参数函数的调用），会按照先进先出的顺序把堆栈内的变量对应到需求的位置上。下面的两张图说明了这个情况。

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/6.png)


![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/7.png)


**执行顺序**：不同与绝大多数的语言，IL代码是没有传统的分支和循环的。因此IL代码的执行就只有顺序执行，跳转和返回三个。分支和循环会通过一些带有条件判断或者不附带条件的跳转指令来实现。

#### 一些IL指令的介绍

熟悉了基础元素之后，接下来会介绍一些基本的指令以便你可以读懂上面那段IL代码的具体内容。

|   指令|含义                                                      |
|-------|----------------------------------------------------------|
|    nop|不进行任何操作，仅消耗处理器的时间。                         |
| ldc.r4|将操作符提供的float32类型值作为F(float)类型压入堆栈。        |
|stloc.N|将堆栈顶部的变量弹出，并储存到第N位局部变量上。               |
|ldloc.N|将第N位局部变量压入堆栈。                                   |
|    add|对堆栈内的变量取出并执行相加操作，并将结果重新压入堆栈。       |
|   br.s|无条件跳转到对应位置的语句（短格式）                         |
|    ret|返回并退出函数。如果函数有返回值，会将堆栈内的值作为返回值返回。|

这里附上一个[指令速查表](https://www.cnblogs.com/flyingbirds123/archive/2011/01/29/1947626.html)，如果遇到不认识的指令看这个就好了。
___
### 第一次解析IL代码

有了上面关于部分指令的介绍，我们就可以真正开始阅读我们的第一段IL代码了。接下来你可以尝试利用介绍的指令和图中的IL代码对照并尝试自己理解（这也是推荐的），也可以直接看下面这段解析。

这次解析假定传入a的值为2.0

|IL代码|含义|完成指令后的堆栈|局部变量|
|-|-|-|-|
|IL_0000|什么也不做||
|IL_0001|将1作为浮点数压入堆栈|1.0||
|IL_0006|将堆栈顶端的变量保存到第一个局部变量||b = 1.0|
|IL_0007|将第一个局部变量压入堆栈|1.0|b = 1.0|
|IL_0008|将第二个参数压入堆栈|1.0，2.0|b = 1.0|
|IL_0009|将堆栈内的值相加并压入堆栈|3.0|b = 1.0|
|IL_000a|将堆栈顶端的变量保存到第一个局部变量||b = 3.0|
|IL_000b|将第一个局部变量压入堆栈|3.0|b = 3.0|
|IL_000c|无条件跳转到||b = 3.0|
|IL_000d|无条件跳转到 IL_000f|||
|IL_000f|将第一个局部变量压入堆栈|3.0|b = 3.0|
|IL_0010|返回|函数返回值为3.0||

可以看到实际上由很多IL代码是没什么意义的，但这确实就是IL比较难以理解的地方，所以以后遇到这种情况忽略它就可以了。
___
### 实例相关的方法

上面的代码部分还只涉及了值类型相关的代码，接下来我们来看一些涉及实例操作的方法。

首先编写一个简单的类并且在Main函数中对他进行一些简单的操作。

``` C#
public static void Main()
{
    var testA = new Test();
    testA.a = 2;
    testA.PrintOutput(1.0f);
    float b = testA.B;
    testA.B = 3;
    Console.ReadLine();
}


public class Test
{
    public int a;
    public int B
    {
        get => a;
        set => a = value;
    }

    public Test()
    {
    }

    public virtual void PrintOutput(float a)
    {
        Console.WriteLine($"PrintOutput called with param {a}");
    }
}
```
然后用ILSpy查看Main函数生成的IL代码，可以得到如下的结果。

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/9.png)

#### 更多的IL指令

很明显上面的il代码就开始变得复杂了。因此需要先介绍一些和实例操作相关的il指令

|   指令|含义                                                       |
|-------|----------------------------------------------------------|
|    newobj|创建一个值类型的新对象或新实例，并将对象引用（O 类型）推送到计算堆栈上。|
| call|调用由操作符定义的方法。<br>详细说，其实是调用方法，如果调用的实例中的方法，注意从堆栈中读取值的顺序，第一值需要是实例本身。同时call只会调用指定的方法，而不会去查找重写方法。另外，call会假定实例本身不为null，不会进行null检查。call同样可以调用静态方法。        |
|callvirt|调用与实例关联的特定方法。<br>详细说，其实也是调用方法。但与call不同的是，callvirt只能调用实例中的方法，同时会去查找正确的重写方法。callvirt会在执行前检查实例是不是null，如果为null会抛出异常。注意：重写方法中调用基类方法一定要用call来调用，不然会形成无穷递归。               |
|stfld|将实例的字段替换为新值。<br>虽然只是对字段进行赋值的操作，但要注意这个指令和调用函数类似，对堆栈中变量提取的顺序和一般情况不同。第一个提取到的变量一定要是拥有该字段的实例本身。                                   |
|    conv|将堆栈顶端的变量转换为其他类型。<br>具体细节可以去IL指令表里查看，这里就不一一列举了。|


简单介绍完这些指令，相信上面那一段IL代码具体执行了什么操作你就能完全看懂了。
___
## 3.如何执行IL Hook

### 基本的介绍

接下来我们就可以开始实际编写一段简单的代码来对游戏代码进行IL操作了。

首先创建一个基本的rw mod项目并添加必要引用。除此之外，我们还需要引用几个额外的程序集。
```
Mono.Cecil
MonoMod.Utils
```

这两个程序集同样可以在rw的文件目录里找到。

接下来我们创建一个名叫PlayerILHook的静态类用于演示（名字叫什么是随意的）。

!!! note "注意"
    这个代码会在我的一个已有项目BetterChiTrans中进行，但不会涉及到它的具体代码，所以不用在意程序集本身的名字

与钩子函数不同的是，这次我们不能在On命名空间中寻找函数了，这次我们要切换到IL命名空间。例如我们想要修改Player.AddFood的行为，所以需要编写一下代码：

```C# title="PlayerILHooks.cs" linenums="1"
using System;
using System.Linq;
using System.Reflection;
using Mono.Cecil.Cil;
using MonoMod.Cil;
using MonoMod.Utils;
using UnityEngine;

namespace BetterChiTrans
{
    internal static class PlayerILHooks
    {
        public static void HookOn()
        {
            IL.Player.AddFood += Player_AddFood;
        }

        private static void Player_AddFood(ILContext il)
        {
            throw new NotImplementedException();
        }
    }
}
```
当然和On类似的，写完+=之后可以让VS为我们自动补全代码。

可以看到，这个函数的声明也和钩子函数有很大不同。无论你想要对什么函数进行il hook的操作，其函数形参都是这个ILContext。从它的命名可以推断出，这个类是代表了这个函数的IL代码主体，就想一个文本文件一样。

接下来如果我们想要对这个主体进行访问和修改，就像访问文本一样，我们需要申请一个指针。在函数中编写如下代码即可做到。
~~~C# title="PlayerILHooks.cs" linenums="18"
private static void Player_AddFood(ILContext il)
{
    ILCursor c = new ILCursor(il);
}
~~~


指针的数量一般没有限制。但要注意的是，在这个函数的作用域内，指针指向的地址是连续变化的。也就是说如果你移动了一次指针，那么下次移动会从上次移动后指向的地址开始计算。

那么接下来我们就可以移动这个指针了。通过对c.index的值进行修改，我们就可以轻松实现移动指针的操作。

~~~C#
c.Index++;
c.Index += 50;
~~~
!!! note "注意"
    虽然图里没有写，但指针的索引是可以减小的
___
### 查找所需的IL代码地址

尽管我们可以单纯使用一次一次移动指针的方法来定位需要的il代码，但那样会让你的代码失去兼容性。一旦有人同样对你修改的函数进行了类似的il操作，那么原本定位的指针就会指向错误的位置。因此monomod为我们提供了一些查找方法：

~~~C#
c.TryFindNext()
c.TryFindPrev()
c.TryGotoNext()
c.TryGotoPrev()
~~~

其中find会查找具体匹配的第一个索引，而goto则会在匹配的情况下自动将指针移动到对应处。一般来说使用最对的是TryGotoNext，所以接下来我会以它为例来进行说明。
~~~C# title="PlayerILHooks.cs" linenums="21"
if(c.TryGotoNext())
{

}
~~~

一个最常见的写法便是这种，当匹配成功之后会将指针移动到对应的位置，以便你后续进行操作。当然这时候你肯定会好奇这个匹配具体要怎么来写了。尽管有很多种不同的写法，但我最推荐的是利用lambda表达式的一种写法。

常见的写法入下：
~~~C# title="PlayerILHooks.cs" linenums="21"
if(c.TryGotoNext(MoveType.After,
    (i) => i.Match(OpCodes.Call),
    (i) => i.MatchLdarg(0)
    ))
{

}
~~~

其中movetype代表了指针会移动到匹配代码块的什么位置。after则是移动到其下方，默认为before，意思是移动到上方。因此这里TryGotoNext的含义为移动到匹配代码块下的向下第一个语句处。

这种写法可以提供最佳的代码可读性，其匹配顺序为从上到下依次匹配你所编写的match类语句。当所有语句均匹配成功时，会认为找到了你需要的il代码段，并且将指针移动到规定处。用图片来表示的话这里就是这样的情况：

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/17.png)


这里的Match有很多种，但不需要你每个记住。有个很简单的办法去记，Match基本是和IL指令一一对应的，例如有call就有MatchCall，有Ldfld就有MatchLdfld。普通的Match本身也可以传入OpCodes来指定匹配的指令。不过需要注意的是，这个OpCodes是位于MonoMod.Cecil.Cil下的那个，而非System.Reflection.Emit下的。不过后面这位以后可以用的上。

接下来用ILSpy查看我们需要修改的函数的IL代码。

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/18.png)


对应的源代码为

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/19.png)


假设我们想对框中的代码进行修改，首先我们需要定位到这一段对应的il代码。

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/20.png)


假如这里我们想要在玩家添加饱食度的时候获取到玩家的最大饱食度上限，同时让这个上限在参与运算时减少2，应该怎么做呢？

### 对IL代码执行添加或修改

最常见的两种添加操作为下：
~~~C#
c.Emit()
c.EmitDelegate()
~~~
![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/21.png)


第一个比较好理解，就是直接在当前指针位置插入一段IL指令，具体情况如图所示：

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/22.png)


第二个相对复杂一些，是直接插入一段委托。这里可以用lambda或者匿名委托来做参数，无需额外声明一个函数，不过如果复用率较高的话也可以额外声明。具体插入的不止一段IL代码，但基本情况和上面是类似的。

对于我们这里的情况，我们需要读取一个数，输出日志并修改，那么最好的情况就是使用EmitDelegate。

看到这里

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/23.png)


原本的代码调用了ldarg0并call了一个属性的get方法消耗掉了堆栈顶端的实例变量，同时压入了一个int32类型的变量，这个变量就是我们需要的值。因此我们需要将指针定位到这段代码的下方。

接着观察上方的语句

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/24.png)


这里有一个br语句，两个ldarg语句，一个call，因此我们可以利用他们来进行匹配。编写如下代码：
~~~C# title="PlayerILHooks.cs" linenums="21"
if (c.TryGotoNext(MoveType.After,
    (i) => i.Match(OpCodes.Br),
    (i) => i.MatchLdarg(1),
    (i) => i.MatchLdarg(0),
    (i) => i.Match(OpCodes.Call)
))
{
}
~~~

当然，你可以在if内加一个debug来判断是不是真的匹配到了东西。

~~~ C# title="PlayerILHooks.cs" linenums="21" hl_lines="8"
if (c.TryGotoNext(MoveType.After,
    (i) => i.Match(OpCodes.Br),
    (i) => i.MatchLdarg(1),
    (i) => i.MatchLdarg(0),
    (i) => i.Match(OpCodes.Call)
))
{
    Debug.Log("MatchFind");
}
~~~

编译程序集并运行游戏，就可以看到这段输出，这说明我们的il已经查找到了匹配的il代码块。

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/27.png)


查找成功后我们就可以进行操作了。

首先可以知道的是，我们需要的函数有一个int类型的返回值，一个int类型的参数，为了方便debug最好能把player实例也输入函数内，所以还需要一个Player类型的参数。那么很明显这是个Func<int,Player,int>类型的委托。至于为什么是先int后player，可以看到原本的代码调用call后堆栈顶端已经是一个int类型的变量，如果我们要再压入player实例，那么int会在堆栈相对靠下的位置。如果还不明白可以去看看IL基本信息中关于执行顺序的描述。

因此基于以上的思考，我们可以编写如下代码来实现我们的功能。
~~~C# title="PlayerILHooks.cs" linenums="28"
Debug.Log("MatchFind");
c.Emit(OpCodes.Ldarg_0);
c.EmitDelegate<Func<int, Player, int>>((origMaxFood, self) =>
{
    Debug.Log($"Player orig maxFoodInStomach {origMaxFood}");
    return origMaxFood - 2;
});
~~~
变量名称是随意的，不过这里我没使用到self变量，如果你想要打印更多信息的话可以使用它。

（补充一点：这里的Func是指有返回值的委托，泛型实参的最后一个会默认为返回值。如果只有一个泛型实参，则为有一个返回值没有参数的委托。如果需要无返回值的话，请使用Action类型的委托。）

然后打开游戏运行测试，可以看到实现了我们想要的效果。

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/C%23%E7%BC%96%E7%A8%8B/IL%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B/29.png)


### 一些补充

1.除了上述方法外，还有另外一些相对不那么常用的方法可以修改IL代码。

c.Next.Operand = xxx / c.Prev.Operand = xxx：直接对指令后的操作符进行修改。

c.Remove() 移除下方的语句。

2.对于一些特殊的IL指令，例如BrTrue这种执行跳转的语句，不能直接在Emit时传入跳转的索引。这是因为在我们执行ILHook的时候，IL代码本身并没有被实际创建，而IL代码的索引也是不确定的。这时候需要先用当前指针或者利用另一个指针查找到我们需要跳转到的语句，然后调用 ILCursor.MarkLabel()并用变量保存。之后Emit跳转指令的时候便可以使用这个ILLabel作为跳转对象。


## 4.一些更多应用的补充

当然除了在mod中使用IL代码外，还有别的地方也可以使用IL代码。这里介绍其中一个运用方式：利用IL来优化反射的性能。

这里直接使用之前写过的一段代码来做例子
~~~C# linenums="1"
    static Dictionary<Type, Func<string, bool>> typeCheckers = new();
    static Dictionary<Type, Func<LingoDataType>> typeCtors = new();

    public static void RegistLingoType(Type type)
    {
        if (typeCheckers.ContainsKey(type))
            return;

        //创建检查方法委托
        DynamicMethod checkerMethod = new DynamicMethod("checkCaller", typeof(bool), new Type[] { typeof(string) }, true);
        MethodInfo origMethod = type.GetMethod("CheckType", BindingFlags.Public | BindingFlags.Static);
        if (origMethod == null)
            throw new NullReferenceException($"类型{type}并没有实现名为CheckType的方法");

        ILGenerator il = checkerMethod.GetILGenerator();
        il.Emit(OpCodes.Ldarg_0);
        il.Emit(OpCodes.Call, origMethod);
        il.Emit(OpCodes.Ret);

        var del = (Func<string, bool>)checkerMethod.CreateDelegate(typeof(Func<string, bool>));


        //创建构造方法委托
        DynamicMethod ctorMethod = new("typeCtor", typeof(LingoDataType), null, true);
        ConstructorInfo origCtor = type.GetConstructor(new Type[0]);
        if (origCtor == null)
            throw new NullReferenceException($"类型{type}并没有正确格式的构造函数");

        ILGenerator il2 = ctorMethod.GetILGenerator();
        //il2.Emit(OpCodes.Ldarg_0);
        il2.Emit(OpCodes.Newobj, origCtor);
        il2.Emit(OpCodes.Ret);

        var del2 = (Func<LingoDataType>)ctorMethod.CreateDelegate(typeof(Func<LingoDataType>));

        typeCheckers.Add(type, del);
        typeCtors.Add(type, del2);
    }
~~~
这个代码创建了两个动态反射委托，每一个都可以用简单的语言来描述其过程。

首先创建一个DynamicMethod变量的实例，这个会返回一个空的动态方法。其方法参数和返回值类型在构造函数中指定。对于这个例子我们想要调用一个函数的方法，那么这个动态函数的返回值要与目标函数对应，参数除了目标函数的所有参数外，还要额外附加函数定义类型的实例。

接下来利用反射拿到指定类型的函数信息。

给先前创造的DynamicMethod申请一个IL构造器。

添加IL代码，并在IL代码中执行反射获取到的函数。

完成DynamicMethod的构造，并申请一个对应的委托。之后方法的调用就由该委托来执行，然后在一个地方保存你的委托。

利用这个方法可以大幅提升需要经常使用的反射代码的性能。


