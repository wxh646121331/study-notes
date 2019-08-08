# 1 Lambda表达式

## 1.1 基础语法

- java8中引入了一个新的操作符"->"，将lambda表达式拆分成两部分：
  - 左侧：Lambda表达式的参数列表（接口中抽象方法的参数列表）

  - 右侧：Lambda表达式中所需执行的功能，即Lambda体（对抽象方法的实现）

## 1.2  语法格式

  - 无参数，无返回值：()->System.out.println("Hello");

    ~~~java
    int num = 0;  //JDK 1.7前，必须是final，JDK 1.8默认是final
    Runnable r = new Runnable() {
                @Override
                public void run() {
                    System.out.println("hello");
                }
            };
    Runnable r1 = ()-> System.out.println("Hello");
    ~~~

  - 有一个参数，无返回值

    ~~~java
    Consumer<String> con = (x)-> System.out.println(x);
    ~~~

  - 若只有一个参数，小括号可以省略不写

    ~~~java
    Consumer<String> con = x -> System.out.println(x);
    ~~~

  - 有两个及以上的参数，有返回值，并且Lambda体中有多条语句

    ~~~ java
    Comparator<Integer> com = (x, y) -> {
                System.out.println("hello");
                return Integer.compare(x, y);
            };
    ~~~

  - 若Lambda体中只有一条语句，大括号和return都可以省略

    ~~~java
    Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
    ~~~

  - Lambda表达式的参数列表的数据类型可以省略不写，因为JVM编译器通过上下文推断出数据类型，即“类型推断”

    ~~~java
    Comparator<Integer> com = (Integer x, Integer y) -> Integer.compare(x, y);
    //等价于
    Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
    ~~~

# 2 函数式接口

- Lambda表达式需要函数式接口支持

- 函数式接口：接口中只有一个抽象方法的接口，称为函数式接口，函数式接口可以用@FunctionalInterface注解修饰，可以检查是否是函数式接口3 方法引用与构造器引用

## 3.1 方法引用

- 若Lambda体中的内容有方法已经实现了，则可以使用“方法引用”
  
- Lambda体中调用方法的参数列表与返回值类型，要与函数式接口中抽象方法的函数列表和返回值类型保持一致
  
- 有三种语法格式：

  - 对象::实例方法名

    ~~~java
    PrintStream ps = System.out;
    //Lambda表达式
    Consumer<String> con = x -> ps.println(x);
    //方法引用
    Consumer<String> con2 = ps::println;
    ~~~

  - 类::静态方法名

    ~~~java
    Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
    Comparator<Integer> com1 = Integer::compare;
    ~~~

  - 类::实例方法名

    ~~~java
    BiPredicate<String, String> bp = (x, y) -> x.equals(y);
    //如果第一个参数是实例方法的调用者，第二个参数是实例方法的参数时，可以用类::实例方法名格式
    BiPredicate<String, String> bp1 = String::equals;
    ~~~

## 3.2 构造器引用

~~~java
//Lambda表达式
Supplier<Object> sup = () -> new Object();
//构造器引用
Supplier<Object> sup1 = Object::new;
~~~

## 3.3 数组引用 

~~~java
Function<Integer, String[]> fun = (x) -> new String[x];
String[] strs = fun.apply(10);
System.out.println(strs.length);
//数组引用       
Function<Integer, String[]> fun2 = String[]::new;
String[] strs2 = fun2.apply(20);
System.out.println(strs2.length);
~~~



# 4 Stream API

## 4.1 Stream简介

- 集合讲的是数据，流讲的是计算
- Stream特点：
  - Stream自己不会存储元素
  - Stream不会改变源对象，相反，他们会返回一个持有结果的新Stream
  - Stream操作是延迟执行的，这意味着他们会等到需要结果的时候才执行。

## 4.2 Stream操作的三个步骤

### 4.2.1 创建stream

- 一个数据源（如：集合、数组），获取一个流

- 获取流的方式：

  - 可以通过Collection 系统集合提供的stream()或parallelStream()

    ~~~java
    List<String> list = new ArrayList<>();
    Stream<String> stream1 = list.stream();
    ~~~

  - 通过Arrays中的静态方法stream()获取数据流

    ~~~java
    Employee[] emps = new Employee[10];
    Stream<Employee> stream2 = Arrays.stream(emps);
    ~~~

  - 通过Stream类中的静态方法of()

    ~~~java
    Stream<String> stream3 = Stream.of("aa","bb","cc");
    ~~~

  - 创建无限流

    ~~~java
    //迭代
    Stream<Integer> stream4 = Stream.iterate(0, x -> x+2);
    stream4.limit(10).forEach(System.out::println);
    //生成
    Stream.generate(() -> Math.random()).limit(5).forEach(System.out::println);
    ~~~

### 4.2.2 中间操作

- 一个中间操作链，对数据源的数据进行处理

- 多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理，而在终止操作时一次性全部处理，这外处理方法称为“惰性求值”

- 筛选与切片

  - filter(Predicate p)——接收Lambda，从流中排除某些元素

    ~~~java
    List<Employee> employees = Arrays.asList(new Employee("wxh", 30),
                    new Employee("tsb", 32),
                    new Employee("www", 23),
                    new Employee("dtt", 20));
    Stream<Employee> stream = employees.stream().filter(e -> e.getAge()>=30);
    //内部迭代
    stream.forEach(System.out::println);
    ~~~

  - limit——截断流，返回前n个元素的流

    ~~~java
    Stream<Employee> stream = employees.stream().filter(e -> e.getAge()>=30).limit(1);
            stream.forEach(System.out::println);
    ~~~

  - skip(n)——跳过元素，返回一个扔掉前n个元素的流。若流中不足n，则返回空流

  - distinct——筛选，通过流所生成元素的hashCode()和equals()去除重复元素

  - 映射

  - map——接收Lambda，将元素转换成其他形式或提取信息。接收一个函数作为参数，该函数会应用到每个元素上，并将其映射成一个新的元素

    ~~~java
    List<String> list = Arrays.asList("aaa", "bbb", "ccc");
    list.stream().map(str -> str.toUpperCase()).forEach(System.out::println);
    ~~~

  - flatMap——接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流

    ~~~java
    @Test
        public void test(){
            List<String> list = Arrays.asList("aaa", "bbb", "ccc");
            Stream<Stream<Character>> st = list.stream().map(TestStream::filter);
            st.forEach(x -> x.forEach(System.out::println));
            Stream<Character> st1 = list.stream().flatMap(TestStream::filter);
            st1.forEach(System.out::println);
    
        }
    
        public static Stream<Character> filter(String str){
            List<Character> list = new ArrayList<>();
            for(Character c : str.toCharArray()){
                list.add(c);
            }
            return list.stream();
        }
    ~~~

    

- 终止操作（终端操作）

  - 一个终止操作，执行中间操作链，并产生结果

# 5 接口中的默认方法与表态方法

# 6 新时间日期API

# 7 其他新特性