# Preconditions类的使用和源码分析

标签（空格分隔）： Guava使用及源码分析

---

Preconditions类给我们带来了哪些便利之处呢？

``` 
// 一般我们校验某个参数是否为空
if(someObj == null){
    throw new IllegalArgumentException(" someObj must not be null");
}

// 通过Preconditions类来校验，又方便又简洁
Preconditions.checkNotNull(someObj,"someObj must not be null");
``` 

## Preconditions的使用

``` 
// 用于校验某个对象是否为空
Preconditions.checkNotNull(obj, "obj must not be null");

// 用于校验表达式必须为true
Preconditions.checkArgument(obj > 7, "obj must be gather than 7");

// 同上
Preconditions.checkState(true, "obj must be true");

// 用于校验你指定的索引值（第一个参数），是否在数组的大小、字符串的长度、集合的大小（第二个参数）中
Preconditions.checkElementIndex(7, list.size(), "can not gather than list's size");

// 同上
Preconditions.checkPositionIndex(7, str.length(), "can not gather than string length");

// 可以指定前索引和后索引，前后索引值都必须在正确的范围内，否则报错
Preconditions.checkPositionIndexes(3, 7, arr.length);
``` 

## Preconditions的源码分析

``` 
public final class Preconditions {
  private Preconditions() {}

  // 其实对执行语句的返回结果进行了封装~
  public static void checkArgument(boolean expression) {
    if (!expression) {
      throw new IllegalArgumentException();
    }
  }

  // 对执行语句的返回结果进行了封住哪个，并且自定义了异常的声明
  public static void checkArgument(boolean expression, @Nullable Object errorMessage) {
    if (!expression) {
      throw new IllegalArgumentException(String.valueOf(errorMessage));
    }
  }

  // 声明返回结果的模板，抛出不同明义的异常
  public static void checkArgument(
      boolean expression,
      @Nullable String errorMessageTemplate,
      @Nullable Object... errorMessageArgs) {
    if (!expression) {
      throw new IllegalArgumentException(format(errorMessageTemplate, errorMessageArgs));
    }
  }

  // 虽然实现和checkArg一样，但是方法名称给人的直观是不一样，这个第一看到应该适用于校验结果是否正确，而checkArg则是用于校验参数的
  public static void checkState(boolean expression) {
    if (!expression) {
      throw new IllegalStateException();
    }
  }

  // 同checkArg
  public static void checkState(boolean expression, @Nullable Object errorMessage) {
    if (!expression) {
      throw new IllegalStateException(String.valueOf(errorMessage));
    }
  }

  // 同checkArg
  public static void checkState(
      boolean expression,
      @Nullable String errorMessageTemplate,
      @Nullable Object... errorMessageArgs) {
    if (!expression) {
      throw new IllegalStateException(format(errorMessageTemplate, errorMessageArgs));
    }
  }

  // 校验一个引用是否指向了空对象，是就报错，否就返回该对象
  public static <T> T checkNotNull(T reference) {
    if (reference == null) {
      throw new NullPointerException();
    }
    return reference;
  }

  // 同上，只不过可以自定义异常说明
  public static <T> T checkNotNull(T reference, @Nullable Object errorMessage) {
    if (reference == null) {
      throw new NullPointerException(String.valueOf(errorMessage));
    }
    return reference;
  }

  // 使用了模板封装了异常声明
  public static <T> T checkNotNull(
      T reference, @Nullable String errorMessageTemplate, @Nullable Object... errorMessageArgs) {
    if (reference == null) {
      // If either of these parameters is null, the right thing happens anyway
      throw new NullPointerException(format(errorMessageTemplate, errorMessageArgs));
    }
    return reference;
  }

  // 检查索引值是否在指定的size里，可以使数组的、字符串的、集合的
  public static int checkElementIndex(int index, int size) {
    return checkElementIndex(index, size, "index");
  }

  // 检查索引值是否在指定的size里，可以使数组的、字符串的、集合的
  public static int checkElementIndex(int index, int size, @Nullable String desc) {
    // Carefully optimized for execution by hotspot (explanatory comment above)
    if (index < 0 || index >= size) {
      throw new IndexOutOfBoundsException(badElementIndex(index, size, desc));
    }
    return index;
  }

  private static String badElementIndex(int index, int size, String desc) {
    if (index < 0) {
      return format("%s (%s) must not be negative", desc, index);
    } else if (size < 0) {
      throw new IllegalArgumentException("negative size: " + size);
    } else { // index >= size
      return format("%s (%s) must be less than size (%s)", desc, index, size);
    }
  }

  // 同上
  public static int checkPositionIndex(int index, int size) {
    return checkPositionIndex(index, size, "index");
  }

  public static int checkPositionIndex(int index, int size, @Nullable String desc) {
    // Carefully optimized for execution by hotspot (explanatory comment above)
    if (index < 0 || index > size) {
      throw new IndexOutOfBoundsException(badPositionIndex(index, size, desc));
    }
    return index;
  }

  private static String badPositionIndex(int index, int size, String desc) {
    if (index < 0) {
      return format("%s (%s) must not be negative", desc, index);
    } else if (size < 0) {
      throw new IllegalArgumentException("negative size: " + size);
    } else { // index > size
      return format("%s (%s) must not be greater than size (%s)", desc, index, size);
    }
  }

  public static void checkPositionIndexes(int start, int end, int size) {
    // Carefully optimized for execution by hotspot (explanatory comment above)
    if (start < 0 || end < start || end > size) {
      throw new IndexOutOfBoundsException(badPositionIndexes(start, end, size));
    }
  }

  private static String badPositionIndexes(int start, int end, int size) {
    if (start < 0 || start > size) {
      return badPositionIndex(start, size, "start index");
    }
    if (end < 0 || end > size) {
      return badPositionIndex(end, size, "end index");
    }
    // end < start
    return format("end index (%s) must not be less than start index (%s)", end, start);
  }

  // 所有方法中就这个方法最有意思了，可以复用
  static String format(String template, @Nullable Object... args) {
    template = String.valueOf(template); // null -> "null"

    // start substituting the arguments into the '%s' placeholders
    StringBuilder builder = new StringBuilder(template.length() + 16 * args.length);
    int templateStart = 0;
    int i = 0;
    while (i < args.length) {
      int placeholderStart = template.indexOf("%s", templateStart);
      if (placeholderStart == -1) {
        break;
      }
      builder.append(template.substring(templateStart, placeholderStart));
      builder.append(args[i++]);
      templateStart = placeholderStart + 2;
    }
    builder.append(template.substring(templateStart));

    // if we run out of placeholders, append the extra args in square braces
    if (i < args.length) {
      builder.append(" [");
      builder.append(args[i++]);
      while (i < args.length) {
        builder.append(", ");
        builder.append(args[i++]);
      }
      builder.append(']');
    }

    return builder.toString();
  }
}
``` 








