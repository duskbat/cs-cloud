# 静态网站生成

<https://jamstack.org/generators/>  
显然现在静态网站生成器的选择还是比较多的, 作为一个程序员, 主要的几个关注点不外乎:

-   代码高亮
-   mermaid 支持
-   公式支持

## 代码块切换

说实话, 代码块切换是选择静态网站而不是纯 markdown 文件的主要原因

## 代码高亮

**后端代码高亮**可以选择 pygments, 说实话 pygments 效果一般般  
预览: <https://swapoff.org/chroma/playground/>

几个主流的**前端代码高亮**选择:

-   highlightjs
-   prismjs
-   shiki

不考虑渲染速度的前提下, shiki 的渲染效果最好, prism 次之, highlight 最差;

## 公式

KaTex 基本可以满足需求

## mermaid 支持

有的生成器对 mermaid 支持不够好, 比如 material of mkdocs

## 本站

本站还是选择了 material of mkdocs, 主要是不想跟 npm 打交道, python 用着挺舒服的;  
但是高亮效果跟 mermaid 效果差点意思
