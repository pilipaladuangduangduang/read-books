# Objects类的使用和源码分析

标签（空格分隔）： Guava使用及源码分析

---

## JDK中的Objects

在学习Guava中的Objects时，发现在JDK1.7之后也有一个名为Objects的类，里面为我们封装了一些常用的方法，所以我们先来看看JDK自带的Objects吧~

``` 
public final class Objects {
    private Objects() {
        throw new AssertionError("No java.util.Objects instances for you!");
    }

    // 两个对象都为null也是返回true
    public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }

   // 深度比较，如果是对象数组则会比较数组中的每个对象
    public static boolean deepEquals(Object a, Object b) {
        if (a == b)
            return true;
        else if (a == null || b == null)
            return false;
        else
            return Arrays.deepEquals0(a, b);
    }

    // 非空校验都替我们做了，然后求出对象的hashCode
    public static int hashCode(Object o) {
        return o != null ? o.hashCode() : 0;
    }

    // 算出一组对象的hashCode
    public static int hash(Object... values) {
        return Arrays.hashCode(values);
    }

    // 如果是null则会返回"null"
    public static String toString(Object o) {
        return String.valueOf(o);
    }

    // 如果是null，返回自己定义的字符串对象
    public static String toString(Object o, String nullDefault) {
        return (o != null) ? o.toString() : nullDefault;
    }

    // 比较两个继承了Comparator接口的对象
    public static <T> int compare(T a, T b, Comparator<? super T> c) {
        return (a == b) ? 0 :  c.compare(a, b);
    }

    // 检查对象是否为空
    public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }

    // 如果为null，声明抛出的异常
    public static <T> T requireNonNull(T obj, String message) {
        if (obj == null)
            throw new NullPointerException(message);
        return obj;
    }

    // 判断对象是否为null
    public static boolean isNull(Object obj) {
        return obj == null;
    }

    // 判断对象是否不为null
    public static boolean nonNull(Object obj) {
        return obj != null;
    }

    // java8新特性，没怎么看懂
    public static <T> T requireNonNull(T obj, Supplier<String> messageSupplier) {
        if (obj == null)
            throw new NullPointerException(messageSupplier.get());
        return obj;
    }
}
``` 

既然JDK8已经帮我们封装了这么多操作，那以后还是需要用起来~

学完了JDK自带的Objects，我们来看看Guava中的Objects吧

## Guava中Objects的使用

在看Guava中的Objects发现这个类已经过时了_(:зゝ∠)_取而代之的是<font color="FF2D2D">MoreObjects</font>类。

## Guava中MoreObjects的使用

``` 
// 参数是两个对象，如果第一个对象非空，则返回第一个对象，如果两个都为空则报错，否则返回第二个对象
MoreObjects.firstNonNull(first, second);

// 这个类中最有用的就是toStringHelper方法了，可以用于重写pojo中的toString方法
@Override
public String toString() {
    // 其中omitNullValues方法炒鸡有用~，忽略null
    return MoreObjects.toStringHelper(this).omitNullValues().add(field,fieldName).addValue(value).toString();
}
``` 

## Guava中MoreObjects的源码分析

