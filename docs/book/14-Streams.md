[TOC]

<!-- Streams -->
# 第十四章 流式编程

Collections 优化了对象的存储。Streams 是和对象组的处理有关。流是一系列与任何特定存储机制无关的元素 — 实际上我们说流没有“存储”。

不需要迭代集合中的元素，而是在管道中绘制元素并对其操作的流。这些管道经常被组合在一起，在流上形成一个操作管道。

在大多数情况下，将对象存储在集合中的原因是为了处理他们，因此你将会发现你将把编程的主要焦点从集合转移到了流上。流的一个核心好处是，它使得程序更加短小并且更易理解。当 Lambda 表达式和方法引用（method references）和流一起使用的时候感觉自成一体。流使得 Java 8 更巨吸引力。

例如，你想展现在 5 到 20 之间随机选择的序列中只出现一次的数字，并且是排序好的。事实上，你对他们进行排序可能使得你的精力首先集中在选择一个已排序的集合。但是对于流，你只需要简单的说明你想要什么：

```java
// streams/Randoms.java
import java.util.*;
public class Randoms {
    public static void main(String[] args) {
        new Random(47)
            .ints(5, 20)
            .distinct()
            .limit(7)
            .sorted()
            .forEach(System.out::println);
    }
}
```

输出为：

```java
6
10
13
16
17
18
19
```

首先，我们给 **Random** 对象一个种子（以便程序再次运行时产生相同的输出）。**ints()** 方法产生一个流并且 **ints()** 方法有多种方式的重载 — 两个参数限定了数值产生的边界。这将生成一个整数流。我们告诉他使用中间流操作（intermediate stream operation） **distinct()** 来获取它们的唯一值，然后使用 **limit()** 方法获取前 7 个元素。接下来，我们使用 **sorted()** 方法希望元素是有序的。最终，我们希望显示每个条目，因此使用 **forEach()**，它根据传递给它的函数对每个流对象执行操作。在这里，我们传递了一个可以在控制台展现每个元素的方法引用 **System.out::println** 。

注意 **Randoms.java** 中没有声明任何变量。流可以对具有状态的系统建模，并且不需要使用赋值或者可变数据，这非常有用。

声明式编程是一种风格，在这种风格中，我们声明我们想要做什么而不是指定如何去做，这就是你在函数式编程中所看到的。注意，理解命令式编程的形式要困难的多：

```java
// streams/ImperativeRandoms.java
import java.util.*;
public class ImperativeRandoms {
    public static void main(String[] args) {
        Random rand = new Random(47);
        SortedSet<Integer> rints = new TreeSet<>();
        while(rints.size() < 7) {
            int r = rand.nextint(20);
            if(r < 5) continue;
            rints.add(r);
        }
        System.out.println(rints);
    }
}
```

输出为：

```java
[7, 8, 9, 11, 13, 15, 18]
```

在 **Randoms.java** 中，我们无需定义任何变量，但是在这里我们定义了 3 个变量： **rand**，**rints** 和 **r**。这个代码变的更加复杂，是由于 **nextInt()** 没有下界选项 — 其内置的下界永远为 0，因此我们生成额外的数值并过滤小于 5 的值。

注意，你必须研究代码来弄清楚发生了什么，而在 **Randoms.java** 中，代码只是告诉了你它在做什么。这种清晰度是 Java 8 中流使人最信服的原因之一。

在 **ImperativeRandoms.java** 中显式的编写迭代机制称之为外部迭代。在 **Randoms.java** 中，你没有看到这些机制，你并没有看到这样类似的机制，它是流编程中的核心特征被称之为内部迭代。内部迭代产生更可读的代码，也更容易使用多个处理器。通过放松对迭代发生的控制，你可以将控制权交给并行化机制。你将在[并发编程]()这一章了解这一点。

流另一个重要方面是他们是惰性（lazy）的，意味着它们只在绝对必要时进行评估。你可以将流看作“延迟列表”。由于评估延迟，流可以使我们表示非常大（甚至无限）的序列，并且没有内存担忧。

<!-- Java 8 Stream Support -->

## 流支持

Java 设计者面临着这样一个难题：现存的大量类库不仅为 Java 所用，同时也被应用在整个 Java 生态圈数百万行的代码中。如何将一个全新的流的概念融入到现有类库中呢？

简单的例子,如在 **Random** 中添加更多的方法。因为只要不改变原有的方法，遗留代码就不会受到干扰。

问题是，接口部分怎么改造呢？特别是涉及集合类接口的部分。如果你想把一个集合转换为流，直接向接口添加新方法会破坏所有老的接口实现类。

Java 8 采用的解决方案是：在[接口](10-Interfaces.md)中添加被 **default**（**默认**）修饰的方法。通过这种方案，设计者们可以将流式（*stream*）方法平滑地嵌入到现有类中。流方法预置的操作几乎已满足了我们平常所有的需求。流操作的类型有三种：创建流，修改流元素（中间操作, *Intermediate Operations*），消费流元素（终端操作, *Terminal Operations*）。最后一种类型通常意味着收集流元素（通常是到集合中）。

下面我们来看下每种类型的流操作。

