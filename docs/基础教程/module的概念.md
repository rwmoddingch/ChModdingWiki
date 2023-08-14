---
comments: true
---
## 什么是module

相信你在使用hook的时候时常会思考一个问题：既然hook是针对这个类型所有的实例生效的，那么我该怎么区分每个实例呢？如果我想要给每个实例保存一份属于它自己的数据要怎么办呢？这个时候便可以请出我们的module了。

首先应当介绍一下module到底是什么。其实module并不是一个严格意义上的代码术语，而是一种经验写法。Module主要包含三个部分：1.绑定的目标实例。2.与目标实例关联的数据。3.与目标实例关联的方法。

由上面的定义可以看出，一个module类所起到的作用便是建立实例和附加数据与方法的桥梁。这也决定了module的实际形式十分多变，具体需要哪些代码完全取决于你设计它的目的。

## 如何编写一个常见的module类

接下来会详细介绍编写module类的基础方法。因为这个类在雨世界mod编写的过程中主要用在Player类上，我们就用Player类来作为我们将要编写的module类的目标类型。

### 类型定义

首先我们定义两个类型。鉴于我们的目标是Player类型，那么首先便是需要一个PlayerModule。其次我们还需要一个管理PlayerModule的静态类，因此可以编写一个PlayerModuleManager。如果你喜欢的话，可以把PlayerModule定义为PlayerModuleManager的内部类。

其次很容易想到，既然我们的PlayerModule是要服务于Player实例的，那么便可以在构造函数里传入Player实例，并且在PlayerModule中保存这个实例的引用以便后续访问。同时可以在PlayerModuleManager中建立一个Player和module相关的字典，以便在别的地方根据Player访问到我们的module类。

按照以上的想法，便可以很容易想到如下的代码编写方式：

```c# linenums="1"
internal static class PlayerModuleManager
{
    public static Dictionary<Player, PlayerModule> PlayerModules = new Dictionary<Player, PlayerModule>();

    internal class PlayerModule
    {
        Player player;
        public PlayerModule(Player player)
        {
            this.player = player;
        }
    }
}
```
### 为什么是弱引用

但是呢，实际上我们并不应该编写上面的代码，而应该尽量使用弱引用来保存与Player和module相关的引用。为什么呢？首先我们需要了解什么是弱引用什么是强引用，如果你很了解这个概念，可以跳过这个介绍部分继续看后续内容。

#### 什么是弱引用

当我们实例化一个对象时，直接引用了这个对象就是强引用。在这个对象被强引用的时，GC(垃圾回收系统)无法回收这个对象。只有当该对象所有的强引用都释放的时候，GC才会回收该对象。而弱引用则很简单，它可以在保持对对象的访问的同时，允许GC回收对象。

了解完定义后，便可以很容易想到为什么要使用弱引用了。使用弱引用可以允许游戏回收不必要的Player对象，防止游戏的内存垃圾堆积过多。而PlayerModule和Player中往往都会保存大量占用内存的数据或引用，因此要尽可能的使用弱引用。

那么接下来就是要介绍两个和弱引用相关的常用类型了。

#### 弱引用的相关类型

1.WeakReference<T>

代表了一个对T类型的弱引用。

创建一个弱引用的过程如下：

```c#
Player playerObj;
WeakReference<Player> playerRef = new WeakReference<Player>(playerObj);
```
或
```c#
playerRef.SetTarget(playerObj);
```

访问引用的方法如下：

```c#
playerRef.TryGetTarget(out player);
```
除了获得playerRef弱引用绑定的实例外，这个方法还有个bool类型的返回值，用来帮助你判断是否真的获取到了实例。如果这个实例已经被GC回收，则会返回false，同时player的值也是null；如果没有被回收，则会返回true，并且player就是你需要访问的实例本身。
2.ConditionalWeakTable<TKey,TValue>

使用该类型首先需要引用System.Runtime.CompilerServices，在代码头部使用using语句引用即可。

虽然名字看上去很长，但这个类型实际上就是一个弱引用字典，相比于字典，它只能用TryGetValue来访问键值对，不能对其中的键值对进行迭代访问。

