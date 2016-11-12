# FluentIterable类的使用和源码分析

标签（空格分隔）： Guava使用及源码分析

---

Iterable类型的集合类往往遍历的方式比较单一，而FluentIterable提供了多种多样的遍历集合类的方法~

## FluentIterable的使用

``` 
ArrayList<String> list = Lists.newArrayList("a","b","c","d","e");
// 将普通的集合封装成功能强大的FluentIterable
FluentIterable<String> fluentIterable = FluentIterable.from(list);

// 返回Optional类型的值：Optional.of(a)
Optional<String> first = fluentIterable.first();

// 返回Optional类型的值：Optional.of(e)
Optional<String> last = fluentIterable.last();

// 返回大小：5
int size = fluentIterable.size();

// 判断是否为空：false
boolean isempty = fluentIterable.isEmpty();

// 返回所有元素拼装的字符串：[a, b, c, d, e]
String all2str = fluentIterable.toString();

// 返回第一个元素：a
String get = fluentIterable.get(0);

// 判断fluentIterable是否包含指定元素：false
boolean iscontains = fluentIterable.contains("l");

// 可以添加多个元素，返回一个新的且不可变的fluentIterable引用：[a, b, c, d, e, x, y, z]
FluentIterable<String> finalFluentIterable = fluentIterable.append("x", "y", "z");

// 添加一个Iterable类型的对象，返回一个新的且不可变的fluentIterable：[a, b, c, d, e, g, h, i, j, k]
FluentIterable<String> finalFluentIterable2 = fluentIterable.append(Lists.newArrayList("g","h","i","j","k"));

// 返回从左往右指定个数、不可变的fluentIterable引用：[a, b, c]
FluentIterable<String> finalLimitFluentIterable = fluentIterable.limit(3);

// 返回忽略从左往右指定个数、不可变的fluentIterable引用：[d, e]
FluentIterable<String> finalSkipFluentIterable = fluentIterable.skip(3);

// 返回一个周而复始的、不可变的fluentIterable引用：[a, b, c, d, e, a, b, c, d, e, ....]
FluentIterable<String> cycleFluentIterable = fluentIterable.cycle();
Iterator<String> cycleItrt = cycleFluentIterable.iterator();
while (cycleItrt.hasNext()) { // 这里就会死循环
    System.out.println(cycleItrt.next());
}

// 复制所有元素到一个新的Collection类型对象中，不可变：set：[a, b, c, d, e]
Set<String> set = new HashSet<String>();
Collection<String> collection = fluentIterable.copyInto(set);

// 转换成相应类型的不可变数组：[a, b, c, d, e]
String[] finalArr = fluentIterable.toArray(String.class);

// 转换成相应类型的不可变List：[a, b, c, d, e]
ImmutableList<String> finalList = fluentIterable.toList();

// 转换成相应类型的不可变Set：[a, b, c, d, e]
ImmutableSet<String> finalS = fluentIterable.toSet();


``` 


