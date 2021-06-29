# Toon Shading Collection 

## CH10 - More Materials 更多材质效果

<br>

![CH10_MoreMaterials_0_GraphicDesignTo3DMaterial](../imgs/CH10_MoreMaterials_0_GraphicDesignTo3DMaterial.jpg)

现在的卡通渲染游戏想要刺激玩家眼球，除了之前几章探讨过的，可能还要融入越来越丰富的材质效果，所以这里列一下可以考虑的材质。大部分源于偏写实的项目，也不知道是不是应该、或者能不能改良成更卡通的感觉。

![CH10_MoreMaterials_0_VariousMateirals](../imgs/CH10_MoreMaterials_0_VariousMateirals.jpg)

**注意：** 不同的光照模型混合计算，会增加额外的性能消耗。

<br>

------

### 半透明

#### 传统方案

传统方案是用Alpha混合或Dither抖动，低性能平台尤其适合抖动，不过美术同事一般不太能接受。

<br>

#### 高品质折射

角色材质中还包括其它特殊的材质，例如：水晶和纱巾等半透明材质，直接使用Alpha混合不能表现出应有的质感，为了更真实细腻的高品质，就需要实现折射和模糊效果，这二个效果都依赖于Unity的Command buffer。

实现折射效果时，Command buffer在渲染折射前获取已经渲染好的Backbuffer作为背景，用于折射采样, Rgb通道设置不同折射系数，分别采样三次来模拟色散效果。

对于模糊效果，则是用Command buffer将Backbuffer降采样并做模糊，生成4张尺寸依次减半模糊度递增的RenderTexture，然后根据相机距离和FOV以及材质固有的模糊参数，确定模糊程度，选择对应的RenderTexture来完成模糊效果。

还可以对这二者的实现做一定的优化，不对直接对Backbuffer使用全屏模糊，把物体本身作为Proxy mesh，只处理需要画的部分。

![CH10_MoreMaterials_A_TransparentRefraction](../imgs/CH10_MoreMaterials_A_TransparentRefraction.jpg)

<br>

<br>

------

### 闪点材质

![CH10_MoreMaterials_B_Glitter](../imgs/CH10_MoreMaterials_B_Glitter.jpeg)

类似于闪光片的闪烁效果，作为第二层材质来处理。

首先用specular map表示区域，然后用normal map增加反射不规则性，反射的光源主要是主光源和环境IBL贴图，此外使用光图查找表做五彩的闪烁效果，主要方法也是根据入射角度决定色彩偏移来查找光谱图，使用不同光谱图也会有不同的色彩表现。

<br>

<br>

------

### 动态赛博风材质

![CH10_MoreMaterials_C_DynamicCyber](../imgs/CH10_MoreMaterials_C_DynamicCyber.jpeg)

角色衣服上的动态条带，随着音乐的节奏变化而变化。

把跳动的音符调在分为两组，分别对应音频文件中的中频和低频信号，然后对音频做频谱分析，每帧得到的中频和低频强度根据强度去伸缩对应层的音符UV，得到动态的效果。

<br>

<br>

------

### 编织纹理布料

为了达到超细节品质，要对面料做深入研究。最为重要的环节便是编织，不同的织法出现的纹理都是不一样的，纹理结合材质可以做出无数种变化，让玩家能够充分享受到细节带来的满足感。

素材资料可以从生活和网络上来进行搜集，比如有时候跟家人逛街的时候，在挑选衣服时，去拍一些衣服的纹理，得到素材以后我们就需要进行信息提取。信息整理完后，感觉编制方式是组成纹理的重要部分，编制方式的不同，直接影响着纹理的效果。

![CH10_MoreMaterials_D_ComplexDetails1](../imgs/CH10_MoreMaterials_D_ComplexDetails1.jpg)

![CH10_MoreMaterials_D_ComplexDetails2](../imgs/CH10_MoreMaterials_D_ComplexDetails2.jpg)

