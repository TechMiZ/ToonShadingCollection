# Toon Shading Collection 

## CH04a - Direct Specular on Hair 头发高光（主光）

<br>

二次元头发高光，俗称天使环。

![CH04a_hairDirectSpecular_0_Intro](../imgs/CH04a_hairDirectSpecular_0_Intro.png)

头发具有各项异性，各样异性就是物体在微观表面，各个方向的粗糙程度不同，然后它是这种性质的宏观表现。右边的两个金属球，左边是GGX高光，右边的是各项异性的GGX高光，可以很清楚的看出各项异性高光的特征，所以头发显示出来的高光就是环状的高光，而不是斑点状的高光，这一点很多动画中也会有体现。

还原的话主要是通过计算各项异性或者直接画到贴图上，用UV采样的方式。

头发高光最重要的两点就是好看的形状和动态的反馈。

<br>

------

### 各向异性高光

![CH04a_hairDirectSpecular_A_AnisoToonHairExample1](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairExample1.jpg)

![CH04a_hairDirectSpecular_A_AnisoToonHairExample2](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairExample2.jpg)

*↑Kajiya-Kay模型的卡通风格效果展示*

<br>对于二次元头发的渲染，第一反应是直接搬运写实渲染的各向异性头发高光模型，比如Kajiya-Kay模型：一层高频，一层低频，再加上一个切线扰动贴图达到各向异性效果， 要想使得高光沿着每根发束移动，还需要美术在头发模型的垂直方向进行uv展开。最好展成方形UV且与法线贴图方向一致，否则会出现偏移方向相反。

如果模型拓扑较为复杂，UV展开较难做到全部垂直，那就要用Flowmap来梳理高光的形状（也有意见说，卡通头发高光形状既不物理也不重复，无法使用flowmap，至少必须拉直UV避免锯齿，再小心手绘处高光偏移和粗糙度凑出高光形状，非常麻烦）。

通过使用合适的法线偏移贴图，我们可以达到日式卡通中W型头发高光的效果。

<br>

![CH04a_hairDirectSpecular_A_AnisoHairKK](../imgs/CH04a_hairDirectSpecular_A_AnisoHairKK.jpg)

![CH04a_hairDirectSpecular_A_AnisoJitterMap](../imgs/CH04a_hairDirectSpecular_A_AnisoJitterMap.jpg)

*↑偏移贴图*

 <br>

**优点：** 最接近真实头发效果。

**缺点：** 

- 移动太过自由，容易看起来油腻。
- 对贴图精度和UV展开要求比较高。

**优化：**

- 根据性能要求，可以考虑叠加两个高光项或者只保留一层高光。
- 一旦视角改变，高光带很容易就会乱掉，可以限死x角度避免光带倾斜，并且限制y轴范围避免光带移动到不合适的范围。

<br>

![CH04a_hairDirectSpecular_A_AnisoToonHairHonkai3MMD1](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairHonkai3MMD1.jpg)

![CH04a_hairDirectSpecular_A_AnisoToonHairHonkai3MMD2](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairHonkai3MMD2.jpg)

![CH04a_hairDirectSpecular_A_AnisoToonHairHonkai3MMD3](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairHonkai3MMD3.jpg)

![CH04a_hairDirectSpecular_A_AnisoToonHairHonkai3MMD](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairHonkai3MMD.jpg)

*↑崩坏3角色MMD：对两层高光设有不同亮度和色相，这样使得高光层次更为丰富*

![CH04a_hairDirectSpecular_A_AnisoToonHairNetEase](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairNetEase.png)

*↑忘川风华录头发高光*

<br>

多吐槽一下，这种高光用在卡通头发上，总觉得油腻！很少看见舒服的，只觉得米哈游官方的八重樱MMD还不错（米厂另外几个MMD我也觉得头发特油）。如有见多识广的大佬们见过更多成功范例，欢迎推荐。

<br>

#### 变体——梯度化

上面的各向异性高光还可以进一步加入梯度化，配合偏移和强度mask，做出卡通化的多变形态。

![CH04a_hairDirectSpecular_A_AnisoToonHairStylized1](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairStylized1.png)

