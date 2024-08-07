
<h1>BuffData</h1>

增益的数据类型，保存存档内数据，也可以获取配置属性。
不会随着单局开始结束而重复创建，仅在更换存档和启动游戏时候创建。

<h2>方法</h2>


```csharp
//轮回结束时触发
public virtual void CycleEnd(); 
```


```csharp
//重复选取时会被调用
public virtual void Stack(); 
```

```csharp
//减少堆叠次数或删除（非堆叠卡）的时候被调用
public virtual void UnStack(); 
```

```csharp
//当存档数据读取后调用
//可以用于数据初始化
public abstract void DataLoaded(bool newData); 
```


<h2>属性</h2>

```csharp
//增益数据对应的BuffID
public abstract BuffID ID { get; }
```

```csharp
//堆叠层数，可以替换为自己的get,set
public virtual int StackLayer { get; set; }
```

```csharp
// 如果为true，则在周期结束时自动移除本增益
public bool NeedDeletion { get; set; }
```


存档数据
```csharp
[JsonProperty]
public AnyType saveInWorld; //创建一个想要保存到猫的游戏存档的数据
```




