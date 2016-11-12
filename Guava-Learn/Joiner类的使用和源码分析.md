# Joiner类的使用和源码分析

标签（空格分隔）： Guava使用及源码分析

---

## 使用Joiner

``` 
/**
 * Guava中的Joiner类
 *     特点：不可变、线程安全的、可用于static修饰
 */
public class JoinerTest {

    private static final String SEPARATOR = "|";
    private static final String KV_SEPARATOR = "=";
    private static final Joiner JOINER = Joiner.on(SEPARATOR);
    private static final Joiner SKIP_NULL_JOINER = JOINER.skipNulls();
    private static final Joiner.MapJoiner MAP_JOINER = JOINER.withKeyValueSeparator(KV_SEPARATOR);

    private final String[] NORMAL_ARR = {"Nie", "Zhi", "Chun"};
    private final String[] HAS_NULL_ARR = {"Nie", "Zhi", null, "Chun"};

    @Test
    public void test() throws IOException {

        // 已指定的分隔符拼接数组或集合中的字符串，返回 Nie|Zhi|Chun
        JOINER.join(NORMAL_ARR);

        // 已指定的分隔符拼接数组或集合中的字符串（含有 null ），抛出NullPointerException
        JOINER.join(HAS_NULL_ARR);

        // 已指定的分隔符拼接数组或集合中的字符串（含有 null ），跳过 null 返回 Nie|Zhi|Chun
        SKIP_NULL_JOINER.join(HAS_NULL_ARR);

        // 已指定的分隔符拼接数组或集合中的字符串（含有 null ），替换 null 返回 Nie|Zhi|Replacement|Chun
        JOINER.useForNull("Replacement").join(HAS_NULL_ARR);

        // Joiner 不允许 skipNulls() 和 useForNull() 同时使用，源码一看便知
        SKIP_NULL_JOINER.useForNull("Missing");

        // Joiner 可以为 StringBuilder 添加字符串，返回的 sb.toString() 为 Nie|Zhi|Chun
        StringBuilder sb = new StringBuilder();
        JOINER.appendTo(sb,"Nie","Zhi","Chun");

        // Joiner 可以为 Appendable 接口的所有子类提供append操作，Appendable 是 append() 这一系列操作的抽象
        // FileWriter 就是 Appendable 接口的某一子类
        FileWriter fileWriter = new FileWriter(new File("path"));
        Date[] dateList = {new Date(), new Date()};
        JOINER.appendTo(fileWriter,dateList).flush();
        fileWriter.close();

        // MapJoiner 中需要定义用于关联 KV 的分隔符，返回 key=键#value=值
        // 这里 Maps 是 Guava 集合中的类（工具类）
        Map<String,String> testMap = Maps.newLinkedHashMap();
        testMap.put("key","键");
        testMap.put("value","值");
        MAP_JOINER.join(testMap);

    }

}
``` 

## Joiner源码解析

