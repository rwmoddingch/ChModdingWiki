<h1>BuffRegister</h1>

负责注册增益。是静态类（真的感觉没的介绍的东西了）

<h2>方法</h2>

<h3>注册增益</h3>

```csharp
//注册新的增益
//HookType 需要存在静态的HookOn函数或者LongLifeCycleHookOn函数
public static void RegisterBuff<BuffType, DataType, HookType>(BuffID id);
```

```csharp
//注册新的增益
public static void RegisterBuff<BuffType, DataType>(BuffID id);
```

<h3>注册条件</h3>

```csharp
//注册新的条件
//其中显示名称暂未使用
//parentId为继承的id，会继承父类的冲突配置
public static void RegisterCondition<BuffType, DataType>(ConditionID id, string displayName,bool isHidden, ConditionID parentId = null);
```

```csharp
//注册新的条件
//banlist为禁止项，注册的条件不会在banlist对应的模式被抽取（包括parentId对应的）
public static void RegisterCondition<TConditionType>(ConditionID id, string displayName, ConditionID parentId = null,params GachaTemplateID[] banList)
```

<h3>注册抽卡模式</h3>

```csharp
// 注册新的抽卡模式
// banlist为禁止的条件（和上文注册Condition只需要有一个ban条件存在即可）
public static void RegisterGachaTemplate<TTemplateType>(GachaTemplateID id, params ConditionID[] banList);
```

<h3>注册使命</h3>

```csharp
// 注册新的使命
// !!!必须要在IMissionEntry.RegisterMission内调用!!!
public static void RegisterMission(MissionID ID, Mission mission);
```

```csharp
// 注册新的任务种类
public static void RegisterQuestType<TQuestCondition>();
```

```csharp
// 注册新的装饰解锁要素
public static void RegisterCosmeticUnlock<TCosmeticUnlock>();
```