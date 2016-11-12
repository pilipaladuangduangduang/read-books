# Splitter类的使用和源码分析

标签（空格分隔）： Guava使用及源码分析

---

String自带了split()方法，为何还需要Splitter类来帮助我们分隔字符串呢？原因如下：

``` 
String testString = "Monday,Tuesday,,Thursday,Friday,,";
//String自带的split方法，不会忽略数组中间的空字符串
String[] parts = testString.split(",");//parts: [Monday, Tuesday, , Thursday,Friday]
``` 

## Splitter的使用

``` 
public class SplitterTest {

    // 定义分隔符
    private static final String SEPARATOR = ",";

    // 定义分隔KV字符串的分隔符
    private static final String MAP_SEPARATOR = "=";

    // 普通的分隔器
    private static Splitter SPLITTER = Splitter.on(SEPARATOR);

    // 忽略空字符串("")的分隔器，但不忽略含空格的字符串("  ")
    private static Splitter NO_EMPTY_SPLITTER = SPLITTER.omitEmptyStrings();

    // 为字符串去除两边空格的分隔器，可以去除这样的("  hehe  ")，但不能去除空字符串("")，也不能去除只含空格的字符串(" ")
    private static Splitter TRIM_SPLITTER = SPLITTER.trimResults(CharMatcher.WHITESPACE);

    // 先正常分隔，然后转换为Map
    private static Splitter.MapSplitter MAP_SPLITTER = SPLITTER.withKeyValueSeparator(MAP_SEPARATOR);

    private static String TRIM_STR = "Monday,Tuesday,,Thursday,Friday   , ,";

    private static String MAP_STR = "Washington D.C=Redskins,New York=Giants,Philadelphia=Eagles,Dallas=Cowboys";

    public static void main(String[] args) {

        // 和String.split()方法作用相同
        SPLITTER.split(TRIM_STR);

        // 该方法的作用其实就相当于，String.split()之后，对数组中每个元素进行trim()操作
        TRIM_SPLITTER.split(TRIM_STR);

        // 忽略空字符串
        NO_EMPTY_SPLITTER.split(TRIM_STR);

        // 转换为Map，{Washington D.C=Redskins, New York=Giants, Philadelphia=Eagles, Dallas=Cowboys}
        MAP_SPLITTER.split(MAP_STR);

        // 已指定长度切割字符串，这个方法真是好用极了
        String mobile = "18888888888";
        Splitter.fixedLength(0).split(mobile);// java.lang.IllegalArgumentException: The length may not be less than 1
        Splitter.fixedLength(1).split(mobile);// [1, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8]
        Splitter.fixedLength(2).split(mobile);// [18, 88, 88, 88, 88, 8]
        Splitter.fixedLength(3).split(mobile);// [188, 888, 888, 88]
        Splitter.fixedLength(4).split(mobile);// [1888, 8888, 888]
        Splitter.fixedLength(5).split(mobile);// [18888, 88888, 8]
        Splitter.fixedLength(6).split(mobile);// [188888, 88888]
        Splitter.fixedLength(7).split(mobile);// [1888888, 8888]
        Splitter.fixedLength(8).split(mobile);// [18888888, 888]
        Splitter.fixedLength(9).split(mobile);// [188888888, 88]
        Splitter.fixedLength(10).split(mobile);// [1888888888, 8]
        Splitter.fixedLength(11).split(mobile);// [18888888888]
        Splitter.fixedLength(12).split(mobile);// [18888888888]
        
        // 定义字符串分隔后的集合大小，规律一看便知
        String limit = "haha~heihei~hehe~meme";
        Splitter.on("~").limit(0).split(limit);// java.lang.IllegalArgumentException: must be greater than zero: 0
        Splitter.on("~").limit(1).split(limit);// [haha~heihei~hehe~meme]
        Splitter.on("~").limit(2).split(limit);// [haha, heihei~hehe~meme]
        Splitter.on("~").limit(3).split(limit);// [haha, heihei, hehe~meme]
        Splitter.on("~").limit(4).split(limit);// [haha, heihei, hehe, meme]
        Splitter.on("~").limit(5).split(limit);// [haha, heihei, hehe, meme]

    }

}
``` 

## Splitter的源码分析