创建一个弱引用字典的过程如下：

```c#
ConditionalWeakTable<Player, PlayerModule> playerModules = new ConditionalWeakTable<Player, PlayerModule>();
```

往弱引用字典中添加键值对的方法如下：

```c#
Player player;
PlayerModule playerModule;
playerModules.Add(player, playerModule);
```

访问弱引用字典中键值对的方法如下：

```c#
playerModules.TryGetValue(player, out playerModule);
```
与WeakReference类似，这个方法也有个bool类型的返回值，功能也和WeakReference相同，就不再过多介绍了。
#### 使用弱引用

了解了方法，我们便可以用弱引用来重写PlayerModuleManager和PlayerModule类了。

代码如下：

```c# title="PlayerModules.cs" linenums="5" hl_lines="3 7 10"
internal static class PlayerModuleManager
{
    public static ConditionalWeakTable<Player, PlayerModule> playerModules = new ConditionalWeakTable<Player, PlayerModule>();
    
    internal class PlayerModule
    {
        WeakReference<Player> playerRef;
        public PlayerModule(Player player)
        {
            playerRef = new WeakReference<Player>(player);
        }
    }
}
```

### 使用module

在有了目前的代码之后，我们便可以开始使用module类了。一般来说，module类的生命周期和它的目标类型是相同的。因此对于我们的PlayerModule，我们需要在Player类的实例被创建时，同时创建一个module类，并且通过manager建立它们之间的连接。

鉴于上述的思想，我们首先需要hook Player类的构造函数，也就是ctor方法。在hook函数里，调用原函数后，我们紧接着创建module类的实例，并且在manager的playermodules中添加键值对：

```c# title="PlayerModules.cs"
internal static class PlayerHooks
{
    public static void HookOn()
    {
        On.Player.ctor += Player_ctor;
    }

    private static void Player_ctor(On.Player.orig_ctor orig, Player self, AbstractCreature abstractCreature, World world)
    {
        orig.Invoke(self, abstractCreature, world);
        PlayerModuleManager.playerModules.Add(self,new PlayerModuleManager.PlayerModule(self));
    }
}
```
假如我们想要在Player.Update时访问与它绑定的module，便可以通过下面的方法进行访问：
```c# title="PlayerModules.cs"
    public static void HookOn()
    {
        On.Player.ctor += Player_ctor;
        On.Player.Update += Player_Update;
    }

    private static void Player_Update(On.Player.orig_Update orig, Player self, bool eu)
    {
        orig.Invoke(self, eu);
        if(PlayerModuleManager.playerModules.TryGetValue(self,out var module))
        {
            //做些什么
        }
    }
```

### module类的简单实践

没错，module类本身其实是个非常简单的概念。在讲完概念之后，我们可以通过一个简单的实践来熟悉它的用法。首先要确定一下我们的需求：给指定类型的猫猫添加一个技能，允许它在玩家进行跳跃并按下另一个按键的时候实施一个超级跳跃，同时伴随有冲击波特效。技能的效果和冷却时间也因猫猫类型而异。

也许这个需求看上去有点复杂，但不用慌张，首先我们分解一下需求，明确一下我们需要实现哪些东西。

首先便是初始化部分，需要根据猫猫的类型来判断是否开启技能，以及确定一些关系技能效果和冷却时间的只读变量。

其次是技能的具体实现部分，首先需要读取输入，需要生成冲击波，基于给定的变量给予玩家额外速度以及重置冷却。

现在我们明确了需要实现哪些东西后，便可以基于上文的PlayerModule开始动手了。当然我们鼓励自己动手试一试，如果你更愿意跟着教程，那么也可以直接往下看。

#### 数据的初始化

首先我们来实现第一个需求：建立和技能相关的数据变量。

可以明确的是，我们需要读取猫猫的类型，然后再根据类型来进行一些判断。那么我们便可以编写一个SetUpModule的函数，并且把猫猫的类型作为参数传入该函数。同时我们需要三个变量： 