<!-- Stream Creation -->
## 流创建

你可以通过 **Stream.of()** 很容易的将一组元素转化成为流（**Bubble** 类在之前的章节中已经定义过了）：

```java
// streams/StreamOf.java
import java.util.stream.*;
public class StreamOf {
    public static void main(String[] args) {
        Stream.of(new Bubble(1), new Bubble(2), new Bubble(3))
            .forEach(System.out::println);
        Stream.of("It's ", "a ", "wonderful ", "day ", "for ", "pie!")
            .forEach(System.out::print);
        System.out.println();
        Stream.of(3.14159, 2.718, 1.618)
            .forEach(System.out::println);
    }
}
```

输出为：

```java
Bubble(1)
Bubble(2)
Bubble(3)
It's a wonderful day for pie!
3.14159
2.718
1.618
```

除此之外，每个 **Collection** 都可以通过 **stream()** 方法来产生一个流：

```java
// streams/CollectionToStream.java
import java.util.*;
import java.util.stream.*;
public class CollectionToStream {
    public static void main(String[] args) {
        List<Bubble> bubbles = Arrays.asList(new Bubble(1), new Bubble(2), new Bubble(3));
        System.out.println(bubbles.stream()
            .mapToint(b -> b.i)
            .sum());
        
        Set<String> w = new HashSet<>(Arrays.asList("It's a wonderful day for pie!".split(" ")));
        w.stream()
         .map(x -> x + " ")
         .forEach(System.out::print);
        System.out.println();
        
        Map<String, double> m = new HashMap<>();
        m.put("pi", 3.14159);
        m.put("e", 2.718);
        m.put("phi", 1.618);
        m.entrySet().stream()
                    .map(e -> e.getKey() + ": " + e.getValue())
                    .forEach(System.out::println);
    }
}
```

输出为：

```java
6
a pie! It's for wonderful day
phi: 1.618
e: 2.718
pi: 3.14159
```

在创建 **List\<Bubble\>** 对象之后，我们只需要简单的调用所有集合中都有的方法 **stream()**。中间操作 **map()** 会获取流中的所有元素，并且对流中元素应用操作从而产生新的元素，并将其传递到流中。通常情况 **map()** 方法获取对象并产生新的对象，但是这里有特殊版本的方法用于数值类型的流。例如，**mapToInt()** 方法将一个对象流（objects stream）转换成为包含整形数字的 **IntStream**。同样有针对 **Float** 和 **Double** 的类似名字的操作。

我们通过在 **String** 类型上面应用 **split()** - split 方法会根据参数来拆分字符串 - 获取元素用于定义 **w**。稍后你会看到这个参数十分复杂，但是在这里我们只是根据空格来分割字符串。

为了从 **Map** 集合中产生流数据，我们首先调用 **entrySet()** 去产生一个对象流，每个对象都包含一个键以及与其相关联的值。然后调用 **getKey()** 和 **getValue()** 将其分开。

### 随机数流

**Random** 类被一组生成流的方法增强了：

```java
// streams/RandomGenerators.java
import java.util.*;
import java.util.stream.*;
public class RandomGenerators {
    public static <T> void show(Stream<T> stream) {
        stream
        .limit(4)
        .forEach(System.out::println);
        System.out.println("++++++++");
    }
    
    public static void main(String[] args) {
        Random rand = new Random(47);
        show(rand.ints().boxed());
        show(rand.longs().boxed());
        show(rand.doubles().boxed());
        // Control the lower and upper bounds:
        show(rand.ints(10, 20).boxed());
        show(rand.longs(50, 100).boxed());
        show(rand.doubles(20, 30).boxed());
        // Control the stream size:
        show(rand.ints(2).boxed());
        show(rand.longs(2).boxed());
        show(rand.doubles(2).boxed());
        // Control the stream size and bounds:
        show(rand.ints(3, 3, 9).boxed());
        show(rand.longs(3, 12, 22).boxed());
        show(rand.doubles(3, 11.5, 12.3).boxed());
    }
}
```

输出为：

```java
-1172028779
1717241110
-2014573909
229403722
++++++++
2955289354441303771
3476817843704654257
-8917117694134521474
4941259272818818752
++++++++
0.2613610344283964
0.0508673570556899
0.8037155449603999
0.7620665811558285
++++++++
16
10
11
12
++++++++
65
99
54
58
++++++++
29.86777681078574
24.83968447804611
20.09247112332014
24.046793846338723
++++++++
1169976606
1947946283
++++++++
2970202997824602425
-2325326920272830366
++++++++
0.7024254510631527
0.6648552384607359
++++++++
6
7
7
++++++++
17
12
20
++++++++
12.27872414236691
11.732085449736195
12.196509449817267
++++++++
```

为了消除冗余代码，我创建了一个泛型方法 **show(Stream\<T\> stream)** （在讲解泛型之前就使用这个特性，确实有点作弊，但是回报是值得的）。类型参数 **T** 可以是任何类型，所以这个方法对 **Integer**， **Long** 和 **Double** 类型都生效。但是 **Random** 类只能生成原始数据类型 **int**， **long**， **double** 的流。幸运的是， **boxed()** 流操作将会自动的把基本类型包装成为对应的装箱类型，从而使得 **show()** 能够接受流。

