# 1、从迭代到流的操作

```java
String contents = new String(Files.readAllBytes(Paths.get("D:/Backup/桌面/ciyu.txt")), StandardCharsets.UTF_8);

List<String> words = Arrays.asList(contents.split("、"));

long count = 0;

for (String w : words){
    if (w.length()<4 && w.length()>2){
        ++count;
    }
}

System.out.println(count);

count = words.stream().filter(w -> w.length()<4).count();

System.out.println(count);
```

```java
//产生一个流，其中包含当前流中满足predicate的所有元素。
Stream<T> filter(Predicate<? super T> predicate);
//产生当前流中元素的数量。这是一个终止操作。
long count();
```

```java
//产生当前集合中所有元素的顺序流或并行流。
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}
default Stream<E> parallelStream() {
   return StreamSupport.stream(spliterator(), true);
}
```

# 2、流的创建

```java
public static <T> void show(String title, Stream<T> stream){
    final int SIZE = 10;
    List<T> firstElements = stream.limit(SIZE+1).collect(Collectors.toList());
    System.out.print(title+": ");
    for (int i = 0;i<firstElements.size();i++){
        if (i>0){
            System.out.print(",");
        }
        if (i<SIZE){
            System.out.print(firstElements.get(i));
        }else {
            System.out.print("...");
        }
    }
    System.out.println("\n");
}

public static void main(String[] args) throws IOException {

    Path path = Paths.get("D:/Backup/桌面/ciyu.txt");
    String contents = new String(Files.readAllBytes(path), StandardCharsets.UTF_8);
    Stream<String> words = Stream.of(contents.split("、"));
    show("words",words);

    /**
     * public static<T> Stream<T> of(T... values)
     * 产生一个元素为给定值的流
     */
    Stream<String> song = Stream.of("gently","down","the","stream");
    show("song",song);

    /**
     * public static<T> Stream<T> empty()
     * 产生一个不包含任何元素的流
     */
    Stream<String> silence = Stream.empty();
    show("silence",silence);

    /**
     * public static<T> Stream<T> generate(Supplier<T> s)
     * 产生一个无限流，它的值是通过反复调用函数s而构建的。
     * T get()
     * 提供一个值
     */
    Stream<String> echos = Stream.generate(()->"Echo");
    show("echos",echos);
    Stream<Double> randoms = Stream.generate(Math::random);
    show("randoms",randoms);

    /**
     * public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)
     *产生一个无线流，它的元素包含seed、在seed上调用f产生的值、在前一个元素上调产生一个无限流。
     */
    Stream<BigInteger> integers = Stream.iterate(BigInteger.ONE,n->n.add(BigInteger.ONE));
    show("integers",integers);

    /**
     * public Stream<String> splitAsStream(final CharSequence input)
     * 产生一个流，它的元素是输入中由该模式界定。
     */
    Stream<String> wordsAnotherWay = Pattern.compile("、").splitAsStream(contents);
    show("wordsAnotherWay",wordsAnotherWay);

    /**
     * public static Stream<String> lines(Path path, Charset cs) throws IOException
     * public static Stream<String> lines(Path path) throws IOException
     * 产生一个流，它的元素是指定文件中的行，该文件的字符集为UTF-8，或者为指定的字符集。
     */
    try (Stream<String> lines = Files.lines(path, StandardCharsets.UTF_8)) {

        show("lines",lines);

    }

    /**
     * public static <T> Stream<T> stream(Spliterator<T> spliterator, boolean parallel)
     * 产生一个流，它包含了由给定的可分割迭代器产生的值。
     * default Spliterator<T> spliterator()
     * 为这个Iterable产生一个可分割的迭代器。默认实现不分割也不报告尺寸。
     */
    Iterable<Path> iterable = FileSystems.getDefault().getRootDirectories();
    Stream<Path> rootDirectories = StreamSupport.stream(iterable.spliterator(),false);
    show("rootDirectories",rootDirectories);

    /**
     * public static <T> Spliterator<T> spliteratorUnknownSize(Iterator<? extends T> iterator,int characteristics)
     * 用给定特性（一种包含诸如Spliterator.ORDERED之类的常量的位模式）将一个迭代器转换为一个具备未知尺寸的可分割的迭代器
     */
    Iterator<Path> iterator = Paths.get("D:/Backup/桌面").iterator();
    Stream<Path> pathComponents = StreamSupport.stream(Spliterators.spliteratorUnknownSize(iterator, Spliterator.ORDERED),false);
    show("pathComponents",pathComponents);

}
```

