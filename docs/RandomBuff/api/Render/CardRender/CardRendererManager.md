<h1>internal static class CardRendererManager</h1>
卡牌渲染器控制器，通过池化管理所有卡牌渲染器

<h2>方法</h2>

```csharp
//获取渲染指定ID卡牌的卡牌渲染器，当对象池中没有可用渲染器时才会创建新的
public static BuffCardRenderer GetRenderer(BuffID buffID)
```

```csharp
//回收管理不使用的卡牌渲染器
public static void RecycleCardRenderer(BuffCardRenderer buffCardRenderer)
```

```csharp
//维护一个有时间限制的对象池，当不活跃程度超过一定时间后，将会销毁该对象。在BuffPlugin.Update中调用
public static void UpdateInactiveRendererTimers(float deltaTime)
```

```csharp
//一次性销毁对象池中的所有对象
public static void DestroyAllInactiveRenderer()
```

<h1>internal static class CardBasicAssets</h1>
古希腊掌管卡牌渲染资源的静态类

<h2>字段</h2>

```csharp
//卡牌渲染贴图的标准尺寸
public static readonly Vector2Int RenderTextureSize = new Vector2Int(600, 1000);
```

<h2>属性</h2>

```csharp
//卡牌高光shader
public static Shader CardHighlightShader { get; private set; }
```

```csharp
//卡牌字体shader，为了让字体可以正常显示对基础shader做了一定调整
public static Shader CardTextShader { get; private set; }
```

```csharp
//卡面shader，为了让字体可以正常显示对基础shader做了一定调整
public static Shader CardBasicShader { get; private set; }
```

```csharp
//卡牌标题字体的TextMeshPro资源
public static TMP_FontAsset TitleFont { get; private set; }
```

```csharp
//卡牌标题的原始字体
public static Font TitleOrigFont { get; private set; }
```

```csharp
//卡牌介绍使用字体的TextMeshPro资源
public static TMP_FontAsset DiscriptionFont { get; private set; }
```

```csharp
//卡牌介绍使用的原始字体
public static Font DiscriptionOrigFont { get; private set; }
```

```csharp
//TextMeshPro的配置文件，使TextMeshPro正常运行必须的资源
public static TMP_Settings TMP_settings { get; private set; }
```

```csharp
//正面增益的卡背
public static Texture MoonBack { get; private set; }
```

```csharp
//负面增益的卡背
public static Texture FPBack { get; private set; }
```

```csharp
//中性增益的卡背
public static Texture SlugBack { get; private set; }
```

```csharp
//文字卡牌的卡背，目前没有使用
public static Texture TextBack { get; private set; }
```

<h2>方法</h2>

```csharp
//从文件中加载资源
public static void LoadAssets()
```

```csharp
//调整TextMeshPro使用的钩子方法，仅内部调用
public static TMP_Settings TMP_Settings_instance_get(Func<TMP_Settings> orig)
```

```csharp
//创建TextMeshPro的字体资源，仅内部调用
public static TMP_FontAsset CreateFontAsset(Font font)
```

```csharp
//创建TextMeshPro的字体资源，仅内部调用
public static TMP_FontAsset CreateFontAsset(Font font, int samplingPointSize, int atlasPadding, GlyphRenderMode renderMode, int atlasWidth, int atlasHeight, AtlasPopulationMode atlasPopulationMode = AtlasPopulationMode.Dynamic, bool enableMultiAtlasSupport = true)
```

```csharp
//在字体资源中加载字符集的协程方法，仅内部调用
public static IEnumerator LoadCharacterForFont(TMP_FontAsset fontToLoad, int index)
```