我们可以使用 **Random** 为任意对象集合创建 **Supplier**。如下是一个从文本文件提供 **String** 对象的例子：

```java
// streams/Cheese.dat
Not much of a cheese shop really, is it?
Finest in the district, sir.
And what leads you to that conclusion?
Well, it's so clean.
It's certainly uncontaminated by cheese.
We use the Files class to read all the lines from a file into a
List<String> :
```

```java
// streams/RandomWords.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
import java.io.*;
import java.nio.file.*;
public class RandomWords implements Supplier<String> {
    List<String> words = new ArrayList<>();
    Random rand = new Random(47);
    RandomWords(String fname) throws IOException {
        List<String> lines = Files.readAllLines(Paths.get(fname));
        // Skip the first line:
        for (String line : lines.subList(1, lines.size())) {
            for (String word : line.split("[ .?,]+"))
                words.add(word.toLowerCase());
        }
    }
    public String get() {
        return words.get(rand.nextint(words.size()));
    }
    @Override
    public String toString() {
        return words.stream()
            .collect(Collectors.joining(" "));
    }
    public static void main(String[] args) throws Exception {
        System.out.println(
            Stream.generate(new RandomWords("Cheese.dat"))
                .limit(10)
                .collect(Collectors.joining(" ")));
    }
}
```

输出为：

```java
it shop sir the much cheese by conclusion district is
```

在这里你可以看到更为复杂的 **split()** 的使用。在构造器中，每一行都被 **split()** 方法通过空格或者被方括号包裹的任意标点符号进行分割。在结束方括号后面的 **+** 代表「+ 前面的东西可以出现一次或者多次」。

你将注意到在构造函数中循环体使用命令式编程（外部迭代）。在以后的例子中，你将会看到我门如何消除这一点。这种旧的形式不是特别糟糕，但是到处使用流会让你觉得更好一些。

在 **toString()** 和 **main()** 中你看到了 **collect()** 收集操作，它根据参数来组合所有流中的元素。

当你使用 **Collectors.joining()**，你将会得到一个 **String** 类型的结果，每个元素都根据 **joining()** 的参数来进行分割。还有许多不同的 **Collectors** 用于获取不同的结果。

在 **main()** 中，我们看到了 **Stream.generate()** 的预览版本，它可以把任意  **Supplier\<T\>** 用于生成 **T** 类型的流。


### int 类型的范围（Ranges of int）

**IntStream** 类提供了  **range()** 方法用于生成整数序列的流。编写循环时，这个方法会更加便利：

```java
// streams/Ranges.java
import static java.util.stream.IntStream.*;
public class Ranges {
    public static void main(String[] args) {
        // The traditional way:
        int result = 0;
        for (int i = 10; i < 20; i++)
            result += i;
        System.out.println(result);
        // for-in with a range:
        result = 0;
        for (int i : range(10, 20).toArray())
            result += i;
        System.out.println(result);
        // Use streams:
        System.out.println(range(10, 20).sum());
    }
}
```

输出为：

```java
145
145
145
```

在 **main()** 方法中的第一种方式是我们传统编写 **for** 循环的方式。在第二种方法，我们使用 **range()** 创建了流并将其转化为数组，然后在 **for-in** 代码块中使用。但是，如果你能够像第三种方法全程使用流是很好的。在每种情况下，我们对范围中的数字进行求和，并且流中可以很方便的使用 **sum()** 操作求和。

注意 **IntStream.range()** 相比 **onjava.Range.range()** 拥有更多的限制。这是由于其可选的第三个参数，后者能够生成步长大于 1 的范围，并且可以从大到小来生成。

为了替换简单的 **for** 循环，这里是一个 **repeat()** 实用程序：

```java
// onjava/Repeat.java
package onjava;
import static java.util.stream.IntStream.*;
public class Repeat {
    public static void repeat(int n, Runnable action) {
        range(0, n).forEach(i -> action.run());
    }
}
```

其产生的循环更加清晰：

```java
// streams/Looping.java
import static onjava.Repeat.*;
public class Looping {
    static void hi() {
        System.out.println("Hi!");
    }
    public static void main(String[] args) {
        repeat(3, () -> System.out.println("Looping!"));
        repeat(2, Looping::hi);
    }
}
```

输出为：

```java
Looping!
Looping!
Looping!
Hi!
Hi!
```

在代码中包含并解释 **repeat()** 似乎有些不值得。它似乎是一个相当透明的工具，但它取决于你的团队和公司的运作方式

### generate()

**RandomWords.java** 在 **Stream.generate()** 中使用 **Supplier\<T\>**。这里是第二个示例：

```java
// streams/Generator.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class Generator implements Supplier<String> {
    Random rand = new Random(47);
    char[] letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();
    
    public String get() {
        return "" + letters[rand.nextInt(letters.length)];
    }
    
    public static void main(String[] args) {
        String word = Stream.generate(new Generator())
                            .limit(30)
                            .collect(Collectors.joining());
        System.out.println(word);
    }
}
```

输出为：

