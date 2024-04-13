<h1>class BuffCardRenderer : MonoBehaviour</h1>
卡牌渲染器，用于管理渲染相关的组件并处理卡牌在场景中的GameObject。

<h2>字段</h2>

```csharp
//该渲染器当前渲染卡牌的静态信息
internal BuffStaticData _buffStaticData;
```

```csharp
//该渲染器的唯一id，用于在CardRenderManager中管理
internal int _id;
```

```csharp
//卡面的贴图
internal Texture _cardTextureFront;
```

```csharp
//卡背的贴图
internal Texture _cardTextureBack;
```

```csharp
//该渲染器的相机管理器
internal CardCameraController cardCameraController;
```

```csharp
//卡面的高光控制器
internal CardHighlightController cardHighlightFrontController;
```

```csharp
//卡背的高光控制器
internal CardHighlightController cardHighlightBackController;
```

```csharp
//卡牌正面的文本渲染器
internal CardTextController cardTextFrontController;
```

```csharp
//卡牌背面的文本渲染器
internal CardTextController cardTextBackController;
```

```csharp
//卡牌堆叠数文本渲染器
internal CardNumberTextController cardStackerTextController;
```

```csharp
//卡牌剩余轮回数文本渲染器
internal CardNumberTextController cardCycleCounterTextController;
```

```csharp
//卡牌按键绑定文本渲染器
internal CardKeyBinderTextController cardKeyBinderTextController;
```

```csharp
//卡牌当前的法线方向，由渲染器自动计算
internal Vector3 normal;
```
<h2>属性</h2>

```csharp
//卡牌渲染器中卡牌物体与相机的距离，大部分情况下保持默认值即可
public float Depth {get; set;}
```

```csharp
//控制卡牌的灰度效果，true时卡牌为低饱和度
public bool Grey {get; set;}
```

```csharp
//控制卡牌是否显示标题文本
public bool DisplayTitle {get; set;}
```

```csharp
//控制卡牌是否显示介绍文本
public bool DisplayDiscription {get; set;}
```

```csharp
//控制卡牌是否显示高光效果
public bool EdgeHighlight {get; set;}
```

<h2>方法</h2>

```csharp
//卡牌渲染器的初始化方法，调用时需要分配一个不会冲突的id，在CardRendererManager中自动调用
public void Init(int id, BuffStaticData buffStaticData)
```