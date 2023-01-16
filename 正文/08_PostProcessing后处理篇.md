# Toon Shading Collection 

## CH08 - Post Processing 后处理

这里仅讨论对角色影响较大的后处理效果，与场景强相关的后处理放到以后的卡通渲染场景篇专文探讨。

<br>

<br>

------

### 卡通渲染后处理的意义

#### 仿动画画面论

后处理也就是传统动画中的后期特效。后处理特效不建议增加太多，用多了会破坏画面的色彩构成，非常花哨。

传统动画有一些后期调色的工作是在After Effects中合成的，同样在3D引擎中也可以通过shading的形式使用这种调色方法。

3D渲染图象后处理的方法实际上就是和视频后期软件差不多的功效，但是要求是要实时渲染，只要性能优化的足够好就可以了。

![CH08_PostProcessing_0_AnimeAfterEffects](../imgs/CH08_PostProcessing_0_AnimeAfterEffects.png)

*↑《崩坏3》宣传片中的后期调色与合成*

<br>

#### “低频信息”论

虽然Flare，Bloom，Volume Light从物理角度是不同的光学现象，但它们都是画面的低频信息提供方，可以近似处理。

风格化渲染实际上非常需要这类低频信息增强画面的柔和感，以及缓解纯色块的单调感。除了Bloom，Flare和体积光同样也是实现这一点的很好用的工具。

同理，其它后处理也与低频信息有或多或少的联系。

<br>

<br>

------

### Bloom

![CH08_PostProcessing_A_BloomOnOff](../imgs/CH08_PostProcessing_A_BloomOnOff.jpg)

*↑罪恶装备：左图开了Bloom，右图没开*

Bloom是卡通渲染中比较常见的后处理，基本算是必备，但放手机上带宽消耗比较大。

<br>

#### 卡通Bloom区域

卡通Bloom和写实渲染里普通的HDR有区别。

Bloom从物理角度应该是一种高亮度产生的视觉现象。普通的HDR是根据原像素rgb通道的最高值作为阈值来决定混合的模糊图的强度的，也就是越亮的部分发光程度越高。

但动画里，Bloom的系数和亮度关系并不是很大，使用PBR的方式来处理Bloom，很容易导致泛光部分发白。我们需要的其实是一个带颜色的光晕。

因为NPR体系下的颜色经过了各种特殊指定，并不和实际现实亮度直接对应，很容易出现纯白区域不希望Bloom，而高饱和低亮度区域反而希望Bloom的情况。

其次，Bloom同时也是一种画面上的低频信息，为了营造一种柔和的画面“氛围”，我们也希望增加这种低频信息的比例。而强行拉高整体Bloom强度又会导致高亮度区域过曝。

因此，常见的Bloom后处理根据颜色或亮度来决定曝光程度，而卡通渲染的Bloom需要能够自己控制曝光区域，与这个区域的颜色和亮度无关。

所以我们需要一个自定义的Bloom缩放系数。因为从崩三开始的游戏都实装了这个特性，是否要实现它一般并不会有疑问。

根据画风，一般也会参考颜色的亮度值，这样Bloom系数就只需要一个修正用的值而非贴图。

<br>

分别处理物体的面光和背光Bloom，可以让亮部单独产生Bloom的效果。

从“物理正确”角度，其实也是需要这样做的。因为Bloom的强弱其实代表了受光的强弱，但是卡通渲染的光照强度并不会完全通过颜色表达出来。为了保持画面的饱和度，亮部的实际亮度并不会太高，而暗部的亮度常常也不能过暗。

因此，将Bloom和颜色剥离开来，反而更加“物理正确”。

卡通渲染还特别喜欢让头发和皮肤具有更高的Bloom强度，使得它们看上去更加具有光泽。

另外通过让边缘光部分的颜色溢出到周围区域，能提高画面的通透感，并且让角色更好的跟周围场景融合。

而为了保持发光物体的饱和度，实际颜色值不能过高，从而也需要用高Bloom值来补齐原本想要的泛光。

<br>

Bloom效果不宜使用过猛，比较多的辉光效果一般在动画中会被用于表现温馨的情感氛围。

<br>

#### 卡通Bloom操作示例