```java
YNZBRNYGCFOWZNTCQRGSEGZMMJMROE
```

使用 **Random.nextInt()** 方法来挑选字母表中的大写字母。**Random.nextInt()** 的参数代表可以接受的最大的随机数范围，所以使用数组边界是经过深思熟虑的。

如果要创建包含相同对象的流，只需要传递一个生成那些对象 **lambda** 到 **generate()** 中：

```java
// streams/Duplicator.java
import java.util.stream.*;
public class Duplicator {
    public static void main(String[] args) {
        Stream.generate(() -> "duplicate")
              .limit(3)
              .forEach(System.out::println);
    }
}
```

输出为：

```java
duplicate
duplicate
duplicate
```

如下是在这个章节中之前例子使用过的 **Bubble** 类。注意它包含了自己的静态生成器（*static generator*）方法。

```java
// streams/Bubble.java
import java.util.function.*;
public class Bubble {
    public final int i;
    
    public Bubble(int n) {
        i = n;
    }
    
    @Override
    public String toString() {
        return "Bubble(" + i + ")";
    }
    
    private static int count = 0;
    public static Bubble bubbler() {
        return new Bubble(count++);
    }
}
```

由于 **bubbler()** 与 **Supplier\<Bubble\>** 是接口兼容的，我们可以将其方法引用直接传递给 **Stream.generate()**：

```java
// streams/Bubbles.java
import java.util.stream.*;
public class Bubbles {
    public static void main(String[] args) {
        Stream.generate(Bubble::bubbler)
              .limit(5)
              .forEach(System.out::println);
    }
}
```

输出为：

```java
Bubble(0)
Bubble(1)
Bubble(2)
Bubble(3)
Bubble(4)
```

这是创建单独工厂类（ separate factory class）的另外一种方式。在很多方面它更加整洁，但是这代表着品味和代码组织的问题 - 你总是可以创建一个完全不同的工厂类。

### iterate()

**Stream.iterate()** 以种子（第一个参数）开头，并将其传给方法（第二个参数）。方法的结果将添加到流，并存储作为第一个参数用于下次调用 **iterate()**，依次类推。我们可以使用 **iterate()** 用于生成一个 Fibonacci 序列（你在上一章中遇到）：

```java
// streams/Fibonacci.java
import java.util.stream.*;
public class Fibonacci {
    int x = 1;
    
    Stream<Integer> numbers() {
        return Stream.iterate(0, i -> {
            int result = x + i;
            x = i;
            return result;
        });
    }
    
    public static void main(String[] args) {
        new Fibonacci().numbers()
                       .skip(20) // Don't use the first 20
                       .limit(10) // Then take 10 of them
                       .forEach(System.out::println);
    }
}
```

输出为：

```java
6765
10946
17711
28657
46368
75025
121393
196418
317811
514229
```

Fibonacci 序列将序列中最后两个元素进行求和以产生下一个元素。**iterate()** 只能记忆结果，因此我们需要使用一个变量 **x** 来用于追踪另外一个元素。

在 **main()** 中，我们使用了一个你之前没有见过的 **skip() ** 操作。它只是根据它的参数丢弃指定数量的流元素。在这里，我们丢弃了前 20 个元素。

### Stream Builders

在建造者设计模式中，首先创建一个 builder 对象，传递给它多个构造器信息，最后执行“构造”。**Stream** 库提供了这样的 **Builder**。在这里，我们重新审视读取文件并将其转换成为单词流的过程：

```java
// streams/FileToWordsBuilder.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;

public class FileToWordsBuilder {
    Stream.Builder<String> builder = Stream.builder();
    
    public FileToWordsBuilder(String filePath) throws Exception {
        Files.lines(Paths.get(filePath))
             .skip(1) // Skip the comment line at the beginning
              .forEach(line -> {
                  for (String w : line.split("[ .?,]+"))
                      builder.add(w);
              });
    }
    
    Stream<String> stream() {
        return builder.build();
    }
    
    public static void main(String[] args) throws Exception {
        new FileToWordsBuilder("Cheese.dat")
            .stream()
            .limit(7)
            .map(w -> w + " ")
            .forEach(System.out::print);
    }
}
```

输出为：

```java
Not much of a cheese shop really
```

注意，构造器会添加文件中的所有单词（除了第一行，它是包含文件路径信息的注释），但是其并没有调用 **build()** 方法。这意味着，只要你不调用 **stream()** 方法，就可以继续向 **builder** 对象中添加单词。

在此类的更完整的版本中，你可以添加一个标志位用于查看 **build()** 方法是否被调用，并且可能的话增加一个可以添加更多单词的方法。在 **Stream.Builder** 调用 **build()** 方法后继续尝试添加单词会产生一个异常。

### Arrays

**Arrays** 类中含有一个名为 **stream()** 的静态方法用于把数组转换成为流。我们可以重写 **interfaces/Machine.java** 中的 **main()** 方法用于创建一个流，并将 **execute()** 应用于每一个元素：

```java
// streams/Machine2.java
import java.util.*;
import onjava.Operations;
public class Machine2 {
    public static void main(String[] args) {
        Arrays.stream(new Operations[] {
            () -> Operations.show("Bing"),
            () -> Operations.show("Crack"),
            () -> Operations.show("Twist"),
            () -> Operations.show("Pop")
        }).forEach(Operations::execute);
    }
}
```