为了得到非常精准的Normal 法线凹凸效果，传统做法是使用最为直接的制作方式，通过3D建模来模拟真实的编织方式。下图是闪耀暖暖开发组早期制作的几组编织模型。

![CH10_MoreMaterials_D_ComplexDetails3](../imgs/CH10_MoreMaterials_D_ComplexDetails3.jpg)

3D建模纹理的制作流程有利有弊，虽然它的品质很高，但其制作成本也比较高。最高的部分其实是修改成本，制作了非常多的纹理，当数量达到一个库的级别后，如果在这个基础上再做修改，成本是不可预想的，而且模型纹理的整理和规范很难做得科学些。

用高模来制作纹理，效果确实很不错，但是后期修改是非常大问题，高模后期修改成本很大，特别是如果换人的话，会增加成倍的工作量，持续修改的成本也很大，比如我想再继续细化的话，比如要怎么一些线头的效果，这个的制作成本就比较高。

虽然担心的问题并没有发生，但是为了追求更完美的效果和更高效且科学的工作流程，决定尝试使用 Substance Designer 制作纹理贴图。为了让团队能够熟练使用 Substance Designer要投入很大的成本，但这一切还是很值得的，所以才有了闪暖第一部宣传片中超高品质的材质效果，但这并不是最终品质，目前还在不断的突破自己。

![CH10_MoreMaterials_D_ComplexDetails4](../imgs/CH10_MoreMaterials_D_ComplexDetails4.jpg)

在制作过程中需要通过布料模型验证效果，该模型需要有不同的纹理方向，纵深程度，以此确认纹理中各个距离的效果。

纹理的检查和验证也是非常重要的，纹理的测试环境选择也很重要，这是我当时做纹理验证的时候做的素材，就放了一块布料在地上，其实我观察的时候只取了这一小块地方去观察。

观察角度需要具备近景、中景和远景不同的纹理效果，游戏一定要考虑这三点。

纹理也是有各个朝向的，一件衣服上，袖子和身体部分的衣服的纹理朝向肯定是不一样的，如果有多个纹理朝向的话，就可以观察到多方面材质的可能性，也可以保证材质的全面性，我们也会观察黑白灰情况下的效果，还有粗糙的纹理和光滑的纹理的效果。

![CH10_MoreMaterials_D_ComplexDetails16](../imgs/CH10_MoreMaterials_D_ComplexDetails16.jpg)

![CH10_MoreMaterials_D_ComplexDetails17](../imgs/CH10_MoreMaterials_D_ComplexDetails17.jpg)

![CH10_MoreMaterials_D_ComplexDetails5](../imgs/CH10_MoreMaterials_D_ComplexDetails5.jpg)

![CH10_MoreMaterials_D_ComplexDetails6](../imgs/CH10_MoreMaterials_D_ComplexDetails6.jpg)

在这个过程中还需要通过调整Roughness和Metallic两个值以确认不同质感下的纹理效果。

![CH10_MoreMaterials_D_ComplexDetails7](../imgs/CH10_MoreMaterials_D_ComplexDetails7.jpg)

![CH10_MoreMaterials_D_ComplexDetails8](../imgs/CH10_MoreMaterials_D_ComplexDetails8.jpg)

为了丰富变化，制作过程中会增加一些随机效果。比如像布料使用久了会有地方被磨损，后来我们加一些细节，这里我增加的编织物上面的线头，这几个是完成的纹理的渲染效果。Base layer + Random wear 随机磨损 + Random thrum 随机线头。

![CH10_MoreMaterials_D_ComplexDetails9](../imgs/CH10_MoreMaterials_D_ComplexDetails9.jpg)

![CH10_MoreMaterials_D_ComplexDetails10](../imgs/CH10_MoreMaterials_D_ComplexDetails10.jpg)

![CH10_MoreMaterials_D_ComplexDetails11](../imgs/CH10_MoreMaterials_D_ComplexDetails11.jpg)

