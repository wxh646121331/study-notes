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



# 5 接口中的默认方法与表态方法

# 6 新时间日期API

# 7 其他新特性

