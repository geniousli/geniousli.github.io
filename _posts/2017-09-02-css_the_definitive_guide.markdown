---
title: Css The Definitive Guide
date: 2017-09-02
tags: html, css
---

The Definitive Guide css
--------

#### css 和 文档
    * 层叠： css中解决样式冲突的规则，称为层叠
    * 元素：
      ```text
          替换、非替换元素
          替换元素： 用来替换元素的内容部分，并非由文档内容直接表示。<img src='xxx'> 标记片段中不包含任何具体内容，只有一个属性
          非替换元素：大多数元素都是非替换元素。
         元素显示角色：
         块级(block-level)，行内(inline-level)
         display: none, inline, block, inline-block, list-item, run-in
      ```
    * css and html:
      1. link 标记： <link rel='stylesheet' type='text/css' href='sheet1.css' media='all' />
      2. media 属性： all, aural(语音合成器), braille, print, screen, tty, tv <link rel='stylesheet' type='text/css' href='sheet1.css' media='screen, tv'/> 在tv， 屏幕设备上使用共同的css样式。
      3. style元素， <style type='text/css'> xxxxx </style>
      4. @import 指令： @import url(style.css) screen, tv; @import 必须写在样式表中，2.， 必须写在文档开头，，出现在样式表中其他规则的前面.

#### 选择器

    * 规则结构： 选择器 + 声明块 h1(选择器) {color: red; background: yellow}(声明块)
    * 声明和关键字： 格式：属性： 关键字\数值;关键字在多个的情况下，由空格分隔
    * 选择器：
      1. 元素： html { color: red}
      2. 选择器分组： 使用，分隔 选择器 h1, p {color: grey}
      3. 通配选择器: * {color: red;}
      4. 类选择器: p.warning 多类选择器， class='urgent warning' 选择器如下:  .urgent.warning
      5. ID选择器: #ID
      6. 属性选择器:
         1.简单属性选择器： <h1 class='people'> hello </h1> , h1[class] {color: red;} img[alt] {border: 1px solid red;}， 可以多个属性同时选择： a[href][title] {color: red; font-weight: bold;}
         2. 属性值选择器： planet[name="1" {color: red;}]
         3. 部分属性值选择：主要应用在class上，因为class可能为多个， 不能直接p[class='name'] 当p以后有多个class时候，而是需要p[class~='name']
         4. 特定属性选择器： *[lang!="en"] {color: "white";}
     7. 使用文档结构：
        1. 后代选择器： h1 em {color: gray;}
        2. 子元素选择器： h1 > em {color: gray;}
     8. 伪类和伪元素：（可以为文档中不一定具体存在的结构制定样式， 会根据某种条件来应用部分样式）
         1, 伪类： a 特有的 :link, :visited, 其他的都存在的 :focus, :hover, :active, 其中a 的样式声明顺序为 :linkm, :visited, :hover, :active
         2. 伪元素选择器(动态的给html改变内容，样式，可以修改文档本身)： first-letter, :first-line, :before, :after,例如：h2:before{}content: '||'}


#### 结构和层叠
　> 继承是　从一个元素向其后代元素传递属性所采用的机制，样式冲突的解决机制：通过考虑声明的特殊性，声明来源，特殊性，这个决策过程就称为层叠(cascade)
    * 特殊性：
      1. 对于选择器中所给定的各个ＩＤ属性：0, 1, 0, 0
      2. 对于选择器中的各个类属性值　属性选择、伪类: 0, 0, 1, 0
      3. 选择器中各个元素和伪元素：0, 0, 0, 1
      4. 通配符选择器对特殊性没有任何贡献
      5. 内联样式的特殊性: 1, 0, 0, 0
    > #id 与　p[id='id'] 中，第一个贡献的为：0, 1, 0, 0第二个贡献为 0, 0, 1, 0

    * 重要性: !important p.dark {color: red !important; background: white !important}
    !important　没有特殊性，不过要与其他的分开考虑，所有!important的会分组在一起考虑，其中的冲突在内部解决。重要声明与非冲要声明相比，总是胜出。
    * 继承没有特殊性，通配符大于继承的特殊性。注意：不应该不加区别的使用通配符选择器。

    * 层叠规则：
      1. 找出所有规则，这些规则都包含与一个给定元素匹配的选择器
      2. 按照权重对应用到该元素的所有声明排序，标志!important的规则的权重高于没有的，按来源进行对应用到元素的所有声明排序，３中来源，　创作人员，读者，用户代理。排序为：　读者的重要声明，　２：创作人员的重要声明，３：创作人员的正常声明，　４：读者的正常声明，　５：用户代理声明
      3. 按特殊性对应用到给定元素的所有声明排序，　有较高特殊性的元素权重大与较低权重的元素
      4. 按出现顺序对应用到元素的所有声明排序，一个声明在样式表或者文档中越后出现，他的权重越大，如果样式表中有导入的样式表，一般认为出现在导入样式表中的声明在前，主样式表中的所有样式表声明在后。（没有比较　多个导入样式表中的样式权重）

