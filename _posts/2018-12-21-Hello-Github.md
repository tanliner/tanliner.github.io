---
layout:     post
title:      Markdown Syntax
subtitle:   Markdown beginning
date:       2018-12-21
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Markdown
---

### MarkDown的语法

字体：加粗、斜体<br/>
块：block<br/>
标题 1 级：文字底部添加等号<br/>
标题 2 级：文字底部添加减号<br/>
标题 1~6 级：# ~ ######<br/>
代码块：以 \`\`\` 开始，并以 \`\`\` 结束<br/>
超链接：链接地址，图片，Gif，点击图片跳转url <br/>
显示gif
注释：[//]:(this is comment)

#### [Markdown Syntax](https://www.markdownguide.org/basic-syntax/)
```markdown
**加粗**
_Italic_
__Bold__
__*Bold-and-Italic*__

>block

Heading level 1
==============

Heading level 2
--------------

# 一级标题
## 二级标题
### 三级标题

`code` usually are tips command

List
1.

2.
<br/>
1.
    1.1
// section
- dd
* aa
+ dd
    - dd

[title](Url)    // click title to view 'Url'
![](ImageUrl)   // show image: url is 'Url'
[![title](assest path)](Url)       // click image to view 'Url'

// gif
![Alt Text](https://media.giphy.com/media/vFKqnCdLPNOKc/giphy.gif)

// table
| column1 | column2 | column3 | column4 |
| ------- |:------- |:-------:|--------:|
|  默认  |左对齐    |   居中   |    右对齐|
```

#### Markdown 注释
```markdown
[comment]: <> (This is a comment, it will not be included)
[comment]: <> (in  the output file unless you use it in)
[comment]: <> (a reference style link.)
[//]: <> (This is also a comment.)
[//]: # (This may be the most platform independent comment)
[^_^]:
    commentted-out contents
    should be shift to right by four spaces (`>>`).
```

进入你的博客主页，新的文章将会出现在你的主页上.

**Update**<br/>
1. 修改了博客里的截图，添加了原作者的博客入口
2. 新建了仓库 [ImageHolder](https://github.com/tanliner/ImageHolder) 用来保存博客中需要的图片，尤其是ReadMe文档引用
3. MD 显示Gif [How to show gif in Markdown](https://stackoverflow.com/questions/34341808/is-there-a-way-to-add-a-gif-to-a-markdown-file)
![Alt Text](https://media.giphy.com/media/vFKqnCdLPNOKc/giphy.gif)

感谢 BY 的模板，[BY](https://github.com/qiubaiying/qiubaiying.github.io)

作者：Ltan