![CH08_PostProcessing_A_BloomAreaAnalysis](../imgs/CH08_PostProcessing_A_BloomAreaAnalysis.jpg)

以上图举例，角色的衣服是白色的，颜色上属于亮度最高的。但是这里我们希望衣服上不要有过多曝光，曝光能集中在角色边缘光的位置。

所以我们需要向屏幕缓冲写入一个专门Bloom系数来控制Bloom强度，在这里我们将bloom强度值写入Alpha通道，利用Alpha值来控制曝光度，就可以让任何颜色的物体泛光了。

假设bloom的曝光主要集中在光照方向，边缘光的部分。于是将边缘光乘以漫反射公式获得比较符合光照方向的边缘光范围，然后将它的值赋给Alpha通道。

![CH08_PostProcessing_A_BloomEffectAnatomy](../imgs/CH08_PostProcessing_A_BloomEffectAnatomy.jpg)

*↑左：通过Alpha通道控制的曝光范围；右：曝光的溢色效果*

![CH08_PostProcessing_A_BloomComparism](../imgs/CH08_PostProcessing_A_BloomComparism.jpg)

*↑左：未开启Bloom；右：开启Bloom*

<br>

Bloom使用高光提取，还可以偏移色相饱和度，这样对卡通渲染来说，色彩看起来就会更加丰富，而且从一定程度上，避免了应用Bloom后，画面会降低饱和度的现象。

![CH08_PostProcessing_A_BloomHueSaturation](../imgs/CH08_PostProcessing_A_BloomHueSaturation.jpeg)

<br>

#### 性能优化方向探讨

用Alpha通道控制Bloom的曝光，在渲染半透明物体的时候会遇到问题，因为半透明混合也需要用到Alpha，这点可以通过使用额外的Pass单独写入Alpha、原pass专门些RGB通道来解决。

还有额外带宽的问题。写Alpha通道的情况，Alpha Blend需要额外PASS，但Additive不需要。如果你的游戏特效使用Additive较多，额外PASS就少，就越适合写Alpha的方案。而且即使是Alpha Blend，如果不在乎Bloom系数的准确性，AlphaBlend物体大多也能堆叠到接近不透明，Bloom系数也允许常态使用最高值1——那特效不用第二个PASS修正也是可以的。

不修正，缺陷就是特效容易因为Bloom而过爆。但战双的过爆现象很少见，是不是修正的比较普遍就不知道了。

还可以让Bloom系数影响程度降低，普通特效需要的Bloom就是Bloom系数的最高值。如果需要较大的Bloom强度，则通过在HDR范围调整颜色本身的亮度来控制（这样Bloom就会更加接近PBR）。虽然束手束脚，应该也可以勉强达到我们的要求，但这个实现也不太稳定。

如果特效是单独渲染到RT进行降采样优化的，可能对这个方法影响更小些。

还有办法是额外开RT传入bloom值，如果是为此单开新RT，只用到一个通道会浪费，但如果本来的RT就有多余通道，比如法线RT，那就正好白给。

<br>

#### 整体方案安排探讨

具体实现的时候，最佳方案其实是准备一个单独的Half通道以储存一个HDR范围的亮度值，并在frag中将原始颜色的亮度值计算出并存入。只有这样，才能保证Alpha Blend, Additive，以及各种特殊混合模式下的Bloom值的正确性。

通常，我们并不愿意花费一个16bit的通道来实现这个效果，只愿意使用一个8bit的通道，所以只会储存一个Bloom的缩放值。这样在透明混合的时候就会出现各种麻烦。

 <br>

目前比较倾向的方案是：

放弃透明物体的Bloom值自定义功能，透明物体阶段并不写入Bloom值，这样也节约了带宽。

而为了最终计算Bloom结果正确，必须将透明物体和不透明物体分别渲染，然后在Bloom阶段重新组合。（为什么必须这样解释起来很麻烦，信我就行了）

这个分离透明物体渲染的功能UE本来就提供了，只需要修改Bloom部分的RT组合，实现还是很简单的。而且通过配置r.SeparateTranslucencyScreenPercentage还可以获得半透物体降分辨率功能，还可以根据帧率自动进行，基本上不会不开，分离渲染导致的性能代价也就无所谓了。