```java
static <T> Stream<T> ofNullable(T t)
如果t为null，返回一个空流，否则返回包含t的流。
从以下版本开始：
9
```

```java
public Stream<String> tokens()
产生一个字符串，该字符串是调用这个扫描器的next方法时返回的。
从以下版本开始：
9
```

# 3、filter、map和flatMap方法

```java
Stream<T> filter(Predicate<? super T> predicate)
产生一个流，它包含当前流中所有满足谓词条件的元素。
```

```java
<R> Stream<R> map(Function<? super T,? extends R> mapper)
产生一个流，它包含将mapper应用于当前流中所有元素所产生的结果。
```

```java
<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)
产生一个流，它是通过将mapper应用于当前流中所有中所有元素所产生的结果连接到一起而获得的。
```

# 4、抽取子流和组合流

```java
Stream<T> limit(long maxSize)
产生一个流，其中包含了当前流中最初的maxSize个元素。
```

```java
Stream<T> skip(long n)
产生一个流，它的元素是当前流中除了前n个元素之外的所有元素
```

```java
default Stream<T> takeWhile(Predicate<? super T> predicate)
产生一个流，它的元素是当前流中所有满足谓词条件的元素。
```

```java
default Stream<T> dropWhile(Predicate<? super T> predicate)
产生一个流，它的元素是当前流中排除不满足谓词条件的元素之外的所有元素。
```

```java
static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
产生一个流，它的元素是a元素后面跟着b的元素。
```

# 5、其它的流转换

```java
Stream<T> distinct()
产生一个流，包含当前流中所有不同的元素。
```

```java
Stream<T> sorted()
Stream<T> sorted(Comparator<? super T> comparator)
产生一个流，它的元素是当前流中的所有元素按照顺序排列的。第一个方法要求元素是实现了Comparable的类实例。
```

```java
Stream<T> peek(Consumer<? super T> action)
产生一个流，它与当前流中的元素相同，在获取其中每个元素时，会将其传递给action。
```

# 6、简单约简

```java
Optional<T> max(Comparator<? super T> comparator)
Optional<T> min(Comparator<? super T> comparator)
分别产生这个流的最大元素和最小元素，使用由给定比较器定义的排序规则，如果这个流为空，会产生一个空的Optional对象。这些操作都是终结操作。
```

```java
Optional<T> findFirst()
Optional<T> findAny()
分别产生这个流的第一个和任意一个元素，如果这个流为空，会产生一个空的Optional对象。这些操作都是终结操作。
```

```java
boolean anyMatch(Predicate<? super T> predicate)
boolean allMatch(Predicate<? super T> predicate)
boolean noneMatch(Predicate<? super T> predicate)
分别在这个流中任意元素，所有元素和没有任何元素匹配给定谓词时返回true。这些操作都是终结操作。
```

# 7、Optional类型

## 7.1、获取Optional值

```java
public T orElse(T other)
产生这个Optional的值，或者在该Optional为空时，产生other
```

```java
public T orElseGet(Supplier<? extends T> supplier)
产生这个Optional的值，或者在该Optional为空时，产生调用supplier的结果
```

```java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X extends Throwable
产生这个Optional的值，或者在该Optional为空时，抛出调用exceptionSupplier的结果
```

## 7.2、消费Optional值

```java
public void ifPresent(Consumer<? super T> action)
如果该Optional不为空，就将它的值传递给action。
```

```java
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)
如果该Optional不为空，就将它的值传递给action，否者调用emptyAction。
```

## 7.3、管道化Optional值

```java
public <U> Optional<U> map(Function<? super T,? extends U> mapper)
产生一个Optional，如果当前的Optional的值存在，那么所产生的Optional的值是通过将给定的函数应用于当前的Optional的值而得到的；否则，产生一个空的Optional。
```

```java
public Optional<T> filter(Predicate<? super T> predicate)
产生一个Optional，如果当前的Optional的值满足给定的谓词条件，那么所产生的Optional的值就是当前Optional的值；否则，产生一个空的Optional。
```

```java
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)
如果当前Optional不为空，则产生当前的Optional；否则由supplier产生一个Optional。
```

## 7.4、不适合使用Optional值的方式

下面是一些有关Optional类型正确用法的提示：

- Optional类型的变量永远都不应该为null。
- 不要使用Optional类型的域。因为其代价是额外多出一个对象。在类的内部，使用null表示缺失的域更易于操作。
- 不要在集合中放置Optional对象，并且不要将它们用作map的键。应该直接收集其中的值。

```java
public T get()
```