#### 值和单位：
    em ex为相对单位长度，　em定义为　一种给定字体的font-size的值，如果一个元素的font-size 为１４px，　那em为14px,　1ex = 0.5em

#### 字体(先略过)

#### 文本属性
　> 文本是内容，　而字体用于显示内容

  * text-indent: 数值： length, percentage, inherit，应用于： 块，继承： 有，百分数：相对于包含块的宽度
  * text-align: 数值: left, right, center, justify, inherit, 应用于：块，继承性：有
  * line-height: 值：length, percentage, number, normal, inherit，初始值: normal, 应用于：所有元素，继承性：有

#### 基本视觉格式化：（书中的css 版本为 2.１　其中大量的规则已经不能使用了）
    * 基本框：css假定每个元素都会生成一个或多个矩形框，这成为元素框。包含：内边框, 边框，外边框，内容
    * 包含块：包含块是一个元素的上下文
    * 块元素：
      1. 水平格式化：元素的宽度：指的是　左外边界到右外边界的距离，可见区域：width + 内边距，规则：正常流中的块级元素框的水平部分总和等于　父元素的width,
      2. 水平属性：　margin-left, border-left, padding-left, width, padding-right, border-right, margin-right
      3. width,, margin-left, margin-right中的某个值设定为auto，其余两个就会计算指定为特定值，使元素的框宽度等于副元素的width,　如果三个都设定为非auto，会产生过分受限，这时候从会把　margin-right强制为auto。margin-left, margin-right 设定为auto, 会将元素居中。border不能使用白分数
      4. 一个元素的默认的高度由其内容决定，　高度还会受内容宽度影响。　段落越窄，相应的高度就会越高。
      5.　垂直属性：margin-top, border-top, padding-top, height, padding-bottom, border-bottom, margin-bottom, 属性值　之和必须等于　包含块的height, 如果一个正常流中的一个块元素的margin-top, margin-bottom设置为auto,他会自动计算为０，并不会让块垂直居中
      6. auto 如果height 为auto，　其高度正好为包含内容的高度，
      7. 合并垂直外边框：这种合并行为只应用于外边距，如果元素有内边距和边框，他们绝对不会合并。
  * 行内元素－－行内布局：　
    1. 基本概念：　匿名文本，em框,　内容区，行间距((font-size - line-height) / 2)，行内框（非替换元素，行内框的高度等于line-height，　替换元素，行内框高度等于内容区的高度　替换元素没有行间距）
    2. 规则：
       1. 内容区类似于一个块级元素的内容框
       2. 行内元素的背景应用于内容区　以及所有　内边距
       3. 行内元素的边框要包围内容区　及所有　内边距和边框
       4. 非替换元素的内边距，边框和外边距对汗内元素或者其生成的框没有垂直效果，也就是说，　他们呢不会影响元素行内框的高度
       5. 替换元素的外边距和边框　会影响该元素行内框的高度，相应的影响该元素行框的高度

   3. 行内　非替换元素：　
      1. font-size：定义内容（字体）高度，　line-height: 定义框高度，　(line-height - font-size) / 2 为行间距，　line-height 可以小于font-size导致行内框小于内容区，行间距为负数，造成行间重叠。
      2. 行框定义为　行中最高行内框的顶端到最低行内框低端之间的距离，为了避免行间重叠。
      3. vertical-align: top, bottom, middle etc,　描述基准线的位置。
      4. line-height: 数值的话：为font-size的缩放比例。该属性是可以继承的，从而在不同的ｆｏｎｔ-size中存在不同的line-height,
      5.　内边距，外边距，边框　可以应用到行内非替换元素，但是却不会影响行框的高度，可能会造成重叠。实际上，外边距不会应用到元素上，但是可以应用到两侧。可以将文本推离到两侧。
  4. 行内块元素(inline-block)：
     1. 类似于　图像放在行中

