# 解决 JP Markdown插件反斜线(\\)丢失的问题

在使用JP-MarkDown时，` ``` code ``` ` 所表示的代码块中，代码当中的反斜线\在文章发布后会丢失，于是代码中的 `printf("\n");` 会变为 `printf("n") …`

有人发现，这是因为插件对` ``` code ``` ` 所表示的代码、缩进所表示的代码，处理不同。对前者进行了一次 stripslashes()，把这个处理去掉就正常了。

`stripslashes()` 应该是多余的处理，不过还是保留，然后在其后将 \\\\ 替换为 \\\\\\\ 也可以。

**处理方法为：**

找到代码：`jetpack-markdown/lib/markdown/gfm.php`。在第150行代码 `$block = esc_html( $block );` 后面增加 `$block = str_replace('\\', '\\\\', $block);` 就可以了：
```php
public function do_codeblock_preserve( $matches ) {
    $block = stripslashes( $matches[3] );
    $block = esc_html( $block );
    $block = str_replace('\\', '\\\\', $block);
    $open = $matches[1] . $matches[2] . "\n";
    return $open . $block . $matches[4];
}
```