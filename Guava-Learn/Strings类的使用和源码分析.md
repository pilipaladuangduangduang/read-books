# Strings类的使用和源码分析

标签（空格分隔）： Guava使用及源码分析

---

其实说白了，Guava中的Strings就相同与org.apache.commons.lang下面的StringUtils类，不过和StringUtils相比，方法还是太少了点~

## 使用Strings

``` 
// 获取到两个字符串中公共的前缀，返回nzc
Strings.commonPrefix("nzc233", "nzc666");

// 获取到两个字符串中获取到公共的后缀，返回3nzc
Strings.commonSuffix("233nzc", "nzc123nzc");

// 当字符串为""就返回null，否则返回原先的字符串
Strings.emptyToNull("");

// 当字符串为null就返回""，否则就返回原先的字符串
Strings.nullToEmpty(null);

// 当字符串为null或者为""，就返回true；否则返回false
Strings.isNullOrEmpty("");

// 往字符串后添加你指定的字符，第二个参数是用于限制添加完后字符串的长度，这里返回的字符串是：niezhichun666
Strings.padEnd("niezhichun", 13, '6');

// 往字符串前添加你指定的字符，第二个参数是用于限制添加完后字符串的长度，这里返回的字符串是：666niezhichun
Strings.padStart("niezhichun", 13, '6');

// 重复合并指定的字符串，第二个参数指定重复的次数，不能小于0，这里返回的字符串：呵呵哒呵呵哒呵呵哒
Strings.repeat("呵呵哒", 3);
``` 

## Strings源码分析

``` 
public final class Strings {

  private Strings() {}

  // 字符串为null就返回""，否则返回原先字符串
  public static String nullToEmpty(@Nullable String string) {
    return (string == null) ? "" : string;
  }

  // 字符串为""就返回null，否则返回原先字符串
  public static String emptyToNull(@Nullable String string) {
    return isNullOrEmpty(string) ? null : string;
  }

  // 字符串为null或""返回true，否则返回false
  public static boolean isNullOrEmpty(@Nullable String string) {
    return string == null || string.length() == 0; // string.isEmpty() in Java 6
  }

  // 往字符串前添加你指定的字符
  public static String padStart(String string, int minLength, char padChar) {
    checkNotNull(string);
    // 字符串长度大于你指定的长度，则返回原先的字符串
    if (string.length() >= minLength) {
      return string;
    }
    // 实现很简单，就是通过StringBuilder一直添加指定的字符
    StringBuilder sb = new StringBuilder(minLength);
    // 注意这里的i是字符串的长度开始的
    for (int i = string.length(); i < minLength; i++) {
      sb.append(padChar);
    }
    // 添加原先字符串
    sb.append(string);
    return sb.toString();
  }

  // 往字符串后添加指定的字符串，原理同上
  public static String padEnd(String string, int minLength, char padChar) {
    checkNotNull(string);
    if (string.length() >= minLength) {
      return string;
    }
    StringBuilder sb = new StringBuilder(minLength);
    sb.append(string);
    for (int i = string.length(); i < minLength; i++) {
      sb.append(padChar);
    }
    return sb.toString();
  }

  // 重复拼装字符串
  public static String repeat(String string, int count) {
    checkNotNull(string); 
    
    if (count <= 1) {
      // 重复的次数不能小于0，否则会报错
      checkArgument(count >= 0, "invalid count: %s", count);
      // 重复的次数等于0时，返回""；等于1时返回原先字符串
      return (count == 0) ? "" : string;
    }

    final int len = string.length();
    // 先转换成long类型，是为了防止相乘后得到的值溢出int类型的取值范围
    final long longSize = (long) len * (long) count;
    // 这里从long类型转回int类型，是用于后序检查相乘之后的值是否溢出了，溢出了再取右32位，然后两者比较肯定是不同的
    final int size = (int) longSize;
    if (size != longSize) {
      throw new ArrayIndexOutOfBoundsException("Required array size too large: " + longSize);
    }

    final char[] array = new char[size];
    // 截取整个字符串然后放到数组的开头
    string.getChars(0, len, array, 0);
    int n;
    // 这个for循环不是很懂？？？
    for (n = len; n < size - n; n <<= 1) {
      // 拷贝数组
      System.arraycopy(array, 0, array, n, n);
    }
    System.arraycopy(array, 0, array, n, size - n);
    return new String(array);
  }

  // 返回两个字符串的公共前缀
  public static String commonPrefix(CharSequence a, CharSequence b) {
    checkNotNull(a);
    checkNotNull(b);
    
    // 返回两个字符串的最小长度
    int maxPrefixLength = Math.min(a.length(), b.length());
    int p = 0;
    // a.charAt(p) == b.charAt(p)是核心代码
    while (p < maxPrefixLength && a.charAt(p) == b.charAt(p)) {
      p++;
    }
    // 这里不懂...
    if (validSurrogatePairAt(a, p - 1) || validSurrogatePairAt(b, p - 1)) {
      p--;
    }
    return a.subSequence(0, p).toString();
  }

  // 返回两个字符串的公共后缀，原理同上
  public static String commonSuffix(CharSequence a, CharSequence b) {
    checkNotNull(a);
    checkNotNull(b);

    int maxSuffixLength = Math.min(a.length(), b.length());
    int s = 0;
    // a.charAt(a.length() - s - 1) == b.charAt(b.length() - s - 1)是核心代码
    while (s < maxSuffixLength && a.charAt(a.length() - s - 1) == b.charAt(b.length() - s - 1)) {
      s++;
    }
    if (validSurrogatePairAt(a, a.length() - s - 1)
        || validSurrogatePairAt(b, b.length() - s - 1)) {
      s--;
    }
    return a.subSequence(a.length() - s, a.length()).toString();
  }

  
  @VisibleForTesting
  static boolean validSurrogatePairAt(CharSequence string, int index) {
    return index >= 0
        && index <= (string.length() - 2)
        && Character.isHighSurrogate(string.charAt(index))
        && Character.isLowSurrogate(string.charAt(index + 1));
  }
}
``` 