```java
public T orElseThrow()
产生这个Optional的值，或者在该Optional为空时，抛出一个NoSuchElementException异常。
从以下版本开始：
10
```

```java
public boolean isPresent()
如果该Optional不为空，则返回true
```

## 7.5、创建Optional值

```java
public static <T> Optional<T> of(T value)
public static <T> Optional<T> ofNullable(T value)
产生一个具有给定值的Optional。如果value为null。那么第一个方法会抛出一个NullPointerException异常，而第二个方法会产生一个空Optional。
```

```Java
public static <T> Optional<T> empty()
产生一个空Optional。
```

## 7.6、用flatMap构建Optional值的函数

```java
public <U> Optional<U> flatMap(Function<? super T,? extends Optional<? extends U>> mapper)
如果Optional存在，产生将mapper应用于当前Optional值所产生的结果，或者在当前Optional为空时，返回一个空Optional。
```

## 7.7、将Optional转换为流

------

下面代码演示了Optional API的使用方式：

```java
public static void main(String[] args) throws IOException {

       String comments =  new String(Files.readAllBytes(Paths.get("D:/Backup/桌面/ciyu.txt")), StandardCharsets.UTF_8);
        List<String> wordList = Arrays.asList(comments.split("、"));

        Optional<String> optionalValue = wordList.stream().filter(s -> s.contains("人")).findFirst();
        /**
         * public T orElse(T other)
         * 产生这个Optional的值，或者在该Optional为空时，产生other
         */
        System.out.println(optionalValue.orElse("No Word")+" 包含\"人\"");

        /**
         * public static <T> Optional<T> empty()
         * 产生一个空Optional。
         */
        Optional<String> optionalString = Optional.empty();
        String result = optionalString.orElse("N/A");
        System.out.println("result: "+result);
        /**
         * public T orElseGet(Supplier<? extends T> supplier)
         * 产生这个Optional的值，或者在该Optional为空时，产生调用supplier的结果
         */
        result = optionalString.orElseGet(()-> Locale.getDefault().getDisplayName());
        System.out.println("result: "+result);
//        try {
//            /**
//             * public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X extends Throwable
//             * 产生这个Optional的值，或者在该Optional为空时，抛出调用exceptionSupplier的结果
//             */
//            result = optionalString.orElseThrow(IllegalAccessError::new);
//            System.out.println("result: "+result);
//        }catch (Throwable t){
//            t.printStackTrace();
//        }

        optionalValue = wordList.stream().filter(s -> s.contains("长")).findFirst();
        /**
         * public void ifPresent(Consumer<? super T> action)
         * 如果该Optional不为空，就将它的值传递给action。
         */
        optionalValue.ifPresent(s -> System.out.println(s+" 包含”长“"));

        Set<String> results = new HashSet<>();
        optionalValue.ifPresent(results::add);
        /**
         * public <U> Optional<U> map(Function<? super T,? extends U> mapper)
         * 产生一个Optional，如果当前的Optional的值存在，那么所产生的Optional的值是通过将给定的函数应用于当前的Optional的值而得到的；否则，产生一个空的Optional。
         */
        Optional<Boolean> added = optionalValue.map(results::add);
        System.out.println(added);

        /**
         * public <U> Optional<U> flatMap(Function<? super T,? extends Optional<? extends U>> mapper)
         * 产生将mapper应用于当前Optional值所产生的结果，或者在当前Optional为空时，返回一个空Optional。
         */
        System.out.println(inverse(4.0).flatMap(Test::squareRoot));
        System.out.println(inverse(-1.0).flatMap(Test::squareRoot));
        System.out.println(inverse(0.0).flatMap(Test::squareRoot));
        Optional<Double> result2 = Optional.of(-4.0).flatMap(Test::inverse).flatMap(Test::squareRoot);
        System.out.println(result2);

    }

    public static Optional<Double> inverse(double x){
        return x==0?Optional.empty():Optional.of(1/x);
    }

    public static Optional<Double> squareRoot(double x){
        return x<0?Optional.empty():Optional.of(Math.sqrt(x));
    }
```

# 8、收集结果

