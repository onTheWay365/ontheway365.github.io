---
title: java空格、空白符处理
date: 2019-02-24 17:51:57
tags: [java]
---

对于java中的空格、空白符处理，我们经常会用到org.apache.commons.lang3包中的StringUtils。
例如，对于空格的判断，我们会使用到isBlank方法。源码如下：
<!--more-->

```
public static boolean isBlank(final CharSequence cs) {
    int strLen;
    if (cs == null || (strLen = cs.length()) == 0) {
        return true;
    }
    for (int i = 0; i < strLen; i++) {
        if (Character.isWhitespace(cs.charAt(i)) == false) {
            return false;
        }
    }
    return true;
}
```
可以看到，isBlank方法其内部使用的是Character.isWhitespace方法来判断空格的，
**该方法会认为普通空格、空白符为blank，但不会识别不换行空格**。

```
    /**
     * Determines if the specified character is white space according to Java.
     * A character is a Java whitespace character if and only if it satisfies
     * one of the following criteria:
     * <ul>
     * <li> It is a Unicode space character ({@code SPACE_SEPARATOR},
     *      {@code LINE_SEPARATOR}, or {@code PARAGRAPH_SEPARATOR})
     *      but is not also a non-breaking space ({@code '\u005Cu00A0'},
     *      {@code '\u005Cu2007'}, {@code '\u005Cu202F'}).
     * <li> It is {@code '\u005Ct'}, U+0009 HORIZONTAL TABULATION.
     * <li> It is {@code '\u005Cn'}, U+000A LINE FEED.
     * <li> It is {@code '\u005Cu000B'}, U+000B VERTICAL TABULATION.
     * <li> It is {@code '\u005Cf'}, U+000C FORM FEED.
     * <li> It is {@code '\u005Cr'}, U+000D CARRIAGE RETURN.
     * <li> It is {@code '\u005Cu001C'}, U+001C FILE SEPARATOR.
     * <li> It is {@code '\u005Cu001D'}, U+001D GROUP SEPARATOR.
     * <li> It is {@code '\u005Cu001E'}, U+001E RECORD SEPARATOR.
     * <li> It is {@code '\u005Cu001F'}, U+001F UNIT SEPARATOR.
     * </ul>
     *
     * <p><b>Note:</b> This method cannot handle <a
     * href="#supplementary"> supplementary characters</a>. To support
     * all Unicode characters, including supplementary characters, use
     * the {@link #isWhitespace(int)} method.
     *
     * @param   ch the character to be tested.
     * @return  {@code true} if the character is a Java whitespace
     *          character; {@code false} otherwise.
     * @see     Character#isSpaceChar(char)
     * @since   1.1
     */
    public static boolean isWhitespace(char ch) {
        return isWhitespace((int)ch);
    }
```
查看方法注释说明，除三种不换行空格({@code '\u005Cu00A0'},{@code '\u005Cu2007'}, {@code '\u005Cu202F'})外，其他列出的各种分隔符、制表符都会被认为是空格。

三种不换行空格，对应的unicode分别为\u00A0, \u2007, \u202F， 其中后四位为16进制的数字
可以通过[http://tool.oschina.net/hexconvert/](http://note.youdao.com/)转换为10进制数字，转换后分别为160、8199、8239。

除上面列出的空白符外，**还有一些中文的空白汉字占位符也会被认为是空格**
>
> 32 == 普通的英文半角空格
>
> 160 == no-break space （普通的英文半角空格但不换行，也就是html中的nbsp）
>
> 12288 == 中文全角空格 （一个中文宽度）
>
> 8194 == en空格 （半个中文宽度，ensp）
>
> 8195 == em空格 （一个中文宽度，emsp）
>
> 8197 == 四分之一em空格 （四分之一中文宽度）
>
> 相比平时的空格（32），nbsp拥有不间断（non-breaking）特性。即连续的nbsp会在同一行内显示。即使有100个连续的nbsp，浏览器也不会把它们拆成两行。


测试代码：

```
@Test
public void blankTest(){
	//普通空格
	Assert.assertTrue(Character.isWhitespace((char)32));
	//英文空白符
	Assert.assertTrue(Character.isWhitespace((char)9));
	Assert.assertTrue(Character.isWhitespace((char)10));
	Assert.assertTrue(Character.isWhitespace((char)11));
	Assert.assertTrue(Character.isWhitespace((char)12));
	Assert.assertTrue(Character.isWhitespace((char)13));
	Assert.assertTrue(Character.isWhitespace((char)28));
	Assert.assertTrue(Character.isWhitespace((char)29));
	Assert.assertTrue(Character.isWhitespace((char)30));
	Assert.assertTrue(Character.isWhitespace((char)31));
	//中文空白符
	Assert.assertTrue(Character.isWhitespace((char)12288));
	Assert.assertTrue(Character.isWhitespace((char)8194));
	Assert.assertTrue(Character.isWhitespace((char)8195));
	Assert.assertTrue(Character.isWhitespace((char)8197));
	//不换行空格
	Assert.assertFalse(Character.isWhitespace((char)160));
	Assert.assertFalse(Character.isWhitespace((char)8199));
	Assert.assertFalse(Character.isWhitespace((char)8239));

	String blankStr = MessageFormat.format("包{0}含{1}普{2}通{3}空{4}格、英{5}文{6}空{7}白{8}符、中{9}文{10}空{11}白{12}符、不{13}换{14}行{15}空{16}格的字符串"
			,(char)32, (char)9, (char)10, (char)11, (char)12, (char)13, (char)28, (char)29, (char)30, (char)31, (char)12288, (char)8194, (char)8195
			, (char)8197, (char)160, (char)8199, (char)8239);

	System.err.println(blankStr);
	//deleteWhitespace方法没法去除不换行空格
	System.err.println(StringUtils.deleteWhitespace(blankStr));
	//下面的方法没法去除28、29、30、31等空格
	System.err.println(StringUtils.replacePattern(blankStr, "[\\s\\p{Zs}]+", ""));
	//去除全部空格
	System.err.println(StringUtils.replacePattern(blankStr, "[\\u00A0\\u2007\\u202F\\p{javaWhitespace}]+", ""));
	//去除全部空格
	System.err.println(StringUtils.replacePattern(StringUtils.deleteWhitespace(blankStr), "[\\u00A0\\u2007\\u202F]+", ""));

}
```

参考：

[https://stackoverflow.com/questions/28295504/how-to-trim-no-break-space-in-java]

[http://www.cnblogs.com/yjf512/p/3216392.html]

[http://www.cnblogs.com/lcf/articles/807523.html]

[http://babelserver.org/java-string-trim-split.html]