![CH04a_hairDirectSpecular_A_AnisoToonHairStylized2](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairStylized2.jpg)

![CH04a_hairDirectSpecular_A_AnisoToonHairStylizedExample](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairStylizedExample.jpg)

<br>

还有米哈游展示的一种做法，感觉有点过于复杂化了：

把每缕头发模型在垂直方向进行UW展开，以便高光可以沿着每根发束移动。然后从将每一缕从左侧向右侧填充0到1，用来标识动态生成的高光形状的起始和结束位置。使用几个曲线定义的模板来描述头发高光的基本形状，然后使用抖动噪声纹理来调制头发高光的粗细变化。材质方面有很多参数用来控制生成图案的形状，位置、偏移、宽度、抖动比例等，通过调整这些参数，可以根据需要获得各种不同的形状。

![CH04a_hairDirectSpecular_A_AnisoToonHairComplicated0](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairComplicated0.jpg)

![CH04a_hairDirectSpecular_A_AnisoToonHairComplicated](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairComplicated.jpg)

*↑米哈游分享的卡通化各向异性高光做法*

![CH04a_hairDirectSpecular_A_AnisoToonHairComplicated1](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairComplicated1.jpg)

![CH04a_hairDirectSpecular_A_AnisoToonHairComplicated2](../imgs/CH04a_hairDirectSpecular_A_AnisoToonHairComplicated2.jpg)

*↑民间还原的米哈游分享效果*

<br>

<br>

#### 常见模型详解与调整示例

头发上使用最广泛的各向异性模型是KK模型，KK是一个经验公式，它最初在1989年发表，ATS在siggraph和GDC中分享了一篇Hair Rendering and Shading，这里面使用了KK来计算头发的高光，这也是各种资料中最常见的方式。

![CH04a_hairDirectSpecular_A_Models1](../imgs/CH04a_hairDirectSpecular_A_Models1.png)

KK最核心的计算就是将bilinphong的NDotH的法线N改为了切线T，将切线加入到了光线的计算中。然后从而得到了各项异性的环状高光。下面的代码就是KK的各项异性计算，这里就不详细讲解了。

值得注意的是，图片中的切线方向是沿发丝方向的。而一般制作时，切线方向通常是垂直于发丝的，所以这个通常是使用副切线来计算的。

因为KK发表实在是太久了，所以03年的时候就有了一个Marschner模型提出。在下面的图左边的是KK模型，中间的是Marschner的模型，右边是真实的头发。可以看出这个模型会更加物理，这个模型描述了光在头发中的传播的主要路径，分别是反射的光路R，先透射再反射再透射的光路TRT，透射再透射的光路TT。

![CH04a_hairDirectSpecular_A_Models2](../imgs/CH04a_hairDirectSpecular_A_Models2.png)

不过这里不细讲这个模型，讲这三条光路的主要原因是他们贡献了头发高光的主要特征，分别是主要镜面反射高光，次要镜面反射高光和透光，主要镜面反射高光和次要镜面反射高光，就是卡通头发中经常使用到两层高光叠加计算的来源。

Ok，那么使用上述各项异性计算，我们就可以获得围绕头发一圈的环状高光了，之后再加上遮罩和扰动便是常见的各项异性高光表现了。这里我提亮了第一层的高光强度让高光明显了一点。

![CH04a_hairDirectSpecular_A_Models3](../imgs/CH04a_hairDirectSpecular_A_Models3.png)

另外在KK的计算中有一个Smoohstep的函数，其中的a可以控制单向的渐变过渡，这样就可以获得左图这种一软一硬的过渡变化了。

不过在这其中还有一个问题，那就是切线修正的问题。

各项异性的计算是依赖于切线的，但是没有经过处理的切线计算出来，高光会非常混乱，就像右图一样，统一切线的方向，才能计算出像左图一样整洁的高光形态。

![CH04a_hairDirectSpecular_A_Models4](../imgs/CH04a_hairDirectSpecular_A_Models4.png)