事实上，这就是原神的方案，所以估计也没啥人会反对。

（原神的透明物体分辨率过低的问题确实严重。我觉得可以将部分需要精度的透明材质转为Mask材质来解决问题，Mask通过TAA抖动模拟单层半透的效果还是不错的）

 <br>

严格来讲，最理想的方案其实是执行2次Bloom，一次无视物理亮度给画面整体做一次模糊叠加，第二次根据亮度做正常Bloom模拟实际的辉光物理现象。但因为Bloom的性能成本较高，当然希望一次做完。

配置Bloom参数的时候要考虑倒这一点。两个Bloom的功能其实是不同的。

<br>

#### 思路转变方向探讨

是否可以把实际亮度和颜色在光照体系下就直接拆分开，把亮度当作RGB之外的第4个颜色分量，然后用“亮度”值直接算Bloom，这样处理Additive物体时也更加方便（否则要直接屏蔽了Additive物体的Bloom值写入）。

但这样改的东西满多的，所有涉及光照的部分都要改，而且这需要让Bloom值是个HDR的值……

真要认真做，可能那才是最完美的方案。

<br>

#### 视觉升级方向探讨

动画和游戏的Bloom还有一个重要的区别是：模糊图和原图的混合模式并不只有线性减淡(Additive)。

如果像动画后期一样使用不同的混合模式，则有希望做出更接近动画的结果（比如有是通过叠加混合实现的加深效果）。

这个还需要尝试和探索，目前的方案终究只是PBR的一个修正，还有很大修正空间。

<br>

<br>

------

### 抗锯齿

使用后处理效果可以避免的一个非常重要的问题就是3D模型的锯齿感。使用抗锯齿（各种 Anti-Aliasing技术）后处理可以大幅度减小锯齿感，更像手绘动画效果。

上面bloom效果在3D游戏中一般配合抗锯齿一起进行。

<br>

<br>

------

### AO

![CH08_PostProcessing_B_AnimeStyleAO](../imgs/CH08_PostProcessing_B_AnimeStyleAO.jpg)

*↑通过对比图可以看出，在应用了AO之后，右图比左图层次感更强*

对于角色上的动态AO实现，米哈游使用修改过的HBAO，用于指定AO区域中颜色的饱和度和色调调整，使加入AO后的图像颜色看起来不会变脏。

其实我觉得不加AO也挺好看啊，加了AO反而感觉纵深感太强了，画面还是有点脏。

<br>

<br>

------

### 动画摄影后期

在日本动画制作过程当中会加入很多后期效果，这些后期效果可以提升角色跟场景的光感，也包括一些模糊的景深效果。这些效果在日本动画流程里面被称为动画摄影，像这里面演示的Flare跟Para的效果，就能够提升角色光感或者是说空气感。

<br>

#### Flare发光渐变

![CH08_PostProcessing_C_AnimeAfterEffectFlare1](../imgs/CH08_PostProcessing_C_AnimeAfterEffectFlare1.png)

首先讲一下Flare。Flare的话实际上是模拟了一个正上方的光照，让场景和角色有一种受光的感觉，你可以理解成叠加了一层发光的遮罩，它是对原始颜色进行了一次叠加。

![CH08_PostProcessing_C_AnimeAfterEffectFlare2](../imgs/CH08_PostProcessing_C_AnimeAfterEffectFlare2.png)

《赛马娘》里有加入这种Flare的效果。可以看到加入这个效果之后，角色的光感提升了非常多。

![CH08_PostProcessing_C_AnimeAfterEffectFlare3](../imgs/CH08_PostProcessing_C_AnimeAfterEffectFlare3.png)

<br>

#### Para叠黑渐变

![CH08_PostProcessing_C_AnimeAfterEffectPara1](../imgs/CH08_PostProcessing_C_AnimeAfterEffectPara1.png)

Pare跟Flare是反过来的，Flare是加了一个光感，Para是加了一个阴影感。它模拟阴影，就像叠加了一层深色的遮罩，对原始颜色进行相乘。

![CH08_PostProcessing_C_AnimeAfterEffect1](../imgs/CH08_PostProcessing_C_AnimeAfterEffect1.jpg)

