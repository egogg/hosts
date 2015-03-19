# 解决JP-Markdown代码区块多出一个空行的问题

当结合使用JP-MarkDown插件和Caryon语法高亮插件的时候，代码区块内总会有一个多余的空行。这里通过新建一个Caryon主题并编辑Caryon主题的css文件来消除那个难看的空白行：
```css
.crayon-nums-content > div:last-child,
.crayon-pre > div:last-child {
  display: none;
}
```