|变量名|变量类型|作用|
|:----|:----|:----|
|canSuperJump|bool|决定猫猫是否能进行超级跳|
|superJumpEffect|float|超级跳的效果|
|jumpCoolDown|int|超级跳的冷却时间|

此外为了让冷却时间可以正确工作我们还需要一个计时器变量coolDown，类型同样为int。

假设我们想让黄猫无法进行超级跳，同时红猫的超级跳效果比白猫更好、冷却时间更长，那么我们可以将PlayerModule类改写成如下形式：

```c#  title="PlayerModules.cs" linenums="9"
    internal class PlayerModule
    {
        WeakReference<Player> playerRef;

        //超级跳设置
        bool canSuperJump;
        float superJumpEffect;
        int jumpCoolDown;

        int coolDown;//冷却计时器

        public PlayerModule(Player player)
        {
            playerRef = new WeakReference<Player>(player);
            SetUp(player.SlugCatClass);
        }

        void SetUp(SlugcatStats.Name name)
        {
            canSuperJump = name != SlugcatStats.Name.Yellow;

            if(name == SlugcatStats.Name.Red)
            {
                superJumpEffect = 10f;
                jumpCoolDown = 80;
            }
            else
            {
                superJumpEffect = 5f;
                jumpCoolDown = 40;
            }
        }
    }
```
我们在module的构造函数中调用了SetUp函数并完成了初始化。此后便不应该再改变和设置相关的三个变量的值了。
#### 完善计时器

根据对计时器的介绍，我们应该知道计时器必须要在Update函数中才可以正常工作。因此我们需要一个和Player.Update对应的函数。因此在module类中编写一个OnPlayerUpdate函数，以表明这个函数会在Player.Update中调用。同时在其中完成计时器的编写：

```c#  title="PlayerModules.cs"
        public void OnPlayerUpdate(Player player)
        {
            if(coolDown > 0)
                coolDown--;
        }   
```
同时添加一个Player.Update的hook，并在其中通过上文介绍的方式调用module中的OnPlayerUpdate方法。
```c#  title="PlayerModules.cs"
    public static void HookOn()
    {
        On.Player.ctor += Player_ctor;
        On.Player.Update += Player_Update;
    }

    private static void Player_Update(On.Player.orig_Update orig, Player self, bool eu)
    {
        orig.Invoke(self, eu);
        if(PlayerModuleManager.playerModules.TryGetValue(self,out var module))
        {
            module.OnPlayerUpdate(self);
        }
    }
```
!!! note "注意"
    这里因为我们将要在Player.Update中调用OnPlayerUpdate方法，因此可以直接通过Player.Update获取到Player实例，此时不经由playerRef访问同样是可以的。

#### 进行超级跳

因为这个技能是触发型的，所有的效果在触发的那一瞬间便完成了，因此我们可以用一个函数来管理我们的效果。

在PlayerModule中编写一个SuperJump函数：

```c#  title="PlayerModules.cs"
        public void SuperJump(Player player)
        {
            foreach (var bodychunk in player.bodyChunks)
                bodychunk.vel += Vector2.up * superJumpEffect;
        }
```
超级跳的实现方式很简单，只需要给player的每个bodychunk添加一个向上的速度即可。
接下来完成冲击波特效的部分。这部分也很简单，只需要在player的位置添加一个冲击波即可。

完成后的代码如下：

```c# title="PlayerModules.cs"
        public void SuperJump(Player player)
        {
            foreach (var bodychunk in player.bodyChunks)
                bodychunk.vel += Vector2.up * superJumpEffect;
            player.room.AddObject(new ShockWave(player.DangerPos, 20 * superJumpEffect, superJumpEffect, 5));
            coolDown = jumpCoolDown;
        }
```
完成了超级跳的功能，我们还需要一个地方来触发它。因为需要在玩家进行跳跃的时候触发超级跳，因此我们需要在Player.Jump中调用一个TrySuperJump函数。
此时我们还需要一个额外的方法来帮助我们判断玩家是否可以进行跳跃，在这个方法中我们需要同时判断计时器状态，先前设置的数据以及玩家本身的状态，因此我们可以编写如下的代码：

