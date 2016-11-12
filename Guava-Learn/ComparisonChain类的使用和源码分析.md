# ComparisonChain类的使用和源码分析

标签（空格分隔）： Guava使用及源码分析

---

我们可以使用Comparator接口来比较两个对象，那么ComparisonChain有什么高明之处呢？

## ComparisonChain的使用

``` 
// 这是传统Comparator接口的比较方法
public int compare(T t1, T t2) {
    // 先按照置顶状态来比较
	int i = t2.top.compareTo(t1.top);
	if (i != 0) {
		return i;
	}
	// 再按照最新回复时间来比较
	i = t2.reply.compareTo(t1.reply);
	if (i != 0) {
		return i;
	}
	// 最后按照id来比较
	return t2.id.compareTo(t1.id);
}

// 这是使用了ComparisonChain类的比较方法
public int compare(T t1, T t2) {
    // 链式比较，炒鸡爽
    return ComparisonChain.start()
                          .compare(t2.top, t1.top)
                          .compare(t2.reply, t1.reply)
                          .compare(t2.id, t1.id)
                          .result();
}
``` 

## ComparisonChain的源码分析

``` 
public abstract class ComparisonChain {
  private ComparisonChain() {}

  // 开启链式比较的方法，返回ComparisonChain的实例
  public static ComparisonChain start() {
    return ACTIVE;
  }
  
  // 匿名对象的方式实现自身定义的抽象方法
  private static final ComparisonChain ACTIVE =
      new ComparisonChain() {
      
        // Compare子类之间的比较
        @SuppressWarnings("unchecked")
        @Override
        public ComparisonChain compare(Comparable left, Comparable right) {
          return classify(left.compareTo(right));
        }

        // Comparator子类之间的比较
        @Override
        public <T> ComparisonChain compare(
            @Nullable T left, @Nullable T right, Comparator<T> comparator) {
          return classify(comparator.compare(left, right));
        }

        // int类型之间的比较
        @Override
        public ComparisonChain compare(int left, int right) {
          return classify(Ints.compare(left, right));
        }

        // long类型之间的比较
        @Override
        public ComparisonChain compare(long left, long right) {
          return classify(Longs.compare(left, right));
        }

        // float类型之间的比较
        @Override
        public ComparisonChain compare(float left, float right) {
          return classify(Float.compare(left, right));
        }

        // double类型之间的比较
        @Override
        public ComparisonChain compare(double left, double right) {
          return classify(Double.compare(left, right));
        }

        // boolean类型之间的比较
        @Override
        public ComparisonChain compareTrueFirst(boolean left, boolean right) {
          return classify(Booleans.compare(right, left)); // reversed
        }

        // boolean类型之间的比较
        @Override
        public ComparisonChain compareFalseFirst(boolean left, boolean right) {
          return classify(Booleans.compare(left, right));
        }
        
        // 返回最终比较结果，还是ComparisonChain实例
        ComparisonChain classify(int result) {
          return (result < 0) ? LESS : (result > 0) ? GREATER : ACTIVE;
        }

        @Override
        public int result() {
          return 0;
        }
      };

  // 比较结果为-1的ComparisonChain实例
  private static final ComparisonChain LESS = new InactiveComparisonChain(-1);

  // 比较结果为1的ComparisonChain实例
  private static final ComparisonChain GREATER = new InactiveComparisonChain(1);

  private static final class InactiveComparisonChain extends ComparisonChain {

    final int result;

    InactiveComparisonChain(int result) {
      this.result = result;
    }

    @Override
    public ComparisonChain compare(@Nullable Comparable left, @Nullable Comparable right) {
      return this;
    }

    @Override
    public <T> ComparisonChain compare(
        @Nullable T left, @Nullable T right, @Nullable Comparator<T> comparator) {
      return this;
    }

    @Override
    public ComparisonChain compare(int left, int right) {
      return this;
    }

    @Override
    public ComparisonChain compare(long left, long right) {
      return this;
    }

    @Override
    public ComparisonChain compare(float left, float right) {
      return this;
    }

    @Override
    public ComparisonChain compare(double left, double right) {
      return this;
    }

    @Override
    public ComparisonChain compareTrueFirst(boolean left, boolean right) {
      return this;
    }

    @Override
    public ComparisonChain compareFalseFirst(boolean left, boolean right) {
      return this;
    }

    @Override
    public int result() {
      return result;
    }
  }

  // 下面的方法都是由内部类所实现
  public abstract ComparisonChain compare(Comparable<?> left, Comparable<?> right);
  
  public abstract <T> ComparisonChain compare(
      @Nullable T left, @Nullable T right, Comparator<T> comparator);

  public abstract ComparisonChain compare(int left, int right);

  public abstract ComparisonChain compare(long left, long right);

  public abstract ComparisonChain compare(float left, float right);

  public abstract ComparisonChain compare(double left, double right);

  // @Deprecated 注解表示已过时
  @Deprecated
  public final ComparisonChain compare(Boolean left, Boolean right) {
    return compareFalseFirst(left, right);
  }

  public abstract ComparisonChain compareTrueFirst(boolean left, boolean right);

  public abstract ComparisonChain compareFalseFirst(boolean left, boolean right);

  public abstract int result();
}
``` 




