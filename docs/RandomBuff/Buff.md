<h1>Buff&lt;TBuff,TData&gt;</h1>
单轮回内创建的实例，负责实际的运行逻辑。
会在单轮回结束时销毁。

<h2>方法</h2>

```csharp
//当点击（或按键）触发时调用
//仅对可触发的增益有效（Triggerable == true）
public abstract bool Trigger(RainWorldGame game);
```

```csharp
//增益的更新方法，与RainWorldGame.Update同步
public abstract void Update(RainWorldGame game);
```

```csharp
//增益的更新方法，与RainWorldGame.Update同步
public abstract void Update(RainWorldGame game);
```

```csharp
// 增益的销毁方法，当该增益实例被移除的时候会调用
// 注意：当前轮回结束时会清除全部的Buff物体
public abstract void Destroy();
```
```csharp
// 强制触发增益效果，一般用于显示HUD反馈
public void TriggerSelf(bool ignoreCheck = false)
```

```csharp
// 创建临时卡牌池，会在轮回末销毁
public TemporaryBuffPool GetTemporaryBuffPool();
```

```csharp
// 获取绑定的按键
// !!!注意请使用BuffInput的方法进行按键判断!!!
public string GetBindKey();
```
<h2>属性</h2>


```csharp
// 是否可以被触发（在info里Triggerable为true的前提下才生效）
public virtual bool Triggerable { get; }
```

```csharp
// 触发后是否处于激活状态
public virtual bool Active { get; }
```

```csharp
//增益数据对应的BuffID
public abstract BuffID ID { get; }
```

```csharp
//增益对应的数据
public TData Data { get; }
```

```csharp
//增益对应的实例（可能为空！）
public static TBuff Instance { get; }
```


```csharp
//卡牌的计时器，在构造方法中赋值后，不可为null
//赋值后可以在任意时机修改值
public BuffTimer MyTimer { get; set; }
```


