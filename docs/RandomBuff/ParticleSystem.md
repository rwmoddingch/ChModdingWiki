## 基本使用方法

新建一个ParticleEmitter实例（粒子发射器）并使用ApplyEmitterModule、ApplyParticleSpawn和ApplyParticleModule方法设置发射器与粒子的具体行为。

```c#
//示例，并不完整，只是为了展示方法和模块该怎么写
emitter.ApplyEmitterModule(new SetEmitterLife(emitter, setLife - 20, false));
emitter.ApplyParticleSpawn(new BurstSpawnerModule(emitter, 260));
emitter.ApplyParticleModule(new AddElement(emitter, new Particle.SpriteInitParam("Futile_White", "FlatLight", alpha: 0.5f)));
```
对于一个设置完成的ParticleEmitter，使用ParticleSystem.ApplyEmitterAndInit(ParticleEmitter emitter)方法“应用”这个发射器使其能实际发挥作用。
## (一般可能用到的)字段

#### **ParticleEmitter.pos****（**Vector2）

发射器的中心位置，如果设置movetype为Relative则粒子的运动以该位置为原点 

#### **ParticleEmitter.vel（**Vector2）

发射器的速度，似乎如果把发射器绑定到有物理行为的物体上时就会对它赋值

#### **ParticleEmitter.room****（**Room）

发射器所在的房间

## 方法与Module

### ApplyEmitterModule

控制发射器的行为（主要为控制发射器生命周期），默认包含以下两个module：

#### **SetEmitterLife**

**(ParticleEmitter emitter, int life, bool loop, bool killOnFinish = true)**

设置发射器的寿命，通过计时器来自动重置或清除发射器

emitter：目标发射器实例

life：目标发射器的寿命，每次更新（一个逻辑帧即0.025s）时减少

loop：是否循环。如果为True则当life=0时重置life的值

killOnFinish：是否在结束时清除，默认为True。如果为True则当life=0时执行Die方法

#### BindEmitterToPhysicalObject

**(ParticleEmitter emitter, PhysicalObject physicalObject, int chunk = 0, bool dieOnNoRoomOrNull = true)**

将发射器绑定到有物理性质的物体上，发射器在初始化和更新时将会尝试同步该物体的位置和速度，而且如果找不到该物体或该物体房间为空则清除发射器

emitter：目标发射器实例

physicalObject：目标物体实例

chunk：如果物体有多个节段则决定绑定到哪一段，默认为0

dieOnNoRoomOrNull：如果无房间或为空则清除（似乎并没有用到？目前只要房间和物体中任何一个找不到就Die）

当然也可以不使用这两个Module，手动给Emitter的速度和位置赋值也是可以的，可以用Die方法强制清除发射器

### ApplyParticleSpawn

控制粒子生成的模式，默认包含以下两个module：

#### RateSpawnerModule

**(ParticleEmitter emitter, int maxParticleCount, int ratePerSec)**

按一定的频率连续生成粒子，直到达到最大值

emitter：目标发射器实例

maxParticleCount：该发射器在一个生命周期中能累计产生的最大粒子数

rateParSec：粒子生成速率

#### BurstSpawnerModule

**(ParticleEmitter emitter, int count)**

在初始化时一次性生成指定数量的粒子

emitter：目标发射器实例

count：生成的粒子数

同上，你大概可以不用这些Module 而是自己调用ParticleEmitter的SpawnParticle()方法让它以你喜欢的方式生成粒子

### ApplyParticleModule

控制粒子的具体行为，默认包含以下模块：

