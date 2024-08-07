<h1>BuffCore</h1>

包含各种Buff相关函数的静态类

<h2>方法</h2>

```csharp
//尝试获取ID对应的Buff
//不存在则返回false
public static bool TryGetBuff(BuffID id,out IBuff buff);
public static bool TryGetBuff<BuffType>(BuffID id,out BuffType buff);
```

```csharp
//获取ID对应的Buff
//不存在则返回null
public static IBuff GetBuff(this BuffID id);
public static BuffType GetBuff<BuffType>(this BuffID id);

```

```csharp
//获取BuffData
//如果不存在则返回 null
public static BuffData GetBuffData(this BuffID id);
public static BuffDataType GetBuffData<BuffDataType>(this BuffID id);
```

```csharp
//获取卡牌的静态属性
public static BuffStaticData GetStaticData(this BuffID id)
```

```csharp
//获取全部启用的Buff ID
//如果当前没有存档则为null
public List<BuffID> GetAllBuffIds();
```

```csharp
//判断当前是否是RandomBuff游戏模式
public bool BuffMode(this RainWorld rainworld);
```

```csharp
//删除或减少卡牌叠层
public static bool UnstackBuff(this BuffID id);
```

```csharp
//创建新的Buff。
//在存在同IDbuff的情况下如果needstack == true，则会尝试增加卡牌叠层，反之则不进行任何处理
public static IBuff CreateNewBuff(this BuffID id, bool needStack = true);
public static TBuff CreateNewBuff<TBuff>(this BuffID id, bool needStack = true);
```