```java
public static Stream<String> noVowels() throws IOException {
    String contents = new String(Files.readAllBytes(Paths.get("D:/Backup/桌面/ciyu.txt")), StandardCharsets.UTF_8);
    List<String> wordList = Arrays.asList(contents.split("、"));
    Stream<String> words = wordList.stream();
    return words.map(s -> s.replaceAll("天","地"));
}

public static <T> void show(String label, Set<T> set){
    System.out.println(label+": "+set.getClass().getName());
    /**
     * <R,A> R collect(Collector<? super T,A,R> collector)
     * 使用给定的收集器来收集当前流中的元素。Collector类有用于多种收集器的工厂方法。
     * public static Collector<CharSequence,?,String> joining()
     * public static Collector<CharSequence,?,String> joining(CharSequence delimiter)
     * public static Collector<CharSequence,?,String> joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)
     * 产生一个连接字符串的收集器。分隔符会置于字符串之间，而第一个字符串之前可以有前缀，最后一个字符串之后可以有后缀。如果没有指定，那么它们都为空。
     */
    System.out.println("["+set.stream().limit(10).map(Objects::toString).collect(Collectors.joining(","))+"]");
}

public static void main(String[] args) throws IOException {

    /**
     * Iterator<T> iterator()
     * 产生一个用于获取当前流中各个元素的迭代器。这是一种终结操作。
     */
    Iterator<Integer> iterator = Stream.iterate(0,n->n+1).limit(10).iterator();
    while (iterator.hasNext()){
        System.out.println(iterator.next());
    }

    /**
     * Object[] toArray()
     * <A> A[] toArray(IntFunction<A[]> generator)
     * 产生一个对象数组，或者在将引用A[]::new传递给构造器时，返回一个A类型的数组这些操作都是终结操作。
     */
    Object[] numbers = Stream.iterate(0,n->n+1).limit(10).toArray();
    System.out.println("Object array:"+Arrays.toString(numbers));


    int number = (int) numbers[0];
    System.out.println("number:"+number);

    //System.out.println("以下语句引发异常:");
    //Integer[] number2 = (Integer[]) numbers;//ClassCastException: [Ljava.lang.Object; cannot be cast to [Ljava.lang.Integer;

    Integer[] number3 = Stream.iterate(0,n->n+1).limit(10).toArray(Integer[]::new);
    System.out.println("Integer array:"+Arrays.toString(number3));

    /**
     * public static <T> Collector<T,?,List<T>> toList()
     * public static <T> Collector<T,?,List<T>> toUnmodifiableList()
     * public static <T> Collector<T,?,Set<T>> toSet()
     * public static <T> Collector<T,?,Set<T>> toUnmodifiableSet()
     * 产生一个将元素收集到列表或集合中的收集器
     */
    Set<String> noVowelSet = noVowels().collect(Collectors.toSet());
    show("noVowelSet",noVowelSet);

    /**
     * public static <T,C extends Collection<T>> Collector<T,?,C> toCollection(Supplier<C> collectionFactory)
     * 产生一个将元素收集到任意集合中的收集器。可以传递一个诸如TreeSet::new的构造器引用。
     */
    TreeSet<String> noVowelTreeSet = noVowels().collect(Collectors.toCollection(TreeSet::new));
    show("noVowelTreeSet",noVowelTreeSet);

    String result = noVowels().limit(10).collect(Collectors.joining());
    System.out.println("Joining: "+result);
    result = noVowels().limit(10).collect(Collectors.joining(","));
    System.out.println("Joining with commas: "+result);

    /**
     * public static <T> Collector<T,?,IntSummaryStatistics> summarizingInt(ToIntFunction<? super T> mapper)
     * public static <T> Collector<T,?,LongSummaryStatistics> summarizingLong(ToLongFunction<? super T> mapper)
     * public static <T> Collector<T,?,DoubleSummaryStatistics> summarizingDouble(ToDoubleFunction<? super T> mapper)
     * 产生能够生成（Int|Long|Double）SummaryStatistics对象的收集器，通过它们可以获得将mapper应用于每个元素后所产生的结果的数量、总和、平均值、最大值和最小值。
     */
    IntSummaryStatistics summary = noVowels().collect(Collectors.summarizingInt(String::length));
    /**
     * public final long getCount()
     * 产生汇总后的元素的个数
     * public final （int|long|double） getSum()
     * public final double getAverage()
     * 产生汇总后的元素的总和或平均值，或者在没有任何元素时返回0
     * public final (int|long|double) getMax()
     * public final (int|long|double) getMin()
     * 产生汇总后的元素的最大值和最小值，或者在没有任何元素时，产生（Integer|Long|Double）.(Max|Min)_VALUE.
     */
    double averageWorldLength = summary.getAverage();
    double maxWorldLength = summary.getMax();
    System.out.println("Average World Length: "+averageWorldLength);
    System.out.println("Max World Length: "+maxWorldLength);
    System.out.println("forEach:");
    noVowels().limit(10).forEach(System.out::println);

}
```