这是动画常用的摄影后期特效，能把卡通角色和环境很好地融合起来。

指的是上图左上角那块偏蓝的部分，有点类似屏幕暗角。在整个游戏里都始终存在存在，是蓝色协议的画面特色，也对其“动画感”的产生起到了很大的效用。

![CH08_PostProcessing_C_AnimeAfterEffect2](../imgs/CH08_PostProcessing_C_AnimeAfterEffect2.jpg)

![CH08_PostProcessing_C_AnimeAfterEffect3](../imgs/CH08_PostProcessing_C_AnimeAfterEffect3.jpg)

*↑效果放大10倍的样子*

这个效果基本是一个蓝色的变色遮罩，是一个软边的弧形，存在于屏幕上方，和太阳方向有关。

其实说白了，就是一个画遮罩进行区域调色的后处理特效。如果走正常动画流程，这一步都是肯定有的。所以如果你想做出和动画差不多的画面，当然也应该走这个流程。而且，这个东西不能用光照代替，因为光照是HDR下的，而后处理特效是LDR的。而调色只有在LDR才是对的。

它必须是一个后处理效果。但后处理效果一般不会再去画指定Mesh，只会给你一个参数调整工具，怎么设计这个效果就是个问题。

蓝色协议的剧情动画部分之所以亮眼，也是因为加上了Flare跟Para效果，画面的光感或者说是空气感就明显更加的强烈了：

![CH08_PostProcessing_C_AnimeAfterEffect4](../imgs/CH08_PostProcessing_C_AnimeAfterEffect4.jpg)

无非就是各种遮罩圆的位置，硬软边。不太在乎性能是可能的。虽然他们什么都没说，只要知道是这是用后处理做的，就可以试试。只考虑固定过场是可以搞的。

后处理也是可以调用管线内容画Mesh的，定制能力足够就行。要效果就不要纠结这点性能。

<br>

而原神的画面，就像是一个没有经过“摄影后期阶段”的动画。

相比动画，它整整少了一个制作阶段，所以看上去才和动画不一样。

所以假如把这个阶段加上去，必然可以大幅向着“像动画”这个方向前进。

向影视界学习显然是一个性价比很高的改进方向。

![CH08_PostProcessing_C_AnimeAfterEffect5](../imgs/CH08_PostProcessing_C_AnimeAfterEffect5.jpg)

![CH08_PostProcessing_C_AnimeAfterEffect6](../imgs/CH08_PostProcessing_C_AnimeAfterEffect6.jpg)

*↑原始背景叠人物 vs 最终背景叠人物*