输出为：

```java
Bing
Crack
Twist
Pop
```

**new Operations[]** 表达式动态创建了 **Operations** 对象的数组。

**stream()** 方法同样可以产生 **IntStream**，**LongStream** 和 **DoubleStream**。

```java
// streams/ArrayStreams.java
import java.util.*;
import java.util.stream.*;

public class ArrayStreams {
    public static void main(String[] args) {
        Arrays.stream(new double[] { 3.14159, 2.718, 1.618 })
            .forEach(n -> System.out.format("%f ", n));
        System.out.println();
        
        Arrays.stream(new int[] { 1, 3, 5 })
            .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        
        Arrays.stream(new long[] { 11, 22, 44, 66 })
            .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        
        // Select a subrange:
        Arrays.stream(new int[] { 1, 3, 5, 7, 15, 28, 37 }, 3, 6)
            .forEach(n -> System.out.format("%d ", n));
    }
}
```

输出为：

```java
3.141590 2.718000 1.618000
1 3 5
11 22 44 66
7 15 28
```

最后一次 **stream()** 的调用有两个额外的参数。第一个参数告诉 **stream()** 从哪里开始在数组中选择元素，第二个参数用于告知在哪里停止。每种不同类型的 **stream()** 方法都有这个版本。

### 正则表达式（Regular Expressions）

Java 的正则表达式已经在[字符串]()这一章节介绍过了。Java 8 在 **java.util.regex.Pattern** 中增加了一个新的方法 `splitAsStream()`，这个方法可以根据你所传入的公式将字符序列转化为流。但是这里有一个限制，输入只能是 **CharSequence**，因此不能将流作为 `splitAsStream()` 的参数。

我们再一次查看将文件处理为单词流的过程。这一次，我们使用流将文件分割为单独的字符串，接着使用正则表达式将字符串转化为单词流。

```java
// streams/FileToWordsRegexp.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;
import java.util.regex.Pattern;
public class FileToWordsRegexp {
    private String all;
    public FileToWordsRegexp(String filePath) throws Exception {
        all = Files.lines(Paths.get(filePath))
        .skip(1) // First (comment) line
        .collect(Collectors.joining(" "));
    }
    public Stream<String> stream() {
        return Pattern
        .compile("[ .,?]+").splitAsStream(all);
    }
    public static void
    main(String[] args) throws Exception {
        FileToWordsRegexp fw = new FileToWordsRegexp("Cheese.dat");
        fw.stream()
          .limit(7)
          .map(w -> w + " ")
          .forEach(System.out::print);
        fw.stream()
          .skip(7)
          .limit(2)
          .map(w -> w + " ")
          .forEach(System.out::print);
    }
}
```

输出为：

```java
Not much of a cheese shop really is it
```

在构造器中我们读取了文件中的所有内容（再一次跳过了第一行注释），并将其转化成为单行字符串。现在，当你调用 `stream()` 方法的时候，可以像往常一样获取一个流，但这次你可以多次调用 `stream()` 方法，在已存储的字符串中创建一个新的流。这里有个限制，整个文件必须存储在内存中；在大多数情况下这并不是什么问题，但是这损失了流中非常重要的好处：

1. 流“不需要存储”。当然它们需要一些内部存储，但是这只是序列的一小部分，和持有整个序列所需要的并不相同。
2. 它们是惰性评估的。幸运的是，我们将在稍晚一些的时候查看如何如解决这个问题。

<!-- Intermediate Operations -->

## 中间操作


<!-- Optional -->
## Optional类

在我们查看终端操作之前，我们必须考虑如果你在一个空流中获取元素会发生什么。我们喜欢为了“happy path”而将流连接起来，并假设为空会被中断。在流中放置 **null** 是很好的中断方法。我们可以使用哪种对象作为流元素的持有者，如果我们寻找的元素并不存在也可以友好的告诉我们（也就是说，没有异常）？

这个想法是通过 **Optional** 实现的。确保标准流操作返回 **Optional** 对象，因为它们并不能保证预期结果一定存在。它们包括：

- `findFirst()` 返回一个包含第一个元素的 **Optional** 对象，如果流为空则返回 **Optional.empty**
- `findAny()` 返回包含任意元素的 **Optional** 对象，如果流为空则返回 **Optional.empty**
- `max` 和 `min()` 返回一个包含最大值或者最小值的 **Optional** 对象，如果流为空则返回 **Optional.empty**

 不再以 “identity”对象开头版本的 `reduce()`将其返回值包装在 **Optional** 中。（“identity”对象成为另一个版本的 `reduce()` 的默认结果，因此不存在空结果的风险）

对于数字流 **IntStream**、**LongStream** 和 **DoubleStream**，`average()` 会将结果包装在 **Optional** 以防止流为空。

以下是对空流进行的所有这些操作的简单测试：

