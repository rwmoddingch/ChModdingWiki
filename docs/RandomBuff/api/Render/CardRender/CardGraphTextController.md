<h1>class CardGraphTextController : MonoBehaviour</h1>
卡牌渲染器组件，卡牌中渲染装饰字的基类。

<h2>字段</h2>

```csharp
//内层图形采用的黑色
public static Color paleBlack
```

```csharp
//关联的卡牌渲染器
protected BuffCardRenderer _renderer;
```

```csharp
//锚点物体
protected GameObject _hoverPoint;
```

```csharp
//外层图形物体
protected GameObject _graphOuter;
```

```csharp
//内层图形物体
protected GameObject _graphInner;
```

```csharp
//关内层图形物体的网格渲染器，用来进行动画控制
protected MeshRenderer _graphInnerRenderer;
```

```csharp
//文本渲染物体
protected GameObject _stackerTextObj;
```

```csharp
//文本网格
protected TextMesh graphTextMesh;
```

```csharp
//初始缩放，用来进行动画控制
protected Vector3 _origScale;
```

```csharp
//装饰字的目标宽度，用来进行动画控制
protected float _targetWidth;
```

```csharp
//装饰字的当前宽度，用来进行动画控制
protected float _currentWidth;
```
<h2>属性</h2>

```csharp
//控制该装饰字是否显示，具有动画效果
public bool Show {get; set;}
```

<h2>方法</h2>

```csharp
//初始化方法
//renderer：关联的卡牌渲染器
//parent：绑定物体的变换组件
//font：文本网格使用的字体
//color：外层图形的颜色
//text：初始化文本
//rotation：内外层图形的旋转量
//primitiveType：内外层图形采用的形状
//posFactor：该装饰字位于卡牌边界垂直位置的分量，介于-1~1之间
public virtual void Init(BuffCardRenderer renderer, Transform parent, Font font, Color color, string text, Vector3 rotation, InternalPrimitiveType primitiveType = InternalPrimitiveType.Quad,  float posFactor = 0.309f)
```

```csharp
//需要更改文本时调用
public virtual void UpdateText()
```

```csharp
//更新方法，每帧调用
public virtual void Update()
```

```csharp
//自动化创建指定形状的物体
protected GameObject CreatePrimitiveObject(InternalPrimitiveType internalPrimitiveType)
```

```csharp
//自动化创建指定形状的物体
protected GameObject CreatePrimitiveObject(InternalPrimitiveType internalPrimitiveType)
```

```csharp
//自动化创建指定参数的圆形面片物体
//radius：半径
//segments：圆形边的段数
//centerCircle：圆心的偏移量
 protected GameObject CreatePrimitiveCircle(float radius, int segments, Vector3 centerCircle)
```

```csharp
//自动化创建指定形状的物体
protected GameObject CreatePrimitiveObject(InternalPrimitiveType internalPrimitiveType)
```

<h1>class CardKeyBinderTextController : CardGraphTextController</h1>
用于渲染卡牌按键绑定的装饰字类

<h2>属性</h2>

```csharp
//当前绑定按键的字符串
public string BindKey {get; set;}
```

```csharp
//控制图形是否闪烁
public string BindKey {get; set;}
```
<h2>方法</h2>

```csharp
//同基类
 public override void Init(BuffCardRenderer renderer, Transform parent, Font font, Color color, string text, Vector3 rotation, InternalPrimitiveType primitiveType = InternalPrimitiveType.Quad, float posFactor = 0.309F)
```

```csharp
//同基类，在没有按键绑定时将文本设置为 “>>” 来提醒玩家可以进行按键绑定
public override void UpdateText()
```

```csharp
//同基类
protected override void Update()
```

<h1>class CardNumberTextController : CardGraphTextController</h1>
用于渲染卡牌数字装饰字的类，目前用于显示剩余轮回数和堆叠数

<h2>属性</h2>

```csharp
//当前数字的值
public string Value {get; set;}
```

```csharp
//是否显示+1符号，用于在选卡界面提示玩家选择后堆叠数加一
public bool AddOne {get; set;}
```


```csharp
//同基类，在数字小于等于0且AddOne为true时候仅显示“+1"
public override void UpdateText()
```
<h1>enum CardGraphTextController.InternalPrimitiveType</h1>

```csharp
Quad,//矩形面片
Circle,//圆形
Hexagon//六边形
```