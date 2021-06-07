# Toon Shading Collection 

## CH10 - More Materials 更多材质效果

<br>

现在的卡通渲染游戏想要刺激玩家眼球，除了之前几张探讨过的，可能还要融入越来越丰富的材质效果，所以这里列一下可以考虑的材质。大部分源于偏写实的项目，也不知道是不是应该、或者能不能改良成更卡通的感觉。

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

（待续）



<br>

<br>

------





<br>

<br>

------