``` 
// 该注解用于类时，表示有可能会与 Google Web Toolkit 一起使用
// 该注解用于方法时，说明这个方法的返回值是 GWT 兼容的
// 总之不是很懂
@GwtCompatible
public class Joiner {

  // JSR-305注解，不是很懂
  @CheckReturnValue
  // 传入字符串类型分隔符，返回Joiner实例，Joiner私有了构造器，所以只能在内部new实例
  public static Joiner on(String separator) {
    return new Joiner(separator);
  }

  @CheckReturnValue
  // 传入char类型分隔符，返回Joiner实例，通过String.valueOf()将char类型转换为String类型
  public static Joiner on(char separator) {
    return new Joiner(String.valueOf(separator));
  }
  
  // 存储分隔符的引用
  private final String separator;

  // 私有构造方法，为分隔符赋值之前，非空检查下
  private Joiner(String separator) {
    this.separator = checkNotNull(separator);
  }

  // 私有构造方法，接受自身类型的参数，也是为分隔符赋值
  private Joiner(Joiner prototype) {
    this.separator = prototype.separator;
  }

  // 第一个参数是Appendable的子类，Appendable接口是对append()这一操作的抽象
  // 第二个参数是Iterable的子类，Iterable接口是对遍历容器元素这一操作的抽象
  public <A extends Appendable> A appendTo(A appendable, Iterable<?> parts) throws IOException {
    // 第二个实参是获取容器的iterator集合，用于b
    return appendTo(appendable, parts.iterator());
  }
  
  // 为Appendable子类添加Iterable子类操作的具体实现
  public <A extends Appendable> A appendTo(A appendable, Iterator<?> parts) throws IOException {
    // 非空检查
    checkNotNull(appendable);
    // 这里的判断手法可以防止分隔符添加的末尾，保证在末尾的永远是字符串
    if (parts.hasNext()) {
      appendable.append(toString(parts.next()));
      while (parts.hasNext()) {
        appendable.append(separator);
        appendable.append(toString(parts.next()));
      }
    }
    return appendable;
  }
  
  // 第一个参数是Appendable的子类
  // 第二个参数是一个数组
  public final <A extends Appendable> A appendTo(A appendable, Object[] parts) throws IOException {
    // Arrays.asList最终会返回一个ArrayList，而List都是实现了Iterable接口的子类
    return appendTo(appendable, Arrays.asList(parts));
  }

  public final <A extends Appendable> A appendTo(
      A appendable, @Nullable Object first, @Nullable Object second, Object... rest)
      throws IOException {
    return appendTo(appendable, iterable(first, second, rest));
  }

  // StringBuilder和StringBuffer都是Appendable的子类，实现原理一致
  public final StringBuilder appendTo(StringBuilder builder, Iterable<?> parts) {
    return appendTo(builder, parts.iterator());
  }

  // StringBuilder和StringBuffer都是Appendable的子类，实现原理一致
  public final StringBuilder appendTo(StringBuilder builder, Iterator<?> parts) {
    try {
      // 还有通过Appendable的实现方法来实现的
      appendTo((Appendable) builder, parts);
    } catch (IOException impossible) {
      throw new AssertionError(impossible);
    }
    return builder;
  }
  
  // StringBuilder和StringBuffer都是Appendable的子类，实现原理一致
  public final StringBuilder appendTo(StringBuilder builder, Object[] parts) {
    return appendTo(builder, Arrays.asList(parts));
  }

  // StringBuilder和StringBuffer都是Appendable的子类，实现原理一致
  public final StringBuilder appendTo(
      StringBuilder builder, @Nullable Object first, @Nullable Object second, Object... rest) {
    return appendTo(builder, iterable(first, second, rest));
  }

  // 组装容器中元素的方法
  @CheckReturnValue
  public final String join(Iterable<?> parts) {
    return join(parts.iterator());
  }

  @CheckReturnValue
  // 实际上还是通过StringBuilder来完成这一操作
  public final String join(Iterator<?> parts) {
    return appendTo(new StringBuilder(), parts).toString();
  }

  @CheckReturnValue
  // join的参数为数组，实现时还是会转为List（实现了Iterable接口）
  public final String join(Object[] parts) {
    return join(Arrays.asList(parts));
  }

  @CheckReturnValue
  // 参数可以不是容器或数组，可以使单个的对象（至少两个）
  public final String join(@Nullable Object first, @Nullable Object second, Object... rest) {
    return join(iterable(first, second, rest));
  }

  @CheckReturnValue
  // 替换null的方法，返回一个新的Joiner对象
  public Joiner useForNull(final String nullText) {
    checkNotNull(nullText);// 替换null的字符串不能为空
    return new Joiner(this) {
      @Override
      CharSequence toString(@Nullable Object part) {
        return (part == null) ? nullText : Joiner.this.toString(part);
      }

      @Override
      // 通过useForNull返回的Joiner对象不支持userForNull()方法
      public Joiner useForNull(String nullText) {
        throw new UnsupportedOperationException("already specified useForNull");
      }

      @Override
      // 通过userForNull返回的Joiner对象不支持skipNulls()方法
      public Joiner skipNulls() {
        throw new UnsupportedOperationException("already specified useForNull");
      }
    };
  }

  @CheckReturnValue
  // 忽略null对象的方法，返回一个新Joiner对象
  public Joiner skipNulls() {
    return new Joiner(this) {
      @Override
      public <A extends Appendable> A appendTo(A appendable, Iterator<?> parts) throws IOException {
        checkNotNull(appendable, "appendable");
        checkNotNull(parts, "parts");
        while (parts.hasNext()) {
          Object part = parts.next();
          if (part != null) {
            appendable.append(Joiner.this.toString(part));
            break;
          }
        }
        while (parts.hasNext()) {
          Object part = parts.next();
          if (part != null) {
            appendable.append(separator);
            appendable.append(Joiner.this.toString(part));
          }
        }
        return appendable;
      }

      @Override
      // skipNulls()返回的Joiner对象，不支持userForNull()方法
      public Joiner useForNull(String nullText) {
        throw new UnsupportedOperationException("already specified skipNulls");
      }

      @Override
      // skipNulls()返回的Joiner对象，不支持withKeyValueSeparator()方法
      public MapJoiner withKeyValueSeparator(String kvs) {
        throw new UnsupportedOperationException("can't use .skipNulls() with maps");
      }
    };
  }

  @CheckReturnValue
  // 获取用于处理KV字符串的MapJoiner类
  public MapJoiner withKeyValueSeparator(String keyValueSeparator) {
    return new MapJoiner(this, keyValueSeparator);
  }
  
  // 静态内部类，实例化时不需要依赖于外部类的对象实例
  public static final class MapJoiner {
    private final Joiner joiner;
    private final String keyValueSeparator;

    private MapJoiner(Joiner joiner, String keyValueSeparator) {
      this.joiner = joiner; // only "this" is ever passed, so don't checkNotNull
      this.keyValueSeparator = checkNotNull(keyValueSeparator);
    }

    public <A extends Appendable> A appendTo(A appendable, Map<?, ?> map) throws IOException {
      return appendTo(appendable, map.entrySet());
    }

    public StringBuilder appendTo(StringBuilder builder, Map<?, ?> map) {
      return appendTo(builder, map.entrySet());
    }

    @CheckReturnValue
    public String join(Map<?, ?> map) {
      return join(map.entrySet());
    }

    @Beta
    public <A extends Appendable> A appendTo(A appendable, Iterable<? extends Entry<?, ?>> entries)
        throws IOException {
      return appendTo(appendable, entries.iterator());
    }

    @Beta
    public <A extends Appendable> A appendTo(A appendable, Iterator<? extends Entry<?, ?>> parts)
        throws IOException {
      checkNotNull(appendable);
      if (parts.hasNext()) {
        Entry<?, ?> entry = parts.next();
        appendable.append(joiner.toString(entry.getKey()));
        appendable.append(keyValueSeparator);
        appendable.append(joiner.toString(entry.getValue()));
        while (parts.hasNext()) {
          appendable.append(joiner.separator);
          Entry<?, ?> e = parts.next();
          appendable.append(joiner.toString(e.getKey()));
          appendable.append(keyValueSeparator);
          appendable.append(joiner.toString(e.getValue()));
        }
      }
      return appendable;
    }

    @Beta
    public StringBuilder appendTo(StringBuilder builder, Iterable<? extends Entry<?, ?>> entries) {
      return appendTo(builder, entries.iterator());
    }

    @Beta
    public StringBuilder appendTo(StringBuilder builder, Iterator<? extends Entry<?, ?>> entries) {
      try {
        appendTo((Appendable) builder, entries);
      } catch (IOException impossible) {
        throw new AssertionError(impossible);
      }
      return builder;
    }

    @Beta
    @CheckReturnValue
    public String join(Iterable<? extends Entry<?, ?>> entries) {
      return join(entries.iterator());
    }

    @Beta
    @CheckReturnValue
    public String join(Iterator<? extends Entry<?, ?>> entries) {
      return appendTo(new StringBuilder(), entries).toString();
    }

    @CheckReturnValue
    public MapJoiner useForNull(String nullText) {
      return new MapJoiner(joiner.useForNull(nullText), keyValueSeparator);
    }
  }
  
  // toString()返回CharSequence对象，CharSequence是String的父类
  CharSequence toString(Object part) {
    checkNotNull(part); // checkNotNull for GWT (do not optimize).
    return (part instanceof CharSequence) ? (CharSequence) part : part.toString();
  }

  // 将一些单个对象组装成AbstractList对象，用于appendTo()方法
  private static Iterable<Object> iterable(
      final Object first, final Object second, final Object[] rest) {
    checkNotNull(rest);
    return new AbstractList<Object>() {
      @Override
      public int size() {
        return rest.length + 2;
      }

      @Override
      public Object get(int index) {
        switch (index) {
          case 0:
            return first;
          case 1:
            return second;
          default:
            return rest[index - 2];
        }
      }
    };
  }
}
``` 