**/初始化/**

这些Module负责控制粒子的初始化行为（指定初始状态的速度/大小/外观等），可以自己选取并组合。有些对同一属性赋值的Module不可叠加并且会按顺序覆盖之前的设定。也可以自己编写新的Module以实现更丰富的效果。

#### AddElement

**(ParticleEmitter emitter, params Particle.SpriteInitParam[] spriteInitParam)**

为粒子添加外观元素（可以多次添加）

emitter：目标发射器实例

spriteInitParam：粒子的sprite渲染初始化 实际格式应为：

**(string element, string shader, int layer = 8, float alpha = 1f, float scale = 1f, Color? constCol = null)**

element：贴图名称

shader：使用的着色器（mod开发群里提供了RW内置贴图和着色器预览）

layer：渲染层，默认为8

alpha：不透明度，默认为1

scale：缩放比例，默认为1

#### SetMoveType

**(ParticleEmitter emitter, Particle.MoveType moveType)**

设置粒子的运动参考系

emitter：目标发射器实例

moveType：如果为Realtive，则计算粒子坐标以发射器为参考系，如果为Global则使用世界坐标系

#### SetConstVelociy

**(ParticleEmitter emitter, Vector2 vel)**

指定粒子的初始速度为固定值

emitter：目标发射器实例

vel：速度

#### SetRandomVelocity

**(ParticleEmitter emitter, Vector2 a, Vector2 b,bool isDir = true)**

使用两个向量随机指定粒子的初始方向和速度

emitter：目标发射器实例

a：指定x速度和y速度的一个值

b：指定x速度和y速度的另一个值

isDir：决定初始速度的计算方式

如果为true，则对a和b的角度进行随机插值后转化为单位向量（作为方向） ，再乘以一个a和b的绝对值之间随机选取的值（作为速度），实际生成a与b夹角范围内的大小介于a和b之间的向量。

如果为false，直接取a.x和b.x之间的随机数为x，a.y和b.y之间的随机数为y，然后合成向量

#### SetSphericalVelocity

**(ParticleEmitter emitter, float vA, float vB)**

粒子飞向任何方向的可能性相等，速度为介于vA和vB之间的随机值

emitter：目标发射器实例

vA：速度的最大值或最小值

vB：速度的最小值或最大值

#### SetVelociyFromEmitter

**(ParticleEmitter emitter, float t)**

基于发射器的速度设置粒子的初始速度

emitter：目标发射器实例

t：速度倍率

#### 
#### SetRandomRotation

**(ParticleEmitter emitter, float rotationA, float rotationB)**

随机化粒子的初始旋转

emitter：目标发射器实例

a，b：粒子旋转的取值范围（单位似乎是角度）

#### 
#### SetRandomScale

**(ParticleEmitter emitter, float a, float b)**

随机化粒子的初始大小

emitter：目标发射器实例

a，b：粒子比例的取值范围（实际是产生(a,a)和(b,b)两个二维向量来控制大小以保证粒子图像等比例缩放）

#### 
#### SetRandomPos

**(ParticleEmitter emitter, float rad)**

粒子的生成位置分散在以发射器为中心 rad范围内的圆形区域中，且越靠近中心密度越高

emitter：目标发射器实例

rad：半径

#### SetRandomLife

**(ParticleEmitter emitter, int a, int b)**

随机化粒子寿命

emitter：目标发射器实例

a，b：粒子的存在时长（单位为逻辑帧）,会取a与b之间的随机值

#### SetConstColor

**ParticleEmitter emitter, Color color)**

设置固定的粒子起始颜色

emitter：目标发射器实例

color：颜色

#### SetRandomColor

**(ParticleEmitter emitter, float hueA, float hueB, float saturation, float lightness)**

设置带有随机色相的初始粒子颜色

emitter：目标发射器实例

hueA：色相A

hueB：色相B

saturation：饱和度

lightness：明度

**/更新/**

这些Module负责控制粒子更新时的行为（生命周期中颜色、外观和运动状态的变化），也可以视情况自由组合使用或自行编写新的Module

#### ConstantAcc

**(ParticleEmitter emitter, Vector2 acc)**

粒子的速度会均匀增加

emitter：目标发射器实例（...这个 真的有必要每次都说一遍吗）

acc：加速度

#### Gravity

**(ParticleEmitter emitter, float g)**

粒子受重力影响

emitter：目标发射器实例

g：重力大小

#### （各种各样的）OverLife

**(emitter,(particle,particle.lifeparam) =>{return})**

指定粒子的某一属性在整个生命周期内变化，用法都差不多。

emitter：目标发射器实例

particle：目标粒子实例

particle.lifeparam：目标粒子的生命周期（已经存在的时间与最大寿命的比值，范围0f-1f）

```c#
//示例
//建立新的发射器实例
emitter = new ParticleEmitter(room);

/*
省略了发射器行为、粒子生成模式和粒子初始化的设置
*/

//对于发射器实例emitter所产生的每一个粒子实例p以及该粒子的生命周期l，这两个被提供用于在大括号内计算并返回（return）一个float类型的值作为粒子的不透明度
emitter.ApplyParticleModule(new AlphaOverLife(emitter,
    (p, l) =>
    {
        if (l < 0.2f)
        {
            return l * 5f;
        }
        else if (l > 0.8f)
        {
            return 1f - (l - 0.8f) * 5f;
        }
        return 1f;
    }));