![CH10_MoreMaterials_D_ComplexDetails12](../imgs/CH10_MoreMaterials_D_ComplexDetails12.jpg)

![CH10_MoreMaterials_D_ComplexDetails13](../imgs/CH10_MoreMaterials_D_ComplexDetails13.jpg)

![CH10_MoreMaterials_D_ComplexDetails14](../imgs/CH10_MoreMaterials_D_ComplexDetails14.jpg)

下面是区分麻纱与细纱的部分，主要改变固有色纹理层，这样可以做出更精确的材质效果。但最重要的还是需要了解如何拆分材质，懂得拆分才会懂得组合，这样便能使用 Substance Designer 制作出更准确的材质。

![CH10_MoreMaterials_D_ComplexDetails15](../imgs/CH10_MoreMaterials_D_ComplexDetails15.jpg)

Substance不仅能制作出高品质的纹理，它庞大的材质库Substance Source还可以提供很多类型的材质纹理以供下载使用。根据项目的规格，当然还是有些纹理需要花点时间进行优化和处理。

程序纹理易于管理，制作效率极高，高效率自然导致大家就更愿意去创造更复杂更完美的资源。

几个简单节点很快就可以得到一个编织的效果，大概只要不到10分钟，如果用高模的话，可能需要一个小时，因为还要烘焙，只要把节点连好以后，就可以直接出我们需要的四张图的纹理。

但这个办法也有大挑战，比如中国风的刺绣，刺绣的排线太考究，几乎只能使用传统的方式来制作，对于 Substance Designer 还没有特别好的思路。

<br>

<br>

------

### 高精度贴图布料

用多层UV的高精度贴图，优点是精度更高，内存占有小，包体也更小。缺点也很明显，渲染消耗增加了，贴图采样数达到了上限，图案的设计需要前期2D和3D就要做好规划，这样才能保证3D能够实现。

![CH10_MoreMaterials_E_HighQualityTextures1](../imgs/CH10_MoreMaterials_E_HighQualityTextures1.jpg)

制作过程非常繁琐，制作方法也很自由，但是过于自由也是一个很头痛的问题。由于早期没有严格要求，所以模型制作人员就会比较随意的进行调配，导致每做出来的一个东西标准都是不一样的，所以过于自由在这方面，反而是一个很大的问题。

这是闪暖中一款裙子的效果，仔细看上面的花纹，并不是tiling的效果，是有一定的随机变化的，上面的花纹是通过UV进行拼接的，可以看一下这个裙子的UV，非常的复杂凌乱，但是这样才能得到我们想要的效果。

![CH10_MoreMaterials_E_HighQualityTextures2](../imgs/CH10_MoreMaterials_E_HighQualityTextures2.jpg)

简单介绍一下制作的流程，先是底色，然后增加布纹、或者花纹，有时候也会用两层UV同时来制作图案的效果，然后会加一些层次变化。

下面这套是闪暖中花纹比较复杂的衣服，整个角色模型可以放大到这个精度去观察。

![CH10_MoreMaterials_E_HighQualityTextures3](../imgs/CH10_MoreMaterials_E_HighQualityTextures3.jpg)

<br>

<br>

------

### 材质编辑器

参考一下闪暖材质编辑的思路。

早期的思路叫ABCD组合法，任意组合可以得到11种变化可能性，但是每个字母都有5个变种的话，可以得到625种变化可能性。

![CH10_MoreMaterials_Z_MaterialEditor](../imgs/CH10_MoreMaterials_Z_MaterialEditor.png)

开发材质编辑工具，这样就可以进行多种的材质组合和材质变化，有了材质编辑工具，可以得到非常多的材质种类。

![CH10_MoreMaterials_Z_MaterialCombination](../imgs/CH10_MoreMaterials_Z_MaterialCombination.jpg)

<br>

<br>

------

（待续）