```java
// streams/OptionalsFromEmptyStreams.java
import java.util.*;
import java.util.stream.*;
class OptionalsFromEmptyStreams {
    public static void main(String[] args) {
        System.out.println(Stream.<String>empty()
             .findFirst());
        System.out.println(Stream.<String>empty()
             .findAny());
        System.out.println(Stream.<String>empty()
             .max(String.CASE_INSENSITIVE_ORDER));
        System.out.println(Stream.<String>empty()
             .min(String.CASE_INSENSITIVE_ORDER));
        System.out.println(Stream.<String>empty()
             .reduce((s1, s2) -> s1 + s2));
        System.out.println(IntStream.empty()
             .average());
    }
}
```

输出为：

```java
Optional.empty
Optional.empty
Optional.empty
Optional.empty
Optional.empty
OptionalDouble.empty
```

当流为空的时候你会获得一个 **Optional.empty** 对象，而不是抛出异常。**Optional** 拥有 `toString()` 方法可以用于展示有用信息。

注意，空流是通过 `Stream.<String>empty()` 创建的。如果你在没有任何上下文环境的情况下调用 `Stream.empty()`，Java 并不知道它的数据类型；这个语法解决了这个问题。如果编译器拥有了足够的上下文信息，比如：

```java
Stream<String> s = Stream.empty();
```

就可以在调用 `empty()` 时推断类型。

这个示例展现了 **Optional** 的两个基本用法：

```java
// streams/OptionalBasics.java
import java.util.*;
import java.util.stream.*;
class OptionalBasics {
    static void test(Optional<String> optString) {
        if(optString.isPresent())
        System.out.println(optString.get()); else
        System.out.println("Nothing inside!");
    }
    public static void main(String[] args) {
        test(Stream.of("Epithets").findFirst());
        test(Stream.<String>empty().findFirst());
    }
}
```

输出为：

```java
Epithets
Nothing inside!
```

当你接收到 **Optional** 对象时，你首先调用 `isPresent()` 检查其中是否包含元素。如果存在，你可以使用 `get()` 获取。

### 便利函数（Convenience Functions）

解包 **Optional** 有许多便利函数，简化了“检查并对所包含的对象执行某些操作”的上述过程：

- `ifPresent(Consumer)`：当值存在时调用 **Consumer**，否则什么不做。
- `orElse(otherObject)`：如果值存在则直接返回，否则生成 **otherObject**。
- `orElseGet(Supplier)`：如果值存在直接生成对象，否则使用 **Supplier** 函数生成一个可替代对象。
- `orElseThrow(Supplier)`：如果值存在直接生成对象，否则使用 **Supplier** 函数生成一个异常。

如下是针对不同便利函数的简单演示：

```java
// streams/Optionals.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
public class Optionals {
    static void basics(Optional<String> optString) {
        if(optString.isPresent())
            System.out.println(optString.get()); 
        else
            System.out.println("Nothing inside!");
    }
    static void ifPresent(Optional<String> optString) {
        optString.ifPresent(System.out::println);
    }
    static void orElse(Optional<String> optString) {
        System.out.println(optString.orElse("Nada"));
    }
    static void orElseGet(Optional<String> optString) {
        System.out.println(
        optString.orElseGet(() -> "Generated"));
    }
    static void orElseThrow(Optional<String> optString) {
        try {
            System.out.println(optString.orElseThrow(
            () -> new Exception("Supplied")));
        }
        catch(Exception e) {
            System.out.println("Caught " + e);
        }
    }
    static void test(String testName, Consumer<Optional<String>> cos) {
        System.out.println(" === " + testName + " === ");
        cos.accept(Stream.of("Epithets").findFirst());
        cos.accept(Stream.<String>empty().findFirst());
    }
    public static void main(String[] args) {
        test("basics", Optionals::basics);
        test("ifPresent", Optionals::ifPresent);
        test("orElse", Optionals::orElse);
        test("orElseGet", Optionals::orElseGet);
        test("orElseThrow", Optionals::orElseThrow);
    }
}
```

输出为：

```java
=== basics ===
Epithets
Nothing inside!
=== ifPresent ===
Epithets
=== orElse ===
Epithets
Nada
=== orElseGet ===
Epithets
Generated
=== orElseThrow ===
Epithets
Caught java.lang.Exception: Supplied
```

` test() ` 方法通过使用与所有示例方法匹配的 **Consumer** 来防止代码重复。`orElseThrow()` 使用 **catch** 关键字来捕获 `orElseThrow()` 抛出的异常。你想会在 [异常]() 这一章节中学习细节。

### 创建 Optional（Creating Optionals）

当你编写自己的代码生成 **Optional** 时，这里有 3 个你可以使用的静态方法：

- `empty()`：生成一个内部没有任何东西的 **Optional**。
- `of(value)`：如果你已经确定值不为空，使用这个方法将值包装成 **Optional**。
- `ofNullable(value)`：如果你不知道值知否为空。这个方法会在值为空的时候自动生成 **Optional.empty**，否则将值包装在 **Optional** 中。

你可以查看这是如何工作的：

