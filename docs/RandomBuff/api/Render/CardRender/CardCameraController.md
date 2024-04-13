<h1>class CardCameraController : MonoBehaviour</h1>
卡牌渲染器组件，用于控制渲染卡牌的相机。

<h2>字段</h2>

```csharp
//当前相机的渲染模板
public RenderTexture targetTexture;
```

```csharp
//相机
public Camera myCamera;
```

<h2>属性</h2>

```csharp
//通过赋值控制相机是否启用渲染，只在必要的时候（比如卡牌发生变换）启用一帧渲染。
public bool CardDirty {get; set;}
```

<h2>方法</h2>

```csharp
//初始化方法
public void Init(int id)
```