``` 
public final class MoreObjects {

  // 参数是两个对象，如果第一个对象非空，则返回第一个对象，如果两个都为空则报错，否则返回第二个对象
  public static <T> T firstNonNull(@Nullable T first, @Nullable T second) {
    return first != null ? first : checkNotNull(second);
  }

  /**
   *   直接看demo
   *   // Returns "ClassName{}"
   *   MoreObjects.toStringHelper(this)
   *       .toString();
   *
   *   // Returns "ClassName{x=1}"
   *   MoreObjects.toStringHelper(this)
   *       .add("x", 1)
   *       .toString();
   *
   *   // Returns "MyObject{x=1}"
   *   MoreObjects.toStringHelper("MyObject")
   *       .add("x", 1)
   *       .toString();
   *
   *   // Returns "ClassName{x=1, y=foo}"
   *   MoreObjects.toStringHelper(this)
   *       .add("x", 1)
   *       .add("y", "foo")
   *       .toString();
   *
   *   // Returns "ClassName{x=1}"
   *   MoreObjects.toStringHelper(this)
   *       .omitNullValues()
   *       .add("x", 1)
   *       .add("y", null)
   *       .toString();
   */
  public static ToStringHelper toStringHelper(Object self) {
    return new ToStringHelper(self.getClass().getSimpleName());
  }

  // 同上
  public static ToStringHelper toStringHelper(Class<?> clazz) {
    return new ToStringHelper(clazz.getSimpleName());
  }

  // 同上
  public static ToStringHelper toStringHelper(String className) {
    return new ToStringHelper(className);
  }

  // 这才是重头戏
  public static final class ToStringHelper {
    // 可以自定义的类名称
    private final String className;
    // 
    private ValueHolder holderHead = new ValueHolder();
    private ValueHolder holderTail = holderHead;
    // 是否忽略null
    private boolean omitNullValues = false;

    private ToStringHelper(String className) {
      this.className = checkNotNull(className);
    }

    // 可以忽略null
    public ToStringHelper omitNullValues() {
      omitNullValues = true;
      return this;
    }
    
    // 往ValueHolder链表尾添加KV节点
    public ToStringHelper add(String name, @Nullable Object value) {
      return addHolder(name, value);
    }

    public ToStringHelper add(String name, boolean value) {
      return addHolder(name, String.valueOf(value));
    }

    public ToStringHelper add(String name, char value) {
      return addHolder(name, String.valueOf(value));
    }

    public ToStringHelper add(String name, double value) {
      return addHolder(name, String.valueOf(value));
    }

    public ToStringHelper add(String name, float value) {
      return addHolder(name, String.valueOf(value));
    }

    public ToStringHelper add(String name, int value) {
      return addHolder(name, String.valueOf(value));
    }

    public ToStringHelper add(String name, long value) {
      return addHolder(name, String.valueOf(value));
    }

    public ToStringHelper addValue(@Nullable Object value) {
      return addHolder(value);
    }

    public ToStringHelper addValue(boolean value) {
      return addHolder(String.valueOf(value));
    }

    public ToStringHelper addValue(char value) {
      return addHolder(String.valueOf(value));
    }

    public ToStringHelper addValue(double value) {
      return addHolder(String.valueOf(value));
    }

    public ToStringHelper addValue(float value) {
      return addHolder(String.valueOf(value));
    }

    public ToStringHelper addValue(int value) {
      return addHolder(String.valueOf(value));
    }

    public ToStringHelper addValue(long value) {
      return addHolder(String.valueOf(value));
    }
    
    // toString的过程中会将ValueHolder这个链表中的KV都分解出来
    @Override
    public String toString() {
      boolean omitNullValuesSnapshot = omitNullValues;
      String nextSeparator = "";
      StringBuilder builder = new StringBuilder(32).append(className).append('{');
      // 开始循环ValueHolder链表中的数据
      for (ValueHolder valueHolder = holderHead.next;
          valueHolder != null;
          valueHolder = valueHolder.next) {
        Object value = valueHolder.value;
        // omitNullValuesSnapshot为false时，允许添加null值
        if (!omitNullValuesSnapshot || value != null) {
          builder.append(nextSeparator);
          nextSeparator = ", ";

          if (valueHolder.name != null) {
            builder.append(valueHolder.name).append('=');
          }
          
          // 学习到了，判断类型是否为数组：value.getClass().isArray()
          if (value != null && value.getClass().isArray()) {
            Object[] objectArray = {value};
            // 通过Arrays.deepToString方法将数组对象转为String
            String arrayString = Arrays.deepToString(objectArray);
            builder.append(arrayString.substring(1, arrayString.length() - 1));
          } else {// 
            builder.append(value);
          }
        }
      }
      return builder.append('}').toString();
    }

    // 往链表尾部添加存储KV的节点（空）
    private ValueHolder addHolder() {
      ValueHolder valueHolder = new ValueHolder();
      holderTail = holderTail.next = valueHolder;
      return valueHolder;
    }

    // 在链表尾部添加节点，然后给对象中的属性赋值
    private ToStringHelper addHolder(@Nullable Object value) {
      ValueHolder valueHolder = addHolder();
      valueHolder.value = value;
      return this;
    }

    // 在链表尾部添加节点，然后给对象中的属性赋值，其中name不能为空
    private ToStringHelper addHolder(String name, @Nullable Object value) {
      ValueHolder valueHolder = addHolder();
      valueHolder.value = value;
      valueHolder.name = checkNotNull(name);
      return this;
    }
    
    // 存储toString中KV数据的链表类
    private static final class ValueHolder {
      String name;
      Object value;
      ValueHolder next;
    }
  }

  private MoreObjects() {}
}
``` 