动画后期技术细节见：[动画摄影后期流程解密——Graphinica摄影杨晓牧采访](https://zhuanlan.zhihu.com/p/20202161)

<br>

<br>

------

### Tone Mapping

我们做HDR的话，经常需要做ToneMapping。

后处理中的ToneMapping，这是HDR数据映射到LDR的过程，因为LDR显示设备只能显示0到1区间的色彩值，所以为了将计算中大于1的色彩值完整显示出来，这就需要一条在正无穷上收敛在1的曲线来映射。

色调映射是一切后续内容的基础。因为风格化场景并不依赖物理规则打光，而需要靠主观调色，如果色调映射一开始就选择错了，之后的所有工作都会打水漂。

现在主流的是ACES ToneMapping。图中的绿色的曲线，就是ACES的映射曲线。

![CH08_PostProcessing_D_ToneMapping1](../imgs/CH08_PostProcessing_D_ToneMapping1.png)

它有一个问题，就是它会让这个物体发光越亮的地方，颜色就越偏于白色，其次它会导致画面的饱和度下降。

![CH08_PostProcessing_D_ToneMapping2](../imgs/CH08_PostProcessing_D_ToneMapping2.png)

那么可以看到一个对比，就是使用了ACES ToneMapping之后，整个画面的饱和度似乎都出现了下降，这就与我们追求高饱和度的卡通渲染不是非常相符。

![CH08_PostProcessing_D_ToneMapping3](../imgs/CH08_PostProcessing_D_ToneMapping3.png)

<br>

尝试过的都会发现，直接使用标准ACES是有问题的。而且这点在做场景的时候可能不重要，因为不管色调映射怎么处理颜色，你手动调光也总能调出预想的画面来。问题出在人物部分的兼容上。

再举个例子：

（下面虽然是图片，但也能代表实际模型在ACES环境下的颜色变化）

![CH08_PostProcessing_D_ToneMapping11](../imgs/CH08_PostProcessing_D_ToneMapping11.jpg)

在使用标准ACES的时候，如果亮度调的较低，暗部细节就会严重丢失（图2），而如果调高亮度至能看清暗部细节，亮部细节就会丢失（图3）。

上面这张图并不明显，但对于一些在脸上画了细节过渡和腮红的角色来讲，亮部丢失细节的问题同样严重。

虽然也确实可以通过手动调整贴图来解决，因为做不到可见即所得，工作量会大幅提升。虽说PBR时代非要手画贴图差不多也是这鸟样……

<br>

#### 调整方案探讨

<br>

- ##### 不用ACES

我们当然还有关闭ACES换用其他ToneMaping方案这样一个选择。但ACES在表现亮度高于1的高亮度区域是有很有优势的，可以显著增强特效的光感，而这正是ACES“高亮度偏白”的功效。

![CH08_PostProcessing_D_ToneMapping12](../imgs/CH08_PostProcessing_D_ToneMapping12.jpg)

如果有这方面的需求，ACES就必须使用。但使用了，人物的亮部就会错误地偏白。

<br>

- ##### 曲线调整

ACES映射曲线极限是1.03，就相当于把0到正无穷的数据映射到了0到1.03上，但是这就将原有的0到1的色彩值映射到了0到0.8的区间上，这对卡通渲染的表现来说是比较致命的，特别是在肤色上。

右图中就是将肤色经过ACES曲线处理的结果，这种色彩变化是卡通渲染不能接受的，所以一般会去修改ACES曲线的多项式值，去降低ToneMapping映射的影响。

![CH08_PostProcessing_D_ToneMapping4](../imgs/CH08_PostProcessing_D_ToneMapping4.png)

蓝色的曲线就是一种修正后的结果，然后可以看到压缩1到正无穷高亮度值的显示区间，可以稍微缓解一部分色彩表现。所以在项目中后处理的色彩表现就要在制作的早期就要预先开始进行评估。

![CH08_PostProcessing_D_ToneMapping5](../imgs/CH08_PostProcessing_D_ToneMapping5.png)

然后这里就是ACES映射曲线修改前后的区别，魔改的映射曲线可以纠正一部分皮肤发灰的问题，上面这张图和下面这张图的是对比。

![CH08_PostProcessing_D_ToneMapping6](../imgs/CH08_PostProcessing_D_ToneMapping6.png)

<br>

在UE中的话，我们可以换用用于模拟旧版ToneMaping的一个兼容性设置，在Post中设置：

![CH08_PostProcessing_D_ToneMapping13](../imgs/CH08_PostProcessing_D_ToneMapping13.jpg)

这会改变ToneMaping的曲线，让前端的S区域变得平缓，从而缓解暗部和亮部细节丢失的问题。

![CH08_PostProcessing_D_ToneMapping14](../imgs/CH08_PostProcessing_D_ToneMapping14.jpg)

而由于RRT/ODT过程导致的偏色，高亮度的脸部依然会偏白，这是这个兼容做法还没解决的问题，但终究比之前强多了。

<br>

- ##### 资源制作流程反推

要在ACES下改善任务亮部偏白问题，重新定制ACES的RRT/ODT的流程就是一个需要做的事情。我们希望亮度低于1的区域不要偏色，但高于1的区域则允许偏色，这并不是一个矛盾的需求。

其中一个方案是：可以考虑在无偏色的原始颜色和偏色后的颜色根据亮度做一个插值，保证小于1的区域是无偏色的，然后渐渐过渡到需要偏色的区域。

然而试了下，好像反转校色方案依然可用（图3）（相关讨论摘录见PBR结合章节“PBR魔改思路参考”）

![CH08_PostProcessing_D_ToneMapping15](../imgs/CH08_PostProcessing_D_ToneMapping15.jpg)

做法是在图2的基础上对人物纹理的颜色输出做以下处理（处理强度可以通过和原颜色lerp来控制）：
$$
color = 3.4475 * color ^ 3 - 2.7866 * color ^ 2 + 1.2281 * color - 0.0056
$$
其实这个公式相当于预处理了一次纹理，将纹理的高光和暗部区域拉长了，这样就不怕明暗细节被ACES破坏了。

本来想换个正常点的解决方案，结果……至少这也是个可行的解决方案。

<br>

总之，场景由于可以通过调整光照来避免亮部和暗部细节丢失，对色调映射的依赖并不严重。但人物由于使用的是二值化光照，细节被迫需要用同一亮度下的纯纹理来表现，就不得不设法减弱色调映射导致的细节丢失。

这其实是一个人物独有的问题，可目前人物和场景很难剥离处理，所以场景也不得不在最开始就开始考虑。

上面的色调映射调整方案，其实牺牲了场景的效果。曲线调平虽然能保暗部细节，但会导致对比无法拉开。而如果使用原来的S曲线，是可以通过动态曝光来平衡“高对比”和“保暗部细节”两者的矛盾的。

当然你要不开自动曝光，也没有动态光照……个人也觉得拉平可能也没啥所谓吧，也就是光照比较平罢了，现在的“手游”不都这样。

但理想的方案还是继续优化第二种改人物贴图颜色的方案，因为它并不限制场景，是更泛用的做法。

<br>

- ##### 修改算法源码

关于如何修改SRP里面的ToneMapping算法，再具体给大家讲一下，怎么修改Unity引用的package包。

![CH08_PostProcessing_D_ToneMapping7](../imgs/CH08_PostProcessing_D_ToneMapping7.png)

Unity的package包都放在Library\PackageCache路径下面，这个路径下面的包是受到Unity管理的，如果我们对它进行修改的话，Unity就会把我们的修改回滚掉。所以，我们需要把这个包拷贝出来，比如说拷贝到Assets目录的同级目录。

Package路径下面有一个叫做manifest.json文件，在这里面我们可以直接指定我们使用包的文件路径。在这里设置为我们拷贝的路径之后，我们就可以对它进行修改了，这个时候Unity就不会回滚修改。

![CH08_PostProcessing_D_ToneMapping8](../imgs/CH08_PostProcessing_D_ToneMapping8.png)

我们对SRP Core里的ACES.hlsl文件进行修改，把其中的两个FACTOR变量 +0.2，然后我们看一眼效果。修改以后的效果就是右边这张图，可以看到应用了ToneMapping之后，颜色饱和度也没有下降。

![CH08_PostProcessing_D_ToneMapping9](../imgs/CH08_PostProcessing_D_ToneMapping9.png)

我们再看一眼应用于角色上的效果，这几个人物的颜色饱和度也保持了我们使用ToneMapping之前的饱和度。

![CH08_PostProcessing_D_ToneMapping10](../imgs/CH08_PostProcessing_D_ToneMapping10.png)

当然这个修改就很tricky，不一定对。

<br>

- ##### 修改算法源码

Gran Turismo曲线效果似乎较好：

![CH08_PostProcessing_D_ToneMapping16](../imgs/CH08_PostProcessing_D_ToneMapping16.png)

[效果演示1](https://www.bilibili.com/video/BV1J341137U5)  [效果演示2](https://www.youtube.com/watch?v=WR_SP4LmOlw&t=3s)  

[曲线演示](https://www.desmos.com/calculator/gslcdxvipg?lang=zh-CN)  

<br>

#### 总结

这里主要也是想给大家抛砖引玉，说一下卡通渲染对于ToneMapping是有特殊要求的。为了满足这些特殊要求，我们就可以尝试下面两个方法。一个是我们自己调整ToneMapping的曲线，另外我们也可以尝试一些其他的ToneMapping的算法，如Filmic ToneMapping。

然后其他的修改方式也有，比如反函数的ToneMapping计算，或者是对皮肤做亮度补偿，或者直接不管使用伽玛流程，它画出来什么就是什么。

<br>

<br>

------

### 自动曝光

一种建议说，做风格化渲染的话，最好不要开动态曝光，不然很难控制光影。

<br>

<br>

------





<br>

<br>

------