```
ScaleOverLife：返回类型可以为float（缩放比例）或Vector2（长宽缩放比例）
ColorOverLife：返回类型为Color（颜色）

VelocityOverLife：返回类型为Vector2（速度）

PositionOverLife：返回类型为Vector2（位置）

RotationOverLife：返回类型为float（旋转角度）

AlphaOverLife：返回类型为float，范围在0f-1f之间（透明度）

#### SimpleParticlePhysic

**(ParticleEmitter emitter, bool bounce, bool die, float velDamping = 0.8f)**

粒子物理模拟

emitter：目标发射器实例

bounce：是否在触碰实体物块时反弹

die：是否在触及实体物块时消失

（如果die为true则会覆盖bounce的效果，如果bounce和die都为false则粒子会在触及实体物块后停止运动）

velDamping：运动阻尼大小

**/渲染/**

这些Module负责控制粒子的渲染。

-如果你只需要正常显示粒子的外观，则不需要手动指定渲染器。**未指定渲染器时，会自动使用默认渲染器的默认设置。**

-如果你不需要显示粒子但需要使用粒子的拖尾效果，则需要指定尾迹渲染器。这时候将不会自动使用默认渲染器 因此实际生效的只有尾迹渲染器。

-如果你需要同时显示粒子本身和拖尾效果，则需要手动指定默认渲染器和尾迹渲染器，以保证两个渲染器都正常工作。

#### DefaultDrawer

**(ParticleEmitter emitter, int[] renderElementIndexs)**

默认的渲染器，用于显示粒子本身。如果不指定渲染器则会自动使用默认渲染器的默认设置

```c#
//默认设置
//注意这边有个自动换行 实际上最好写在一行内
emitter.ApplyParticleModule(new DefaultDrawer(emitter, new int[1] { 0 }));

//如果需要手动指定默认渲染器，建议直接套用默认设置，否则可能会引起不可预测的后果
```
#### TrailDrawer

**(ParticleEmitter emitter, int index, int trailCount = 10)**

尾迹渲染器，仅用于显示尾迹效果，仅在手动指定渲染器时启用。

emitter：emitter：目标发射器实例

index：尾迹节段索引，暂时不建议设为1意外的任何值

trailCount：尾迹总段数，会直接影响尾迹的长度，默认为10

尾迹渲染器的具体设置需要在语句后的大括号内进行，类似于上面那些“OverLife”。

提供了六个可被赋值的变量用于控制尾迹的外观：

**gradient和colorOverLife**

控制尾迹的颜色，两者的返回值都为Color类型，**且两者中必须有且只有一个被指定**。gradient提供尾迹某一节的索引i与尾迹的总节数a来单独为每一节计算颜色，而colorOverLife通过粒子的生命周期l来计算颜色，颜色将会应用于整条尾迹

**width和widthModifyOverLife**

控制尾迹的宽度，两者的返回值都为float类型，**且必须指定width**，两者效果可叠加。width提供尾迹某一节的索引i与尾迹的总节数a来单独为每一节计算宽度，而widthModifyOverLife通过粒子粒子的生命周期l来计算整条尾迹的宽度倍率，结果以乘积方式叠加

**alpha和alphaModifyOverLife**

用于控制尾迹的不透明度，两者的返回值都为float类型，**且必须指定alpha**，两者效果可叠加。alpha提供尾迹某一节的索引i与尾迹的总节数a来单独为每一节计算不透明度，而alphaModifyOverLife通过粒子的生命周期l来计算整条尾迹的不透明度倍率，结果以乘积方式叠加

gradient/width/alpha提供粒子p、尾迹某一节的索引i、尾迹的总节数a。

colorOverLife/widthModifyOverLife/alphaModifyOverLife提供粒子p、粒子的生命周期l。

```c#
//示例,来自ClusterBombBuff.cs
//小 心 自 动 换 行
emitter.ApplyParticleModule(new TrailDrawer(emitter, 1, 20)
{
    alpha = (p, i, a) => 1f - i / (float)a,
    alphaModifyOverLife = (p, l) => 1f - l,
    gradient = (p, i, a) => Color.Lerp(Color.white, p.setColor, i / (float)a),
    width = (p, i, a) => (1f - i / (float)a) * p.setScaleXY.x * 0.1f
});
```




















