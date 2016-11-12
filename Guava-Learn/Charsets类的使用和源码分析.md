# Charsets类的使用和源码分析

标签（空格分隔）： Guava使用及源码分析

---

在使用Charsets之前（原生JDK已经有了相同的实现），我们还是要思考这么一个问题：就是为什么要使用它？

## 使用Charsets

``` 
try {
    byte[] bs = "memeda".getBytes("UTF-8");
  // JDK肯定是支持UTF-8的，所以这个异常永远不会发生
} catch (UnsupportedEncodingException e) {
    e.printStackTrace();
}

// 无需捕获异常
byte[] bytes2 = "heheda".getBytes(Charsets.UTF_8);

// 原生JDK中也有类似的常量类了：StandardCharsets
byte[] bytes2 = "heheda".getBytes(StandardCharsets.UTF_8);

// 在不同编码转换的时候我们可以使用Charsets，其实Charsets就是为我们封装了JDK中Charset.forName()方法

// 就是有一点不好：需要JDK7以上的版本支持
``` 

源码也很简单，就是对java.nio.charset.Charset的forName()方法的封装，但是现在JDK7以后已经有一个常量类来代替Charsets了，就是java.nio.charset.StandardCharsets，所以说<font color="FF2D2D">这个类已经过时了</font>（毕竟原生中都有了，你还有不就显得多余了么~）

## Charsets源码

``` 
public final class Charsets {

  private Charsets() {}
    
  // ASCII 编码
  public static final Charset US_ASCII = Charset.forName("US-ASCII");

  // ISO-8859-1 编码
  public static final Charset ISO_8859_1 = Charset.forName("ISO-8859-1");

  // UTF-8 编码
  public static final Charset UTF_8 = Charset.forName("UTF-8");

  // UTF-16BE 编码
  public static final Charset UTF_16BE = Charset.forName("UTF-16BE");

  // UTF-16BE 编码
  public static final Charset UTF_16LE = Charset.forName("UTF-16LE");

  // UTF-16 编码
  public static final Charset UTF_16 = Charset.forName("UTF-16");

}
``` 