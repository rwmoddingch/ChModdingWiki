<h1>class CardHighlightController : MonoBehaviour</h1>
卡牌渲染器组件，用于控制卡面的特效。


<h2>属性</h2>

```csharp
//控制是否启用边缘高光
internal bool EdgeHighlight {get; set;}
```
```csharp
//控制卡面是否降低饱和度
internal bool Grey {get; set;}
```
<h2>方法</h2>

```csharp
//初始化方法，提供卡面贴图和关联的卡牌渲染器
public void Init(BuffCardRenderer buffCardRenderer, Texture texture)
```