对于切线的纠正，主要有两种方式，一种是打直UV，一种是绘制Flow Map。因为切线的方式是和UV的U轴有关的。所以打直UV，以统一切线方向是最常见的方式，另外也有使用Houdini、krita之类的软件手动绘制或者生成Flow Map来指定切线方向的方式，而且如果头发是一个类似于球体的形状，也可以虚拟一个球心，然后通过计算确定切线的方向。

![CH04a_hairDirectSpecular_A_Models5](../imgs/CH04a_hairDirectSpecular_A_Models5.png)

<br>

<br>

------

### MatCap伪高光

![CH04a_hairDirectSpecular_B_MatCapToonHairExampleUTS](../imgs/CH04a_hairDirectSpecular_B_MatCapToonHairExampleUTS.png)

*↑UTS的头发高光*

UTS里使用视线空间法线采样matcap贴图做出假高光，法线贴图做出各向异性，加上遮罩图调整出某些位置的渐隐，也能达到不错的效果。

**优点：** 高光形状的自由度比较高。

**缺点：** 从任何视角方向看都只能看到为指定视角设计的高光，对自然披散的头发效果最佳，但对扎起的头发、刺猬头等发型不一定看上去效果好。

<br>

<br>

------

### UV上下移动高光

《樱花大战》中是把UV排列整齐，通过uv采样遮罩图精准控制到每根发丝，然后把高光贴图贴上去，通过高光贴图的上下移动来表现动态的变化。

**优点：** 高光形状的自由度比较高，整体观感比较好控制。

**缺点：** 展UV调整位置比较繁琐。

继续吐槽，新樱花大战的头发，静态还行，动态看也觉得很油腻。是不是动态越强的头发越油腻？

![CH04a_hairDirectSpecular_B_MatCapToonHairTextures](../imgs/CH04a_hairDirectSpecular_B_MatCapToonHairTextures.jpg)

![CH04a_hairDirectSpecular_B_MatCapToonHairExample](../imgs/CH04a_hairDirectSpecular_B_MatCapToonHairExample.jpg)

<br>

<br>

------

### UV2固定位置缩放高光

蓝色协议为了头发高光点搞得超麻烦的一种做法。简而言之是先画高光，然后通过预先烘培的贴图去计算出来高光的重心，之后再根据视角去缩放高光的大小。因为绘制的高光没有动态反馈会比较死板，所以加入对相对应的动态变化。

![CH04a_hairDirectSpecular_D_SpecialSolution1](../imgs/CH04a_hairDirectSpecular_D_SpecialSolution1.png)

![CH04a_hairDirectSpecular_D_SpecialSolution2](../imgs/CH04a_hairDirectSpecular_D_SpecialSolution2.png)

他们的目标是让光点随着镜头距离而缩小，并不会根据镜头角度移动，整个过程也和法线无关，和一般人想实现的各向异性效果完全不同。但为了实现它，使用了UV2单独为这些高光元素准备UV，这并不是为了拉直UV，而是为了精度和留出可供缩放的空白区域。

![CH04a_hairDirectSpecular_D_SpecialSolution3](../imgs/CH04a_hairDirectSpecular_D_SpecialSolution3.jpg)

然后用一张RGBA控制图来处理。R通道是基本亮度，GB通道记录的是当前像素距离光斑中心点的距离值，A通道不明，有可能是缩放幅度。

具体做法是，当渲染一个像素时，根据读取到的BG通道数据，和当前UV相加，得到光斑中心。然后反过来减这个BG值同时进行一个根据距离的缩放。数值放大的时候，光斑在视觉上就会缩小。

这种做法可以确保光斑的边缘形状和R通道保持一致。

但是离线生成这张图就要做不少工作，必须让人手动指定每个光点的中心然后生成控制图，而且在固定距离看，人物光斑其实并没有任何变化。

但这种方式似乎是可以做和Normal挂钩的缩放的。一般做法不行，是因为光斑上每个像素点的Normal是不同的，根据Normal缩放的程度就不同，会导致光斑扭曲。但它这个方案，每个点都可以找到一个共同的中心点，只要使用这个中心点的Normal……

对，我取不到这个点的Normal，所以不行。

所以只能如此。

**优点：** 