``` 
@GwtCompatible(emulated = true)
public final class Splitter {

  /** 定义用于分隔的分隔符 */
  private final CharMatcher trimmer;
  
  /** 用于判断是否忽略分隔符之间的空字符 */
  private final boolean omitEmptyStrings;
  
  /** 该内部接口是对分隔后元素集合的抽象 */
  private final Strategy strategy;
  
  
  private final int limit;

  private Splitter(Strategy strategy) {
    this(strategy, false, CharMatcher.NONE, Integer.MAX_VALUE);
  }

  private Splitter(Strategy strategy, boolean omitEmptyStrings, CharMatcher trimmer, int limit) {
    this.strategy = strategy;
    this.omitEmptyStrings = omitEmptyStrings;
    this.trimmer = trimmer;
    this.limit = limit;
  }

  @CheckReturnValue
  // 参数虽然是char类型，但是都会被统一转回CharMatcher类型的
  public static Splitter on(char separator) {
    return on(CharMatcher.is(separator));
  }

  @CheckReturnValue
  // 返回Splitter实例的方法
  public static Splitter on(final CharMatcher separatorMatcher) {
    checkNotNull(separatorMatcher);

    return new Splitter(
        // 匿名函数实例化接口Strategy
        new Strategy() {
          @Override
          // 实现接口中的抽象方法
          public SplittingIterator iterator(Splitter splitter, final CharSequence toSplit) {
            // 匿名函数实例化抽象类SplittingIterator
            return new SplittingIterator(splitter, toSplit) {
              @Override
              // 实现抽象类中的抽象方法
              int separatorStart(int start) {
                return separatorMatcher.indexIn(toSplit, start);
              }

              @Override
              // 实现抽象类中的抽象方法
              int separatorEnd(int separatorPosition) {
                return separatorPosition + 1;
              }
            };
          }
        });
  }

  @CheckReturnValue
  public static Splitter on(final String separator) {
    checkArgument(separator.length() != 0, "The separator may not be the empty string.");
    
    // 字符串的长度为1时，转换为char类型
    if (separator.length() == 1) {
      return Splitter.on(separator.charAt(0));
    }
    return new Splitter(
        // 匿名函数实例化接口Strategy
        new Strategy() {
          @Override
          // 实现Strategy接口中的方法
          public SplittingIterator iterator(Splitter splitter, CharSequence toSplit) {
            // 匿名函数实例化抽象类SplittingIterator
            return new SplittingIterator(splitter, toSplit) {
              @Override
              // 实现抽象类的方法
              public int separatorStart(int start) {
                int separatorLength = separator.length();
                // 定义标签，用于循环语句的跳转，跳出双层for循环
                positions:
                for (int p = start, last = toSplit.length() - separatorLength; p <= last; p++) {
                  for (int i = 0; i < separatorLength; i++) {
                    if (toSplit.charAt(i + p) != separator.charAt(i)) {
                      continue positions;
                    }
                  }
                  return p;
                }
                return -1;
              }

              @Override
              // 实现抽象类的方法
              public int separatorEnd(int separatorPosition) {
                return separatorPosition + separator.length();
              }
            };
          }
        });
  }

  @CheckReturnValue
  @GwtIncompatible("java.util.regex")
  // 通过正则对象来分隔字符串
  public static Splitter on(final Pattern separatorPattern) {
    checkNotNull(separatorPattern);
    checkArgument(
        !separatorPattern.matcher("").matches(),
        "The pattern may not match the empty string: %s",
        separatorPattern);

    // 返回当前类的实例对象
    return new Splitter(
        // 匿名对象实现接口Strategy
        new Strategy() {
          @Override
          // 实现抽象方法
          public SplittingIterator iterator(final Splitter splitter, CharSequence toSplit) {
            // 定义匹配对象
            final Matcher matcher = separatorPattern.matcher(toSplit);
            return new SplittingIterator(splitter, toSplit) {
              @Override
              public int separatorStart(int start) {
                return matcher.find(start) ? matcher.start() : -1;
              }

              @Override
              public int separatorEnd(int separatorPosition) {
                return matcher.end();
              }
            };
          }
        });
  }

  @CheckReturnValue
  @GwtIncompatible("java.util.regex")
  // 传入正则表达式字符串，实际上还是使用上面的方法
  public static Splitter onPattern(String separatorPattern) {
    // Pattern.compile("xx") 返回Pattern对象
    return on(Pattern.compile(separatorPattern));
  }

  @CheckReturnValue
  // 根据指定一定长度来分隔字符串
  public static Splitter fixedLength(final int length) {
    checkArgument(length > 0, "The length may not be less than 1");

    return new Splitter(
        // 匿名对象实现接口Strategy
        new Strategy() {
          @Override
          // 实现接口Strategy的抽象方法
          public SplittingIterator iterator(final Splitter splitter, CharSequence toSplit) {
            // 匿名对象实现抽象类SplittingIterator
            return new SplittingIterator(splitter, toSplit) {
              @Override
              public int separatorStart(int start) {
                int nextChunkStart = start + length;
                return (nextChunkStart < toSplit.length() ? nextChunkStart : -1);
              }

              @Override
              public int separatorEnd(int separatorPosition) {
                return separatorPosition;
              }
            };
          }
        });
  }

  @CheckReturnValue
  // 忽略分隔符之间的空字符串
  public Splitter omitEmptyStrings() {
    return new Splitter(strategy, true, trimmer, limit);
  }

  @CheckReturnValue
  // 定义分隔字符串后集合中的元素的个数
  public Splitter limit(int limit) {
    checkArgument(limit > 0, "must be greater than zero: %s", limit);
    return new Splitter(strategy, omitEmptyStrings, trimmer, limit);
  }

  @CheckReturnValue
  // 对分隔字符串后集合中的各个元素进行trim操作，默认是对空格进行trim
  public Splitter trimResults() {
    return trimResults(CharMatcher.WHITESPACE);
  }

  @CheckReturnValue
  // 对指定的CharMatcher类型对结果集进行trim操作
  public Splitter trimResults(CharMatcher trimmer) {
    checkNotNull(trimmer);
    return new Splitter(strategy, omitEmptyStrings, trimmer, limit);
  }

  @CheckReturnValue
  // 分隔指定的字符串
  public Iterable<String> split(final CharSequence sequence) {
    checkNotNull(sequence);

    // 匿名对象获取接口的实例
    return new Iterable<String>() {
      @Override
      // 实现接口中的抽象方法，返回SplittingIterator
      public Iterator<String> iterator() {
        return splittingIterator(sequence);
      }

      @Override
      public String toString() {
        return Joiner.on(", ")
            .appendTo(new StringBuilder().append('['), this)
            .append(']')
            .toString();
      }
    };
  }

  private Iterator<String> splittingIterator(CharSequence sequence) {
    return strategy.iterator(this, sequence);
  }

  @CheckReturnValue
  @Beta
  // 直接返回List集合
  public List<String> splitToList(CharSequence sequence) {
    checkNotNull(sequence);

    Iterator<String> iterator = splittingIterator(sequence);
    List<String> result = new ArrayList<String>();

    while (iterator.hasNext()) {
      result.add(iterator.next());
    }

    return Collections.unmodifiableList(result);
  }

  @CheckReturnValue
  @Beta
  // 用于分隔KV字符串的分隔器
  public MapSplitter withKeyValueSeparator(String separator) {
    return withKeyValueSeparator(on(separator));
  }

  @CheckReturnValue
  @Beta
  public MapSplitter withKeyValueSeparator(char separator) {
    return withKeyValueSeparator(on(separator));
  }

  @CheckReturnValue
  @Beta
  public MapSplitter withKeyValueSeparator(Splitter keyValueSplitter) {
    return new MapSplitter(this, keyValueSplitter);
  }

  @Beta
  public static final class MapSplitter {
    private static final String INVALID_ENTRY_MESSAGE = "Chunk [%s] is not a valid entry";
    private final Splitter outerSplitter;
    private final Splitter entrySplitter;

    private MapSplitter(Splitter outerSplitter, Splitter entrySplitter) {
      this.outerSplitter = outerSplitter; // only "this" is passed
      this.entrySplitter = checkNotNull(entrySplitter);
    }

    @CheckReturnValue
    public Map<String, String> split(CharSequence sequence) {
      Map<String, String> map = new LinkedHashMap<String, String>();
      for (String entry : outerSplitter.split(sequence)) {
        Iterator<String> entryFields = entrySplitter.splittingIterator(entry);

        checkArgument(entryFields.hasNext(), INVALID_ENTRY_MESSAGE, entry);
        String key = entryFields.next();
        checkArgument(!map.containsKey(key), "Duplicate key [%s] found.", key);

        checkArgument(entryFields.hasNext(), INVALID_ENTRY_MESSAGE, entry);
        String value = entryFields.next();
        map.put(key, value);

        checkArgument(!entryFields.hasNext(), INVALID_ENTRY_MESSAGE, entry);
      }
      return Collections.unmodifiableMap(map);
    }
  }

  // 该内部接口十分的重要，是对分隔后结果集的抽象
  private interface Strategy {
    Iterator<String> iterator(Splitter splitter, CharSequence toSplit);
  }

  // 静态内部抽象类，我的理解该类就是为了实现Strategy中的方法而产生的
  private abstract static class SplittingIterator extends AbstractIterator<String> {
    final CharSequence toSplit;
    final CharMatcher trimmer;
    final boolean omitEmptyStrings;

    abstract int separatorStart(int start);

    abstract int separatorEnd(int separatorPosition);

    int offset = 0;
    int limit;

    protected SplittingIterator(Splitter splitter, CharSequence toSplit) {
      this.trimmer = splitter.trimmer;
      this.omitEmptyStrings = splitter.omitEmptyStrings;
      this.limit = splitter.limit;
      this.toSplit = toSplit;
    }

    @Override
    protected String computeNext() {

      int nextStart = offset;
      while (offset != -1) {
        int start = nextStart;
        int end;

        int separatorPosition = separatorStart(offset);
        if (separatorPosition == -1) {
          end = toSplit.length();
          offset = -1;
        } else {
          end = separatorPosition;
          offset = separatorEnd(separatorPosition);
        }
        if (offset == nextStart) {

          offset++;
          if (offset >= toSplit.length()) {
            offset = -1;
          }
          continue;
        }

        while (start < end && trimmer.matches(toSplit.charAt(start))) {
          start++;
        }
        while (end > start && trimmer.matches(toSplit.charAt(end - 1))) {
          end--;
        }

        if (omitEmptyStrings && start == end) {
          // Don't include the (unused) separator in next split string.
          nextStart = offset;
          continue;
        }

        if (limit == 1) {

          end = toSplit.length();
          offset = -1;

          while (end > start && trimmer.matches(toSplit.charAt(end - 1))) {
            end--;
          }
        } else {
          limit--;
        }

        return toSplit.subSequence(start, end).toString();
      }
      return endOfData();
    }
  }
}
``` 

