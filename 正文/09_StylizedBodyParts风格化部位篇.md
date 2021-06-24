# Toon Shading Collection 

## CH09 - Stylized Body Parts 特殊风格化部位

<br>

这章收集一下零散的风格化角色效果。

<br>

------

### 效果概括融合

通过《犬夜叉》动画可以看出，除了瞳孔或光滑表面有高光之外，其他地方很少会有高光，并且头发的各向异性高光和边缘光合在了一起。这个“合在一起”的操作就很灵性了，可以考虑技术还原这种高度概括的画风，也可以提炼更独特的简化方向并实现它们。

![CH09_StylizedBodyParts_C_CustomStyleAnalysis](../imgs/CH09_StylizedBodyParts_C_CustomStyleAnalysis.jpg)

<br>

<br>

------

面部三角光

![CH09_StylizedBodyParts_A_NoseTriangleLightReference](../imgs/CH09_StylizedBodyParts_A_NoseTriangleLightReference.jpg)

![CH09_StylizedBodyParts_A_NoseTriangleLightRealization](../imgs/CH09_StylizedBodyParts_A_NoseTriangleLightRealization.png)

有些二次元手绘画风，有事没事都会在鼻梁边加一道一点也不科学的三角形“高光”，强调鼻子的立体感。

可以使用一张mask配合特别敏感的菲涅尔计算，实现脸稍稍侧过去就会出现三角光的效果。

<br>

<br>

------

### 脸部特殊高光

![CH09_StylizedBodyParts_B_FaceHighlight](../imgs/CH09_StylizedBodyParts_B_FaceHighlight.png)

*↑新樱花大战给角色脸上一些部位加强了反光*

<br>

有些手绘漫画风是会给脸上点些高光。

这个高光的算法可以是传统高光，也可以魔改光源方向比如改成视线方向，也可以反向边缘光，做成正面永久可见。

也有观点认为脸上别加高光，不然显得太油了。 

但嘴唇一般影响不大，鼻梁高光也还可以，可能颧骨上的高光不太好操作。

新樱花大战确实太油了，跟整体渲染方案也有关系。

![CH09_StylizedBodyParts_B_FaceHighlightPositive](../imgs/CH09_StylizedBodyParts_B_FaceHighlightPositive.png)

*↑战双的鼻梁和嘴唇高光，这就比较舒服（忽略右边脸的一层渐变加亮，自己加的）*

![CH13_Shadow_C_SimplifiedAO](../imgs/CH13_Shadow_C_SimplifiedAO.png)

*↑原神角色脸直接在固有色贴图上叠了一层淡淡的高光，位置还是鼻梁和嘴唇*

<br>

<br>

------

### 



<br>

------





<br>

------







<br>

------







<br>

------