- 虽然不能移动，但如果需求就要这样的话还算是一种独特的画风和处理方式，视觉上不易油腻。
- 高光形状设计比较自由。

**缺点：**

- 高光整体位置不能移动。
- 资源准备比较麻烦。
- 算法比较奇葩。

<br>

<br>

------

### 经典高光计算配合mask限制形状

算是一种偷懒方式，放弃了光照角度的正确性。如果能接受高光位置固定的话，就可以大胆采用。虽然不能自然滑动，但油腻度大大减少。也可以改出各种形状，比如有些画风的头发高光是画成小圆点形的。

战双在头发高光区的边缘都做了羽化，会让每片高光本身在随视角移动时还带有像气泡一样的膨胀收缩效果，很有意思。这样人物运动的时候起码这个高光带有点变化，不是完全的死高光。（这个方法也可以用在肌肉、嘴唇光点等希望产生较为固定但又能稍微带点变化的区域。）

这个头发高光处于暗部时，一种建议是，不应该隐藏，只能处理成稍微变暗，这是头发材质特性的一部分。

<br>

**优点：** 

- 不容易油腻。
- 形状设计的自由度还算高。
- 高光算法和其它区域保持统一，不用附加专门处理。

**缺点：** 位置过于固定。

![CH04a_hairDirectSpecular_C_MaskedToonHairGenshin](../imgs/CH04a_hairDirectSpecular_C_MaskedToonHairGenshin.png)

*↑原神：NNN形高光*

![CH04a_hairDirectSpecular_C_MaskedToonHairPunishing](../imgs/CH04a_hairDirectSpecular_C_MaskedToonHairPunishing.png)

*↑战双：气泡点形高光*

<br>

光带在正视的时候看起来最美观，但是在自由视角就存在玩家把镜头转到头顶上的问题。在那个视角上头顶光带会在头的周围变成一个圈，非常违和。偷懒办法是禁止视角拉到头顶。

《子弹少女》给出一种办法是，在y轴角度到达一个阈值后，会把这个光带渐渐裁掉，但裁掉光带后必须在头顶位置显示一个新的光带（图中的中心散射光带），否则从玩家视觉角度，他们会觉得高光突然消失了，好像头发的光照特性突然改变了。

这个处理方法虽然不错，但并不是所有高光画风都能兼容。

![v2-CH04a_hairDirectSpecular_C_MaskedToonHairImprovement](../imgs/CH04a_hairDirectSpecular_C_MaskedToonHairImprovement.jpg)

<br>

#### 变体——反向高光

上面经典高光的做法，算出来的高光一定是中心强外围弱的，但我们经常看见一些画风的头发高光是外围强中心弱的。

于是可以直接把经典高光算法做个反向，一步到位实现效果。

潜在的问题是，假设正面打光，我们只能大致从正面观察才能看到最佳效果，而绕到背光面看头发时，整个高光区都亮了。取巧办法是，直接压暗这种背光视角中的高光，或者单独对头发高光动态调整光照方向。

![CH04a_hairDirectSpecular_C_MaskedToonHairInverse](../imgs/CH04a_hairDirectSpecular_C_MaskedToonHairInverse.png)

*↑上排：反向高光；下排：默认高光*

<br>

<br>

------

### 永久加亮（偷懒办法）

直接画在基础色贴图上或者相关mask上，永久加亮高光区域，做成死高光，管它呢！

甚至有的日本动画公司在为了表现头发固定的高光位置，依然使用了取巧的3D建模手段来实现，效果也还可以。

![CH04a_hairDirectSpecular_E_LazySolution](../imgs/CH04a_hairDirectSpecular_E_LazySolution.png)

<br>

<br>

------

### 头发高光画风参考

上下双层颜色的效果：

![CH04a_hairDirectSpecular_F_HighlightColorStyle1](../imgs/CH04a_hairDirectSpecular_F_HighlightColorStyle1.jpg)

横向变化的彩虹色高光：

![CH04a_hairDirectSpecular_F_HighlightColorStyle2](../imgs/CH04a_hairDirectSpecular_F_HighlightColorStyle2.png)

<br><br>

------





<br>

<br>

------





<br>

<br>

------