#### 边框：
    * 基本元素框：
      1. 元素的width：左内边　－－　右内边，　height: 上内边－－下内边，　不能应用到行内非替换元素, width: 白分数想对于　包含块的　width, 正常流中的元素很少设定height，height: 白分数相对于　包含块的height。
      2. 背景颜色会延伸到内边距中，而不会到外边距。
      3. margin: 白分数相对于包含块的width, margin 的左右，上下都是相对于包含块width
      3. padding: 白分数相对于包含块的width, padding 的左右，上下都是相对于包含块width      
      4. 值复制：　top right bottom left, top -> right, top -> bottom, right -> left。
      5. 边框：border-style: top right bottom left; (none, hidden, dotted, dashed, solid double, grove, ridge, inset, outset, inherit) 注意在边框指定为none 的时候，设定其他属性是没有作用的。可以设定单边样式：　border-top-style, border-right-style etc, border-width　同style类似（thin, medium, thick, length, inherit）, 单边设定border-top-width, border-color 同style 一样,单边设定 border-top-color: 简写：border-top: thick solid gray;全局边框: border: thick solid gray;


#### 颜色和背景：（略过）

#### 浮动和定位：
    *　属性：　float: left, right, none, inherit, 应用于：所有元素，　
    * 浮动会将元素从正常流中　删除，浮动元素包含块为　其最近的块级祖先元素。只要是浮动元素，就会生成一个块级框，回像块及元素一样表现和摆放。简单规则：
      1. 浮动元素的左右外边界不能超过其包含块的　左右内边界。
　　　2. 浮动元素的顶端　不能比其父元素的内边界更高
    * 定位：　position: static, relative, absolute, fixed, inherit
    * 对于一个非根元素，　如果其position为　relative, static包含块设定为最近的块级框、表单元格、行内块祖先框的内容边界构成
    * 对于非根元素，如果其position 为absolute，包含块设定为最近的position值不是static　的祖先元素，如果这个祖先是块级元素，　包含块设定为该元素的内边距边界。
    * relative: 元素想对于之前元素偏移位置，它原本所占据的空间仍然保留。
    * absolute: 从文档流中完全删除，并相对于包含块定位。元素定位之后会生成一个块级框。
    * fixed: 表现类似与absolute，不过包含块是视图本身。
    * relative, absolute, fixed, 描述偏移：　top, right, bottom, left, 用来描述　距离包含块最近边的偏移距离。width, height，　对于定位元素并不重要，因为可以通过四个属性来隐形的确定。
    * min-width, min-height, max-width, max-height　作用：　可以相对安全的混合使用不同的单位，　使用白分数的时候，可以设定长度限制。
    * 内容溢出和可见性：　overflow: visible(超出边框) | hidden(超出边框的被剪裁) | scroll(html中添加滚轮展示) | auto | inherit, visibility:  visible | hidden | collapse | inherit, visible　设定为展示内容，　hidden 隐藏内容，但是并不从文档流中删除，　区别于 display: none, 会从文档中删除，不占据位置，　所以 visibility　是可以继承的，可以设定父元素hidden, 子元素为visible
    - [还没有介绍 collapse呢]
    * 绝对定位：包含块： 最近的position值不为 static 的元素。通常简单的做法是， 选在一个元素作为绝对定位元素的包含块，将其position设定为relative，并没有偏移。元素绝对定位时候，还为其后代元素建立了一个包含块。文档可以滚动的话，绝对定位是随着文档滚动的，因为定位元素的包含块是文档流的一部分。外边距为 auto可以得到垂直居中的 效果，
    * 固定定位： 固定定位与绝对定位相似，只是包含块是 视窗。
    * 相对定位：

#### 家:

    1. 海，天空，星辰，公主，王子，烤箱（蛋糕），狗，毛绒玩具