# 9、收集到映射表中

```java
public static class Person{
    private int id;
    private String name;
    public Person(int id,String name){
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

public static Stream<Person> people(){
    return Stream.of(new Person(1,"xxx"),new Person(2,"ccc"),new Person(3,"zzz"));
}

public static void main(String[] args) throws Exception{

    /**
     * public static <T,K,U> Collector<T,?,Map<K,U>> toMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper)
     * public static <T,K,U> Collector<T,?,Map<K,U>> toMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper, BinaryOperator<U> mergeFunction)
     * public static <T,K,U,M extends Map<K,U>> Collector<T,?,M> toMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper, BinaryOperator<U> mergeFunction, Supplier<M> mapFactory)
     * public static <T,K,U> Collector<T,?,Map<K,U>> toUnmodifiableMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper)
     * public static <T,K,U> Collector<T,?,Map<K,U>> toUnmodifiableMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper, BinaryOperator<U> mergeFunction)
     * public static <T,K,U> Collector<T,?,ConcurrentMap<K,U>> toConcurrentMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper)
     * 产生一个收集器。它会产生一个映射表、不可修改的映射表或并发映射表。keyMapper和valueMapper函数会应用于每个收集到元素上，从而在所产生的映射表中生成一个键/值项。默认情况下，当两个元素产生相同的键时，会抛出一个IllegalStateException异常。你可以提供一个mergeFunction
     * 来合并具有相同键的值。默认情况下。其结果是一个HashMap或ConcurrentHashMap。你可以提供一个mapSupplier，它会产生所期望的映射表实例。
     */
    Map<Integer,String> idToName = people().collect(Collectors.toMap(Person::getId,Person::getName));
    System.out.println("idToName:"+idToName);

    Map<Integer,Person> idToPerson = people().collect(Collectors.toMap(Person::getId, Function.identity()));
    System.out.println("idToPerson:"+idToPerson.getClass().getName()+idToPerson);

    idToPerson = people().collect(Collectors.toMap(Person::getId,Function.identity(),(existingValue,newValue)->{throw new IllegalStateException();}, TreeMap::new));
    System.out.println("idToPerson:"+idToPerson.getClass().getName()+idToPerson);

    Stream<Locale> locales = Stream.of(Locale.getAvailableLocales());
    Map<String,String> languageNames = locales.collect(Collectors.toMap(Locale::getDisplayLanguage,l->l.getDisplayLanguage(l),(existingValue,newValue)->existingValue));
    System.out.println("languageNames:"+languageNames);

    locales = Stream.of(Locale.getAvailableLocales());
    Map<String, Set<String>> countryLanguageSets = locales.collect(Collectors.toMap(Locale::getDisplayCountry,l->new HashSet<>(Arrays.asList(l.getDisplayLanguage())),(a,b)->{
        Set<String> union = new HashSet<>(a);
        union.addAll(b);
        return union;
    }));
    System.out.println("countryLanguageSets:"+countryLanguageSets);

}
```

# 10、群组和分区

```java
public static void main(String[] args) {
        Stream<Locale> locales = Stream.of(Locale.getAvailableLocales());
        /**
         * public static <T,K> Collector<T,?,Map<K,List<T>>> groupingBy(Function<? super T,? extends K> classifier)
         * public static <T,K> Collector<T,?,ConcurrentMap<K,List<T>>> groupingByConcurrent(Function<? super T,? extends K> classifier)
         * 产生一个收集器，它会产生一个映射表或并发映射表，其键是将classifier应用于所有收集到的元素上所产生的结果，而值是由具有相同键的元素构成的一个个列表。
         */
//        Map<String, List<Locale>> countryToLocales = locales.collect(Collectors.groupingBy(Locale::getCountry));
//        List<Locale> swissLocales = countryToLocales.get("CH");
//        System.out.println(swissLocales);

        /**
         * public static <T> Collector<T,?,Map<Boolean,List<T>>> partitioningBy(Predicate<? super T> predicate)
         * 产生一个收集器，它会产生一个映射表，其键是true/false，而值是由满足/不满足断言的元素构成的列表。
         */
        Map<Boolean,List<Locale>> englishAndOtherLocales = locales.collect(Collectors.partitioningBy(l->l.getLanguage().equals("en")));
        List<Locale> englishLocales = englishAndOtherLocales.get(true);
        System.out.println(englishLocales);

    }
```

------

**注释：**如果调用groupingByConcurrent方法，就会在使用并行流时获得一个被并行组装的并行映射表。这与toConcurrentMap方法完全类似。

------