```java
// streams/CreatingOptionals.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
class CreatingOptionals {
    static void test(String testName, Optional<String> opt) {
        System.out.println(" === " + testName + " === ");
        System.out.println(opt.orElse("Null"));
    }
    public static void main(String[] args) {
        test("empty", Optional.empty());
        test("of", Optional.of("Howdy"));
        try {
            test("of", Optional.of(null));
        } catch(Exception e) {
            System.out.println(e);
        }
        test("ofNullable", Optional.ofNullable("Hi"));
        test("ofNullable", Optional.ofNullable(null));
    }
}
```

输出为：

```java
=== empty ===
Null
=== of ===
Howdy
java.lang.NullPointerException
=== ofNullable ===
Hi
=== ofNullable ===
Null
```

如果我们试图将 **null** 传递 `of()` 用于创建 **Optional** 对象，这就会爆炸。`ofNullable()` 会优雅的处理 **null**，所以它似乎是最安全的。

### Optional 对象操作

3 个方法开启了 **Optional** 的后续操作，所以如果你的流管道生成了 **Optional** 对象，你可以在结尾做更多的事情：

- `filter(Predicate)`：将 **Predicate** 应用于 **Optional** 的内容，并将结果返回。如果 **Optional** 不满足 **Predicate**，则返回 **empty**。如果 **Optional** 已经为空，则将其返回。
- `map(Function)`：如果 **Optional** 不为空，则将 **Function**  应用于 **Optional** 的内容，并将结果返回。否则，直接返回 **Optional.empty**。
- `flatMap(Function)`：如同 `map()` ， 但是提供的映射函数将结果包装在 **Optional** 对象中，因此 `flatMap()` 不会在最后进行任何包装。

如上方法都不适用于数值型 **Optional**。普通流过滤器会在 **Predicate** 返回 false 时删除流元素。**Optional.filter()** 当 **Predicate** 失败时不会删除 **Optional**——it leaves it, 但将其转化为空：

```java
// streams/OptionalFilter.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
class OptionalFilter {
    static String[] elements = {
            "Foo", "", "Bar", "Baz", "Bingo"
    };
    static Stream<String> testStream() {
        return Arrays.stream(elements);
    }
    static void test(String descr, Predicate<String> pred) {
        System.out.println(" ---( " + descr + " )---");
        for(int i = 0; i <= elements.length; i++) {
            System.out.println(
                    testStream()
                            .skip(i)
                            .findFirst()
                            .filter(pred));
        }
    }
    public static void main(String[] args) {
        test("true", str -> true);
        test("false", str -> false);
        test("str != \"\"", str -> str != "");
        test("str.length() == 3", str -> str.length() == 3);
        test("startsWith(\"B\")",
                str -> str.startsWith("B"));
    }
}
```

输出为：

```java
---( true )---
Optional[Foo]
Optional[]
Optional[Bar]
Optional[Baz]
Optional[Bingo]
Optional.empty
---( false )---
Optional.empty
Optional.empty
Optional.empty
Optional.empty
Optional.empty
Optional.empty
---( str != "" )---
Optional[Foo]
Optional.empty
Optional[Bar]
Optional[Baz]
Optional[Bingo]
Optional.empty
---( str.length() == 3 )---
Optional[Foo]
Optional.empty
Optional[Bar]
Optional[Baz]
Optional.empty
Optional.empty
---( startsWith("B") )---
Optional.empty
Optional.empty
Optional[Bar]
Optional[Baz]
Optional[Bingo]
Optional.empty
```

即使输出看起来像流，但是特别注意 `test()` 中的 for 循环。它在每一次 for 循环时重新启动流，然后根据 for 循环的索引跳过指定个数的元素，这就是它在流中的每个连续元素上结束的原因。接下来调用 `findFirst()` 获取剩余元素中的第一个元素，结果会包装在 **Optional** 中。

值得注意的是，不同于普通的 for 循环。这里的索引值范围并不是 **i < elements.length**， 而是 **i <= elements.length**，所以最后一个元素实际上超越了流。方便的是，这将自动成为 **Optional.empty**，你可以在每一个测试的结尾中看到。

像 `map()`一样 ， `Optional.map()` 应用函数，但是对于 **Optional**，它仅在 **Optional** 不为空时才应用映射函数。它还将 **Optional** 的内容提取到映射函数：

```java
// streams/OptionalMap.java
import java.util.Arrays;
import java.util.function.Function;
import java.util.stream.Stream;

class OptionalMap {
    static String[] elements = {"12", "", "23", "45"};

    static Stream<String> testStream() {
        return Arrays.stream(elements);
    }

    static void test(String descr, Function<String, String> func) {
        System.out.println(" ---( " + descr + " )---");
        for (int i = 0; i <= elements.length; i++) {
            System.out.println(
                    testStream()
                            .skip(i)
                            .findFirst() // Produces an Optional
                            .map(func));
        }
    }

    public static void main(String[] args) {
        // If Optional is not empty, map() first extracts
        // the contents which it then passes
        // to the function:
        test("Add brackets", s -> "[" + s + "]");
        test("Increment", s -> {
            try {
                return Integer.parseInt(s) + 1 + "";
            } catch (NumberFormatException e) {
                return s;
            }
        });
        test("Replace", s -> s.replace("2", "9"));
        test("Take last digit", s -> s.length() > 0 ?
                s.charAt(s.length() - 1) + "" : s);
    }
    // After the function is finished, map() wraps the
    // result in an Optional before returning it:
}
```