```c# title="PlayerModules.cs"
        bool CanSuperJump()
        {
            return canSuperJump &&
                   coolDown == 0;
        }
```
然后我们可以编写TrySuperJump函数：
```c# title="PlayerModules.cs"
        public void TrySuperJump(Player player)
        {
            if (CanSuperJump())
                SuperJump(player);
        }
```
最后我们在Player.Jump的hook中调用TrySuperJump：
```c# title="PlayerModules.cs"
    public static void HookOn()
    {
        On.Player.ctor += Player_ctor;
        On.Player.Update += Player_Update;
        On.Player.Jump += Player_Jump;
    }

    private static void Player_Jump(On.Player.orig_Jump orig, Player self)
    {
        orig.Invoke(self);
        if (PlayerModuleManager.playerModules.TryGetValue(self, out var module))
        {
            module.TrySuperJump(self);
        }
    }
```

完成后在OnModInit中调用HookOn方法，编译并进入游戏启用mod，便可以看到超级跳效果按照预期的方式生效了。

![图片](https://rwmoddingch.github.io/ChModdingWiki/assets/%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/module%E7%9A%84%E6%A6%82%E5%BF%B5/1.png)


完整代码如下：

```c# title="PlayerModules.cs" linenums="1"
using System;
using System.Runtime.CompilerServices;
using UnityEngine;

internal static class PlayerModuleManager
{
    public static ConditionalWeakTable<Player, PlayerModule> playerModules = new ConditionalWeakTable<Player, PlayerModule>();
    
    internal class PlayerModule
    {
        WeakReference<Player> playerRef;

        //超级跳设置
        bool canSuperJump;
        float superJumpEffect;
        int jumpCoolDown;

        int coolDown;//冷却计时器

        public PlayerModule(Player player)
        {
            playerRef = new WeakReference<Player>(player);
            SetUp(player.SlugCatClass);
        }

        void SetUp(SlugcatStats.Name name)
        {
            canSuperJump = name != SlugcatStats.Name.Yellow;

            if(name == SlugcatStats.Name.Red)
            {
                superJumpEffect = 10f;
                jumpCoolDown = 80;
            }
            else
            {
                superJumpEffect = 5f;
                jumpCoolDown = 40;
            }
        }

        public void OnPlayerUpdate(Player player)
        {
            if(coolDown > 0)
                coolDown--;
        }

        public void SuperJump(Player player)
        {
            foreach (var bodychunk in player.bodyChunks)
                bodychunk.vel += Vector2.up * superJumpEffect;
            player.room.AddObject(new ShockWave(player.DangerPos, 20 * superJumpEffect, superJumpEffect, 5));
            coolDown = jumpCoolDown;
        }

        public void TrySuperJump(Player player)
        {
            if (CanSuperJump())
                SuperJump(player);
        }

        bool CanSuperJump()
        {
            return canSuperJump &&
                   coolDown == 0;
        }
    }
}

internal static class PlayerHooks
{
    
    public static void HookOn()
    {
        On.Player.ctor += Player_ctor;
        On.Player.Update += Player_Update;
        On.Player.Jump += Player_Jump;
    }

    private static void Player_Jump(On.Player.orig_Jump orig, Player self)
    {
        orig.Invoke(self);
        if (PlayerModuleManager.playerModules.TryGetValue(self, out var module))
        {
            module.TrySuperJump(self);
        }
    }

    private static void Player_Update(On.Player.orig_Update orig, Player self, bool eu)
    {
        orig.Invoke(self, eu);
        if(PlayerModuleManager.playerModules.TryGetValue(self,out var module))
        {
            module.OnPlayerUpdate(self);
        }
    }

    private static void Player_ctor(On.Player.orig_ctor orig, Player self, AbstractCreature abstractCreature, World world)
    {
        orig.Invoke(self, abstractCreature, world);
        PlayerModuleManager.playerModules.Add(self,new PlayerModuleManager.PlayerModule(self));
    }
}
```