输出为：

```java
---( Add brackets )---
Optional[[12]]
Optional[[]]
Optional[[23]]
Optional[[45]]
Optional.empty
---( Increment )---
Optional[13]
Optional[]
Optional[24]
Optional[46]
Optional.empty
---( Replace )---
Optional[19]
Optional[]
Optional[93]
Optional[45]
Optional.empty
---( Take last digit )---
Optional[2]
Optional[]
Optional[3]
Optional[5]
Optional.empty
```

映射函数的返回结果会自动包装成为 **Optional**。正如你所看到的，**Optional.empty** 会被直接跳过不使用任何映射函数。

对于 **Optional** 的 `flatMap()` 应用于已经生成 **Optional** 的映射函数，所以 `flatMap()` 不会像 `map()` 所做的那样将结果封装在 **Optional** 中：

```java
// streams/OptionalFlatMap.java
import java.util.Arrays;
import java.util.Optional;
import java.util.function.Function;
import java.util.stream.Stream;

class OptionalFlatMap {
    static String[] elements = {"12", "", "23", "45"};

    static Stream<String> testStream() {
        return Arrays.stream(elements);
    }

    static void test(String descr,
                     Function<String, Optional<String>> func) {
        System.out.println(" ---( " + descr + " )---");
        for (int i = 0; i <= elements.length; i++) {
            System.out.println(
                    testStream()
                            .skip(i)
                            .findFirst()
                            .flatMap(func));
        }
    }

    public static void main(String[] args) {
        test("Add brackets",
                s -> Optional.of("[" + s + "]"));
        test("Increment", s -> {
            try {
                return Optional.of(
                        Integer.parseInt(s) + 1 + "");
            } catch (NumberFormatException e) {
                return Optional.of(s);
            }
        });
        test("Replace",
                s -> Optional.of(s.replace("2", "9")));
        test("Take last digit",
                s -> Optional.of(s.length() > 0 ?
                        s.charAt(s.length() - 1) + ""
                        : s));
    }
}
```

输出为：

```java
---( Add brackets )---
Optional[[12]]
Optional[[]]
Optional[[23]]
Optional[[45]]
Optional.empty
 ---( Increment )---
Optional[13]
Optional[]
Optional[24]
Optional[46]
Optional.empty
 ---( Replace )---
Optional[19]
Optional[]
Optional[93]
Optional[45]
Optional.empty
 ---( Take last digit )---
Optional[2]
Optional[]
Optional[3]
Optional[5]
Optional.empty
```

如同 `map()` 一样，`flatMap()` 将解压非空 **Optional** 的内容并将其应用在映射函数。唯一的区别就是 `flatMap()` 不会把结果包装在 **Optional** 中，因为映射函数已经做了这件事情。在如上的示例中，我已经在每一个映射函数中显示的完成了包装，但是但显然 `Optional.flatMap()` 是为已经自己生成 **Optional** 的函数而设计的。

### Optional 流（Streams of Optionals）

假设你有一个可能产生 **null** 的生成器。如果你使用这个生成器来创建流，你会自然的想用  **Optional** 来包装元素。如下是它的样子：

```java
// streams/Signal.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
public class Signal {
    private final String msg;
    public Signal(String msg) { this.msg = msg; }
    public String getMsg() { return msg; }
    @Override
    public String toString() {
        return "Signal(" + msg + ")";
    }
    static Random rand = new Random(47);
    public static Signal morse() {
        switch(rand.nextInt(4)) {
            case 1: return new Signal("dot");
            case 2: return new Signal("dash");
            default: return null;
        }
    }
    public static Stream<Optional<Signal>> stream() {
        return Stream.generate(Signal::morse)
                .map(signal -> Optional.ofNullable(signal));
    }
}
```

当你想使用这个流的时候，你必须弄清楚如何解包 **Optional**：

```java
// streams/StreamOfOptionals.java
import java.util.*;
import java.util.stream.*;
public class StreamOfOptionals {
    public static void main(String[] args) {
        Signal.stream()
                .limit(10)
                .forEach(System.out::println);
        System.out.println(" ---");
        Signal.stream()
                .limit(10)
                .filter(Optional::isPresent)
                .map(Optional::get)
                .forEach(System.out::println);
    }
}
```

输出为：

```java
Optional[Signal(dash)]
Optional[Signal(dot)]
Optional[Signal(dash)]
Optional.empty
Optional.empty
Optional[Signal(dash)]
Optional.empty
Optional[Signal(dot)]
Optional[Signal(dash)]
Optional[Signal(dash)]
---
Signal(dot)
Signal(dot)
Signal(dash)
Signal(dash)
```

在这里，我们使用 `filter()` 来保留那些非空 **Optional**，然后在 `map()` 中使用 `get()` 获取元素。因为每一种情况都需要你决定“无价值”的含义，所以通常要为每个应用程序采用不同的方法。

<!-- Terminal Operations -->

## 终端操作


<!-- Summary -->
## 本章小结


<!-- 分页 -->

<div style="page-break-after: always;"></div>
