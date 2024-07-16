# JAVA

java是一种高级编程语言，由Sun Microsystems公司于1995年推出。Java的设计目标之一是实现“一次编写，到处运行”的能力，即编写的代码可以在任何支持Java的平台上运行，而无需进行额外的修改。Java广泛应用于企业级应用、桌面应用、移动应用和Web开发等领域。

## Java 优缺点

*   优点：
    *   跨平台性：通过在不同平台安装JVM，一套代码在不同操作系统上运行。
    *   面向对象：Java是一种面向对象编程的高级编程语言。
    *   内存管理：Java有严格的访问控制和内存管理机制，可以防止内存泄漏和程序崩溃，并提供安全性相关的特性，如安全沙箱机制和自动内存管理。
    *   多线程支持
    *   丰富的开发工具支持：Java拥有各种开发工具和框架，如集成开发环境（IDE）、调试器和性能分析工具，可以提高开发效率和代码质量。
*   缺点：
    *   垃圾回收机制：Java使用垃圾回收机制来自动管理内存，但这也会导致一些性能问题和额外的开销。
    *   执行速度较慢：由于Java是解释执行的语言，相比于编译语言执行速度较慢。
    *   内存消耗大：Java虚拟机会消耗大量的内存资源，对于某些嵌入式或资源受限的环境，不适合使用Java。
    *   安全性：虽然Java提供了一些安全特性，但仍然存在一些安全漏洞和风险，需要开发者注意安全编码的问题。

## 面向对象编程概念

*   抽象
    > 将一个具有现实意义的事物抽象为对象， 一个对象具有自己的状态，行为和标识。这意味着对象有自己的内部数据(提供状态)、方法 (产生行为)，并彼此区分（每个对象在内存中都有唯一的地址）。
*   接口(interface)

    > 每一个示例对象都具有他所属类型的通用特征和行为，每个对象仅能接受特定的请求触发对象的行为。我们向对象发送请求是通过他的接口定义的，对象的“类型”或“类”则规定了它的接口形式。
    > 允许一个类实现多个接口
    > 解耦，可以通过接口将方法的具体使用和实现类进行解耦，只关注行为，不关注具体实现。
    > Java 8 以后允许接口的方法拥有默认实现。

    |     特性    | 接口                            | 抽象类                  |
    | :-------: | :---------------------------- | :------------------- |
    |     组合    | 新类可以组合多个接口                    | 只能继承单一抽象类            |
    |     状态    | 不能包含属性（除了静态属性，不支持对象状态）        | 可以包含属性，非抽象方法可能引用这些属性 |
    | 默认方法和抽象方法 | 不需要在子类中实现默认方法。默认方法可以引用其他接口的方法 | 必须在子类中实现抽象方法         |
    |    构造器    | 没有构造器                         | 可以有构造器               |
    |    可见性    | 隐式public                      | 可以是protected或无       |
*   封装

    > 规范Java类的访问权限以及范围

    |   权限修饰符   | 权限   | 权限范围              |                  |
    | :-------: | ---- | ----------------- | :--------------- |
    |   public  | 公开   | 任何人都可以使用或访问该元素。   |                  |
    |  default  | 默认   | 该权限下的资源可以被同一包（库组件 | 同一子目录）中其他类的成员访问。 |
    | protected | 受保护的 | 只有子类能够访问。         |                  |
    |  private  | 私有的  | 只有类本身能够访问。        |                  |
*   复用
    > 我们可以将一个类的对象作为另一个类的成员变量使用，对不同类按需求进行组合。
    > 继承也是一种现有类的复用方式。
    > 当我们使用继承来实现复用效果时，一般情况下父类是一个通用类，两者之间通常表示为“是一个”的关系。而当我们使用组合的方式，通常表示类A有一个类B的关系。
*   继承(extends)
    > 子类继承(extends)父类能够访问父类权限是修饰符允许的属性以及方法，并能够通过编写与父类方法相同名称的方法，对父类方法进行覆盖。
    > Java 语法原则上只能够继承(extends)一个父类,但是可以通过多层继承实现多继承的效果。如：A extends B ; B extends C; C extends D;
*   多态
    > 向上转型，子类能够向上转型为父类，但同样也只能够获取父类拥有的属性和行为。
    > 创建：当我们实例化一个类时，会优先初始化父类，若父类同样拥有父类则依次向上递归。其次是初始化类属性，最后进行类实例化。
    > 销毁：当我们清理一个对象时，会按照子类 --> 父类的顺序依次进行清理。
*

## 初始化和清理

*   Java使用构造器对类进行实例化；若类为声明构造器，默认存在隐式无参构造器。若声明构造器，则清除隐式无参构造。
*   重载：多个类方法具有相同方法名，但参数或参数顺序不相同，返回值可以相同或不相同。
*   重写：子类方法和父类方法的名称、参数列表、返回值类型完全一致。
*   this，指代当前调用对象。
*   在Java中，垃圾回收器会自动地释放所有对象的内存.但是在个别情况下如io操作、资源占用等情况需要手动对资源进行释放。

## 数据结构

*   基本数据类型
    |   数据类型  |  字节数  |         默认值         |                    注                   |
    | :-----: | :---: | :-----------------: | :------------------------------------: |
    |   byte  | 1byte |          0          |                                        |
    |  short  | 2byte |          0          |                                        |
    |   int   | 4byte |          0          |                                        |
    |   long  | 8byte |          0          |                声明时，添加后缀L               |
    |  float  | 4byte |         0.0         |                  添加后缀F                 |
    |  double | 8byte |         0.0         |                                        |
    |   char  | 2byte | \u0000’是个空格，转化为整数是0 | 声明char类型的数据的时候，使用单引号声明。并且单引号里面只能放一个元素。 |
    | boolean | 1byte |        false        |                                        |
*   引用数据类型
    > `除去基本数据类型，都为引用数据类型`
*   关于使用原生类型或是包装类型的选择
    *   基本数据类型
        > 性能更好：原生数据类型在底层内存上的表示更紧凑，因此通常更节省内存，运行更快。
    *   引用数据类型
    > 可以表示null值；拥有更丰富的方法和属性；支持泛型；
    *   使用建议
    > 在考虑性能方面，建议优先使用原生类型。
    > 当需要更丰富的方法、表示null值、使用泛型时，建议使用包装类型。

## 运算符

*   类型提升
    > float型和double型相乘，结果是double型的；int和long相加，结果是long型。

*   精度丢失

    > 高精度数据类型向低精度数据类型进行转换时，低精度无法转义部分会被解掉；浮点数 29.7 被转换为整型值会丢失掉小数点后的数值。

    |         运算符         |       作用       | 注                                  |                                     |
    | :-----------------: | :------------: | ---------------------------------- | :---------------------------------- |
    |          =          |       赋值       |                                    |                                     |
    |      + - \* / %     |      算术运算符     | 运算符后都可以跟随'='号，表示运算后赋值              |                                     |
    |        ++ --        |     递增/减运算符    | 根据放置在变量前后位置不同，分别为参与运算前递增/减和运算后递增/减 |                                     |
    |   < >  =< => != ==  |      关系运算符     |                                    |                                     |
    |    &&    \|\|  !    |      关系运算符     | &和                                 | 如果是两个一起使用具有短路效果，如果第一个条件不满足就不再继续向下判断 |
    | & \| ^ \~ << >> >>> |      位运算符      | 操作字节码，运算符后都可以跟随'='号，表示运算后赋值        |                                     |
    |    boolean? a : b   |      三目运算符     |                                    |                                     |
    |  a instanceof type  | 检查a是否是type这个类型 |                                    |                                     |

*   控制流
    *   if-else
    ```java
      if(condition){

      }else if(condition){

      }else{

      }
    ```
    *   while
    ```java
    while(condition){

    }
    ```
    *   do-while
    > 在条件语句判断前，先运行一次循环；
    ```java
    do{

    }while(condition)
    ```
    *   for
    ```java
    for(int i=0; i<10; i++){

    }

    for(Object object in objects){

    }
    for(Object object : objects){

    }
    ```
    *   switch-case
    ```java
    switch(selector){
      case value 0 : 
          statement;
           break;
      case value 1 :
          statement;
          break;
      default :
          statement;
    }
    ```
    *   return
    > 从当前方法跳出;
    *   break
    > 从当前循环跳出;
    *   continue
    > 从本次循环跳出，继续循环;

## 字符串

> *   Java每个类从根本上都继承自Object类，都拥有继承自根类的toString()方法。在重载以后，使得它生成的String结果能够表达自身。若未重载则输出父类方法实现，Object方法默认为内存地址。若需要输出内存地址可以使用 super.toString() | Object.toString()。
>
> *   toString()循环调用：在toString()方法中使用this，编译器识别到this不是String类型就会调用this的toString()方法。
>
> ```java
>    @Override 
>    public String toString() {
>        return " InfiniteRecursion address: " + this + "\n"
>    }
> ```

### String

*   String是不可变的，因为他是只读的，所有指向String的引用都不能够修改它的值。
*   在使用”+”拼接字符串时，编译器会使用StringBuilder对字符串拼接进行优化，采用StringBuilder.appen()方法对字符串进行拼接。

### StringBuilder

*   更高效。
*   在使用append()方法时，注意不要在append()方法内使用“+”对字符串进行串联，会促使编译器在StringBuilder的基础上继续内部创建StringBuilder。

### StringBuffer

*   线程安全字符串。

### Formatter

> Java SE5 引入了format()方法，可用于PrintStream或者PrintWriter对象（你可以在附录:流式 I/O了解更多内容），其中也包括System.out对象。format()方法模仿了 C 语言的printf(),用于格式化输出。

|  类型 |     含义    |
| :-: | :-------: |
|  d  |  整型（十进制）  |
|  c  | Unicode字符 |
|  b  |  Boolean值 |
|  s  |   String  |
|  f  |  浮点数（十进制） |
|  e  | 浮点数（科学计数） |
|  x  |  整型（十六进制） |
|  h  | 散列码（十六进制） |
|  %  |   字面值“%”  |

### 正则表达式

> 一般来说，正则表达式就是以某种方式来描述字符串，并以此对字符串进行匹配。

```java
            Pattern p = Pattern.compile(regex);       
            Matcher m = p.matcher(content);
            m.find()    //从字符串中查找匹配位置
            m.matches() //字符串整体是否匹配
            m.lookingAt() //起始部位是否匹配    
```

*   正则表达式匹配符

|       表达式      |                      含义                      |
| :------------: | :------------------------------------------: |
|        B       |                     指定字符B                    |
|      \xhh      |                 十六进制值为0xhh的字符                |
|     \uhhhh     |            十六进制表现为0xhhhh的Unicode字符           |
|       \t       |                    制表符Tab                    |
|       \n       |                      换行符                     |
|       \r       |                      回车                      |
|       \f       |                      换页                      |
|       \e       |                  转义（Escape）                  |
|        .       |                     任意字符                     |
|     \[abc]     |          包含a、b或c的任何字符（和a\|b\|c作用相同）          |
|     \[^abc]    |               除a、b和c之外的任何字符（否定）              |
|    \[a-zA-Z]   |              从a到z或从A到Z的任何字符（范围）              |
|  \[abc\[hij]]  | a、b、c、h、i、j中的任意字符（与a\|b\|c\|h\|i\|j作用相同）（合并） |
| \[a-z&&\[hij]] |                  任意h、i或j（交）                  |
|       \s       |             空白符（空格、tab、换行、换页、回车）             |
|       \S       |                 非空白符（\[^\s]）                 |
|       \d       |                  数字（\[0-9]）                  |
|       \D       |                 非数字（\[^0-9]）                 |
|       \w       |              词字符（\[a-zA-Z\_0-9]）             |
|       \W       |                 非词字符（\[^\w]）                 |

| 逻辑操作符 |                    含义                   |
| :---: | :-------------------------------------: |
|   XY  |                  Y跟在X后面                 |
|  X\|Y |                   X或Y                   |
|  (X)  | 捕获组（capturing group）。可以在表达式中用\i引用第i个捕获组 |

| 边界匹配符 |    含义    |
| :---: | :------: |
|   ^   |   一行的开始  |
|   \$  |   一行的结束  |
|   \b  |   词的边界   |
|   \B  |   非词的边界  |
|   \G  | 前一个匹配的结束 |

*   贪婪型： 匹配量词总是贪婪的，除非有其他的选项被设置。贪婪表达式会为所有可能的模式发现尽可能多的匹配。导致此问题的一个典型理由就是假定我们的模式仅能匹配第一个可能的字符组，如果它是贪婪的，那么它就会继续往下匹配。`尽可能地匹配所有符合的表达式`

*   懒惰型： 用问号来指定，这个量词匹配满足模式所需的最少字符数。因此也被称作懒惰的、最少匹配的、非贪婪的或不贪婪的。`仅匹配一次`

*   占有型： 目前，这种类型的量词只有在 Java 语言中才可用（在其他语言中不可用），并且也更高级，因此我们大概不会立刻用到它。当正则表达式被应用于String时，它会产生相当多的状态，以便在匹配失败时可以回溯。而“占有的”量词并不保存这些中间状态，因此它们可以防止回溯。它们常常用于防止正则表达式失控，因此可以使正则表达式执行起来更高效。

|   贪婪型  |  勉强型`?` |  占有型`+` |         如何匹配        |
| :----: | :-----: | :-----: | :-----------------: |
|   X?   |   X??   |   X?+   |      一个或零个X`?`      |
|   X\*  |   X\*?  |   X\*+  |      零个或多个X`*`      |
|   X+   |   X+?   |   X++   |      一个或多个X`+`      |
|  X{n}  |  X{n}?  |  X{n}+  |      恰好n次X`{n}`     |
|  X{n,} |  X{n,}? |  X{n,}+ |     至少n次X`{n,}`     |
| X{n,m} | X{n,m}? | X{n,m}+ | X至少n次，但不超过m次`{n,m}` |

*   find()

> find()方法像迭代器那样向前遍历输入字符串。而第二个重载的find()接收一个整型参数，该整数表示字符串中字符的位置，并以其作为搜索的起点。

*   Groups
    组是用括号划分的正则表达式，可以根据组的编号来引用某个组。组号为 0 表示整个表达式，组号 1 表示被第一对括号括起来的组
    `A(B(C))D`中有三个组：组 0 是ABCD，组 1 是BC，组 2 是C。
    *   public String group(int i)    返回前一次匹配操作（例如find()）的第 0 组（整个匹配）。
    *   public int groupCount()   返回该匹配器的模式中的分组数目，组 0 不包括在内。
    *   public int start(int group)   返回在前一次匹配操作中寻找到的组的起始索引。`若匹配失败调用方法则会NPE`
    *   public int end(int group) 返回在前一次匹配操作中寻找到的组的最后一个字符索引加一的值。`若匹配失败调用方法则会NPE`

|              编译标记             | 效果                                                                                                                                             |
| :---------------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------- |
|       Pattern.CANON\_EQ       | 当且仅当两个字符的完全规范分解相匹配时，才认为它们是匹配的。例如，如果我们指定这个标记，表达式`\u003F`就会匹配字符串`?`。默认情况下，匹配不考虑规范的等价性                                                            |
| Pattern.CASE\_INSENSITIVE(?i) | 默认情况下，大小写不敏感的匹配假定只有`US-ASCII`字符集中的字符才能进行。这个标记允许模式匹配不考虑大小写（大写或小写）。通过指定`UNICODE_CASE`以及`Pattern.CASE_INSENSITIVE(?i)` `Unicode`字符集也可以实现大小写不敏感的匹配 |
|      Pattern.COMMENTS(?x)     | 忽略空格符，并且以`#`开始直到行末的注释也会被忽略掉。通过嵌入的标记表达式也可以开启`Unix`的行模式                                                                                          |
|       Pattern.DOTALL(?s)      | 在dotall模式下，表达式`.`匹配所有字符，包括行终止符。默认情况下，`.`不会匹配行终止符                                                                                               |
|     Pattern.MULTILINE(?m)     | 在多行模式下，表达式`\^`和`\$`分别匹配一行的开始和结束。默认情况下，这些表达式仅匹配输入的完整字符串的开始和结束                                                                                   |
|   Pattern.UNICODE\_CASE(?u)   | 参考Pattern.CASE\_INSENSITIVE(?i)用于开启`Unicode`字符集不区分大小写                                                                                          |
|    Pattern.UNIX\_LINES(?d)    | 在这种模式下，在`.、^和$`的行为中，只识别行终止符`\n`                                                                                                                |

```java
// 可以通过“或”(|)操作符组合多个标记的功能
Pattern p =  Pattern.compile("^java", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE);
```

```java
String input = "This!!unusual use!!of exclamation!!points";
Pattern.compile("!!").split(input);
```

*   split() 可以配合Pattern对正则表达式进行匹配分割

*   replaceFirst(String replacement)  以参数字符串replacement替换掉第一个匹配成功的部分。

*   replaceAll(String replacement)    以参数字符串replacement替换所有匹配成功的部分。

*   appendReplacement(StringBuffer sbuf, String replacement)  执行渐进式的替换，它允许你调用其他方法来生成或处理replacement（replaceFirst()和replaceAll()则只能使用一个固定的字符串），第一个参数为StringBuffer，能够对可变字符串进行匹配操作。

*   appendTail(StringBuffer sbuf) 在执行了一次或多次appendReplacement()之后，调用此方法可以将输入字符串余下的部分复制到sbuf中。

*   正则表达式匹配文件输入

```java
        Pattern p = Pattern.compile(args[1]);    
        int index = 0;    
        //在循环体外创建一个空的Matcher对象，然后用reset()方法每次为Matcher加载一行输入，这种处理会有一定的性能优化。
        Matcher m = p.matcher("");    
        for(String line: Files.readAllLines(Paths.get(args[0]))) {      
            m.reset(line);      
            while(m.find())        
                System.out.println(index++ + ": " +          
                  m.group() + ": " + m.start());    
        }  
```

### 扫描文件输入

> 从文件或标准输入读取数据,如使用readLine()一行一行读，对需要的内容需要对每行数据进行条件分割。
> Java SE5 新增了Scanner类，构造器可以接收任意类型的输入对象，包括File、InputStream、String或者像此例中的Readable实现类。
> 有了Scanner，所有的输入、分词、以及解析的操作都隐藏在不同类型的next方法中。普通的next()方法返回下一个String。所有的基本类型（除char之外）都有对应的next方法，包括BigDecimal和BigInteger。所有的 next 方法，只有在找到一个完整的分词之后才会返回。Scanner还有相应的hasNext方法，用以判断下一个输入分词是否是所需的类型，如果是则返回true。

*   Scanner
    *   默认情况下，Scanner根据空白字符对输入进行分词，但是你可以用正则表达式指定自己所需的分隔符：
    *   在输入结束时会抛出IOException，所以Scanner会把IOException吞掉。
    *   delimiter() 用来返回当前正在作为分隔符使用的Pattern对象。
    *   useDelimiter() 设置分割符

```java
public static BufferedReader input =  new BufferedReader(new StringReader("Sir Robin of Camelot\n22 1.61803")); 

Scanner stdin = new Scanner(SimpleRead.input);
scanner.useDelimiter("\\s*,\\s*"); 
while(scanner.hasNextInt())    
    System.out.println(scanner.nextInt());  
```

## 集合

> *   Iterable `迭代器（可迭代的）`
> *   Iterator `迭代器（迭代器对象具体接口方法）`
>     *   Collection `集合根类`
>         *   List
>             *   ArrayList `数组集合`
>             *   LinkedList `链表集合`
>             *   Vector `数组、线程安全集合，多线程情况下有更优线程安全类型选择`
>         *   Set
>             *   HashSet
>                 *   LinkedHashSet
>             *   TreeSet
>     *   Map
>         *   HashMap
>         *   TreeMap

### List

*   ArrayList
    ArrayList维护了一个Object\[]数组，size字段、DEFAULT\_CAPACITY初始化默认数组大小。所以数据结构同数组相同。
    ```java
        private int size;
        private static final int DEFAULT_CAPACITY = 10;
        int newCapacity = ArraysSupport.newLength(
            //当前数组长度。
            oldCapacity, 
            //add()方法传递minCapacity = oldCapacity+1
            minCapacity - oldCapacity,
         /* 最小增长长度，右移一位相当于除二小数位忽略*/
        oldCapacity >> 1   );
    ```
    *   LinkedList&#x20;

        LinkedList使用链表数据结构，linkedList每个节点都是一个Node对象；

        由于链表数据结构的特殊性，链表在扩容时只需要添加节点即可。不需要设置初始大小值，以及增长量；

        ```java
            transient int size = 0;

            /**
             * Pointer to first node.
             */
            transient Node<E> first;

            /**
             * Pointer to last node.
             */
            transient Node<E> last;

        private static class Node<E> {
            E item;	//数据
            Node<E> next;	//下一个节点位置信息
            Node<E> prev;	//上一个节点位置信息

            Node(Node<E> prev, E element, Node<E> next) {
                this.item = element;
                this.next = next;
                this.prev = prev;
            }
        }
         
        // 链表每次获取指定位置数据都需要对表进行遍历  
        Node<E> node(int index) {
              // assert isElementIndex(index);
              if (index < (size >> 1)) {
                  Node<E> x = first;
                  for (int i = 0; i < index; i++)
                      x = x.next;
                  return x;
              } else {
                  Node<E> x = last;
                  for (int i = size - 1; i > index; i--)
                      x = x.prev;
                  return x;
              }
        }
        ```

### Set

Set值唯一

*   HashSet&#x20;

    HashSet基于HashMap实现；默认初始化大小16；加载因子为0.75（初始值占用大小占总量的0.75）；

    HashSet的值存入HashMap的Key，Value始终存储PRESENT Object对象；

    HashSet存储数据无序；

    ```java
    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    /**
     * Constructs a new, empty set; the backing {@code HashMap} instance has
     * default initial capacity (16) and load factor (0.75).
     */
    public HashSet() {
        map = new HashMap<>();
    }
    ```
*   TreeSet

    TreeSet存储数据按照Comparator进行比较排序；

    TreeSet基于TreeMap实现，

### Map

*   HashMap
*   TreeMap

## 类型信息

RTTI（RunTime Type Information，运行时类型信息）能够在程序运行时发现和使用类型信息。

RTTI 和反射的真正区别在于，使用 RTTI 时，编译器在编译时会打开并检查`.class`文件。换句话说，你可以用“正常”的方式调用一个对象的所有方法。通过反射，`.class`文件在编译时不可用；它由运行时环境打开并检查。

### 泛化的Class

*   某一个类的Class与类本身，虽然类的Class代表这一个类，但本质上依旧是一个Class，泛型的继承、实现关系不能够以类本身的类型信息理解。

```java
public class GenericClassReferences {
    public static void main(String[] args) {
        Class intClass = int.class;
        Class<Integer> genericIntClass = int.class;
        genericIntClass = Integer.class; // 同一个东西
        intClass = double.class;
        // genericIntClass = double.class; // 非法
		// Integer继承自Number。但事实却是不行，因为Integer的Class对象并不是Number的Class对象的子类
		// Class<Number> geenericNumberClass = int.class; 非法
    }
}
```

### 类型转换检测

```java
//静态类型判断
if(b instanceof a)
    // error: illegal generic type for instanceof
    if (arg instanceof T) {
    }

//动态
if(b.isInstance(a))
    public boolean f(Object arg) {
        return kind.isInstance(arg);
    }
```

### 类的等价比较

*   instanceof 与 isInstance 结果相同
*   \== 与 equals 结果相同

```java
// 子类： class typeinfo.Base
x instanceof Base true
x instanceof Derived false
Base.isInstance(x) true
Derived.isInstance(x) false
x.getClass() == Base.class true
x.getClass() == Derived.class false
x.getClass().equals(Base.class)) true
x.getClass().equals(Derived.class)) false

// 父类class typeinfo.Derived
x instanceof Base true
x instanceof Derived true
Base.isInstance(x) true
Derived.isInstance(x) true
x.getClass() == Base.class false
x.getClass() == Derived.class true
x.getClass().equals(Base.class)) false
x.getClass().equals(Derived.class)) true
```

### 类方法的提取

```java
// 获取类本身以及父类、父类接口所有方法(仅public)
Class.getMethods()

// 获取类本身所有方法(所有权限修饰符)
Class.getDeclaredMethods()
```

## 泛型

### 泛型声明方式

```java
// 泛型接口
public class BasicSupplier<T> implements Supplier<T> {}

//泛型方法 若方法是静态的，必须生命泛型。
 public static <T> List<T> makeList(T... args) {}
 
 public static <A, B, C, D, E> Tuple5<A, B, C, D, E> tuple(A a, B b, C c, D d, E e) {
        return new Tuple5<>(a, b, c, d, e);
}

// 动态生命泛型类型
new T(); // 不能够以这种形式声明对象
T[] array = (T[]) new Object[sz]; //允许声明Object类型数组、再强转为目标泛型
```

### 泛型擦除

*   Class 并不会记录泛型

```java
    Class c1 = new ArrayList<String>().getClass();
    Class c2 = new ArrayList<Integer>().getClass();
    System.out.println(c1 == c2); //true
```

*   无法直接通过泛型调用指定方法，需要指定类型边界\<? extends Parent>

```java
class Manipulator<T> {
    private T obj;
    
    Manipulator(T x) {
        obj = x;
    }
    
    // Error: cannot find symbol: method f():
    public void manipulate() {
        obj.f();
    }
}

public class Manipulator2<T extends HasF> {
    private T obj;

    Manipulator2(T x) {
        obj = x;
    }

    public void manipulate() {
        obj.f();
    }
}
```

### 边界

```java
// 继承自Coord 并实现了HasColor
class WithColorCoord2<T extends Coord & HasColor> extends WithColor2<T> {} 

// contains() 和 indexOf() 接受的参数类型是 Object。
// 因此当你指定一个 ArrayList<? extends Fruit> 时，add() 的参数就变成了"? extends Fruit"
public static void main(String[] args) {
    List<? extends Fruit> flist = Arrays.asList(new Apple());
    Apple a = (Apple) flist.get(0); // No warning
    flist.contains(new Apple()); // Argument is 'Object'
    flist.indexOf(new Apple()); // Argument is 'Object'
}


    public static void main(String[] args) {
        Holder<Apple> apple = new Holder<>(new Apple());
        Apple d = apple.get();
        apple.set(d);
        // Holder<Fruit> fruit = apple; // Cannot upcast
        Holder<? extends Fruit> fruit = apple; // OK
        Fruit p = fruit.get();
        d = (Apple) fruit.get();
        try {
            Orange c = (Orange) fruit.get(); // No warning
        } catch (Exception e) {
            System.out.println(e);
        }
        // fruit.set(new Apple()); // Cannot call set()
        // fruit.set(new Fruit()); // Cannot call set()
        System.out.println(fruit.equals(d)); // OK
    }
```

## 枚举

## 异常处理

## 日期时间

## 文件 nio

## 函数式编程

## 并发编程
> 在单处理器主机上使用并发编程并不一定会提高程序的运行速度，过多的线程切换同样会给处理器增加额外的开销。但在线程会出现堵塞、等待等情况下，并发编程通常会产生令人满意的运行效率。

* 关于thread.join()
  如果在当前线程(可以是main()主线程，或是某一线程内调用), 线程thread调用.join()，那么当前线程将会进入堵塞状态，等待thread运行结束后，才能够继续向下运行。

### 术语
* 并发：并发是关于正确有效地控制对共享资源的访问。`并发性是一系列性能技术，专注于减少等待`。
> 并发的关键在于`减少等待`，如果程序的某些部分被迫等待，利用并发提高程序整体的运行效率，减少等待造成的效率影响。`并发`如果在没有什么可以等待，那么就没有可加速的地方。
* 并行：并行是使用额外的资源来更快地产生结果。
    * 纯并发：仍然在单个CPU上运行任务。纯并发系统比顺序系统更快地产生结果，但是它的运行速度不会因为处理器的增加而变得更快。
    * 并发-并行：使用并发技术，结果程序可以利用更多处理器更快地产生结果。
    * 并行-并发：使用并行编程技术编写，如果只有一个处理器，结果程序仍然可以运行（Java 8 Streams就是一个很好的例子）。
    * 纯并行：除非有多个处理器，否则不会运行。 
* 多任务
* 多处理
* 多线程
* 分布式系统
* 线程：多个线程共享内存和I/O等资源，因此编写多线程程序时遇到的困难是在不同的线程驱动的任务之间协调这些资源，一次不能通过多个任务访问它们。
* 进程：是一个拥有自己的地址空间的独立的程序，进程与进程之间隔离不会相互影响。

### 注意点
* 尽可能的少编写自己的代码，去使用完善的库。
* 并发编程可能会在任意部分出现意料之外的问题，例如，你必须知道`处理器缓存`以及保持`本地缓存`与`主内存`一致的问题。你必须深入了解`对象构造的复杂性`，以便你的构造器不会`意外地`将数据暴露给其他线程`进行更改`。问题还有很多。
* 它起作用,并不意味着它没有问题
* 线程是否独立，是否存在与其他线程共享的内存、变量、外部资源。
* 
### 多线程的几种实现方式
* 线程池 ExecutorService
  1. `Callable` 和 `Runnable`
    * Runnable: 该接口定义的多线程任务无返回值，并且不会抛出异常
    ```java
        @FunctionalInterface
        public interface Runnable {
            void run();
        }

        ExecutorService executor = Executors.newFixedThreadPool(10);

        Runnable runnableTask = () -> {
            System.out.println("Executing Runnable task");
        };

        executor.submit(runnableTask);
        executor.shutdown();
    ```

    * Callable: 该接口可以返回结果，并抛出异常检查。
    ```java
        @FunctionalInterface
        public interface Callable<V> {
            V call() throws Exception;
        }

        ExecutorService executor = Executors.newFixedThreadPool(10);

        Callable<String> callableTask = () -> {
            return "Task result";
        };

        Future<String> future = executor.submit(callableTask);

        try {
            String result = future.get();
            System.out.println("Callable result: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            executor.shutdown();
        }

    ```
  2. 标准用法
        > 避免使用 Executors 类中的静态工厂方法，如 newFixedThreadPool、newCachedThreadPool，因为它们返回的线程池可能会存在潜在的问题，如无界队列。可以使用 ThreadPoolExecutor 类来精细控制线程池的参数。
        ```java
        ExecutorService executor = new ThreadPoolExecutor(
            2, 4, 60L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>()
        );

        ```
     1. 配置合理的核心池大小和最大池大小：
        * 根据应用程序的需求和系统的资源配置合理的核心池大小和最大池大小。可以通过 CPU 核心数和任务的性质（CPU 密集型还是 IO 密集型）来进行估算。
       ```java
       int corePoolSize = Runtime.getRuntime().availableProcessors();
       int maxPoolSize = corePoolSize * 2;
       ExecutorService executor = new ThreadPoolExecutor(
           corePoolSize, maxPoolSize, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>()
       );
       ```
     2. 线程工厂自定义线程名称
        * 自定义 ThreadFactory 可以为线程池中的线程设置有意义的名称，以便于调试和监控。
       ```java
       ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
           .setNameFormat("worker-%d")
           .build();
       ExecutorService executor = new ThreadPoolExecutor(
           corePoolSize, maxPoolSize, 60L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(), namedThreadFactory
       );
       ```
     3. 线程池配置拒绝策略
       ```java
       ExecutorService executor = new ThreadPoolExecutor(
           corePoolSize, maxPoolSize, 60L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(), new ThreadPoolExecutor.CallerRunsPolicy()
       );
       ```

* 多线程流
  1. 并行流 Stream
    > 数据源的分割特性：数据源的分割特性（splittable）会影响并行流的性能。例如，ArrayList 和 Arrays 的分割性能较好，而 LinkedList 的分割性能较差。分割性较好的数据类型，并行流更够将数据源更好的分割，多线程并行处理，而分割性较差的数据类型，并行流无法将数据进行分割，并行流无法产生应有的效率，甚至会更缓慢。
     * Stream.generate().parallel()： Stream.generate 会无限动态创建过多线程调用 Supplier，直到流的终止操作（例如 limit）告诉它停止。由于是并行流，所以会有多个线程并行地调用 IntGenerator 的 get 方法。会产生超过需求的方法调用。
     * IntStream.range().parallel()：流在调用 .parallel() 之前就知道元素数量，以及跳出条件，并行流会合理分配线程进行处理，而不会像无限流那样需要动态创建过多的线程。
     * 默认的并行流使用 ForkJoinPool.commonPool()。可以通过设置自定义的 ForkJoinPool 来控制并行任务的执行环境。
       ```java
       ForkJoinPool customThreadPool = new ForkJoinPool(4);
       customThreadPool.submit(() -> {
           Stream.of(1, 2, 3, 4, 5)
               .parallel()
               .forEach(i -> System.out.println(Thread.currentThread().getName() + " - " + i));
       }).get();
       ```

  2. Lambda
    > 线程池也可以通过执行`Lambda`表达式以达到与`Runnable` 或 `Callable` 相同的实现效果。
    ```java
    class NotRunnable {
        public void go() {
            System.out.println("NotRunnable");
        }
    }
    class NotCallable {
        public Integer get() {
            System.out.println("NotCallable");
            return 1;
        }
    }
    public class LambdasAndMethodReferences {
        public static void main(String[] args)throws InterruptedException {
        ExecutorService exec =
            Executors.newCachedThreadPool();
        exec.submit(() -> System.out.println("Lambda1"));
        exec.submit(new NotRunnable()::go);
        exec.submit(() -> {
            System.out.println("Lambda2");
            return 1;
        });
        exec.submit(new NotCallable()::get);
        exec.shutdown();
        }
    }
    ```
* CompletableFuture
  > `CompletableFuture` 提供了强大的异步编程能力，可以链式调用和组合多个异步任务。
  ```java
  CompletableFuture.supplyAsync(() -> {
      // 异步任务
      return "Result";
  }).thenApply(result -> {
      // 后续处理
      return result.toUpperCase();
  }).thenAccept(System.out::println);
  ```
  1. 同步调用 `thenApply()` 和异步调用 `thenApplyAsync()`
* 其他多线程应用方式

  1. 虚拟线程池 Project Loom
    > 虚拟线程是 JDK 20 的新特性，允许创建大量轻量级线程，从而更高效地利用多核处理器。

    ```java
    Thread.startVirtualThread(() -> {
        // 虚拟线程任务
        System.out.println("Hello from virtual thread!");
    });
    ```
  2. 结构化并发
    > 结构化并发提供了一种新的编程模型，使得并发编程更加简洁和安全。

    ```java
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        Future<String> task1 = scope.fork(() -> "Task 1 result");
        Future<String> task2 = scope.fork(() -> "Task 2 result");
        scope.join();           // 等待所有任务完成
        scope.throwIfFailed();  // 如果任何一个任务失败则抛出异常
        System.out.println(task1.resultNow());
        System.out.println(task2.resultNow());
    }
    ```

### 线程池

### CompletableFuture


## 注解

### 元注解（meta-annoation）

| 注解          | 解释                                                                                                                                                                                                 |                       |
| :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------- |
| @Target     | 表示注解可以用于哪些地方。可能的 ElementType 参数包括：</br>CONSTRUCTOR：构造器的声明</br>FIELD：字段声明（包括 enum 实例）</br>LOCAL\_VARIABLE：局部变量声明</br>METHOD：方法声明</br>PACKAGE：包声明</br>PARAMETER：参数声明</br>TYPE：类、接口（包括注解类型）或者 enum 声明 | SOURCE：注解将被编译器丢弃</br> |
| @Retention  | 表示注解信息保存的时长。可选的 RetentionPolicy 参数包括：</br>SOURCE：注解将被编译器丢弃</br>CLASS：注解在 class 文件中可用，但是会被 VM 丢弃。</br>RUNTIME：VM 将在运行期也保留注解，因此可以通过反射机制读取注解的信息。                                                      |                       |
| @Documented | 将此注解保存在 Javadoc 中                                                                                                                                                                                  |                       |
| @Inherited  | 允许子类继承父类的注解                                                                                                                                                                                        |                       |
| @Repeatable | 允许一个注解可以被使用一次或者多次（Java 8）。                                                                                                                                                                         |                       |

### 注解元素可用的类型如下所示：

*   所有基本类型（int、float、boolean等）
*   String
*   Class
*   enum
*   Annotation （注解嵌套：注解也可以作为元素的类型。）
*   以上类型的数组
*   注意，也不允许使用任何包装类型，但是由于自动装箱的存在，这不算是什么限制。

### 注解处理器

> 如果没有用于读取注解的工具，那么注解不会比注释更有用。

*   通过操作class 确认方法或类是否被标注注解

```java
trackUseCases(List<Integer> useCases, Class<?> cl) {
        // 获取所有方法
        for(Method m : cl.getDeclaredMethods()) {
        // 获取方法上是否被指定注解标注
            UseCase uc = m.getAnnotation(UseCase.class);
            if(uc != null) {
                System.out.println("Found Use Case " +
                        uc.id() + "\n " + uc.description());
                useCases.remove(Integer.valueOf(uc.id()));
            }
        }
        useCases.forEach(i ->
                System.out.println("Missing use case " + i));
    }
```

### 注解默认值限制

*   注解元素在声明中，所有元素都存在，并且具有相应的值。不能有不确定的值。也就是说，元素要么有默认值，要么就在使用注解时提供元素的值。
*   无论是在源代码声明时还是在注解接口中定义默认值时，都不能使用 null 作为其值。解决方案是提供每个类型对应的空值。(如String为 "")
*   注解不支持继承

# 数据结构

## 红黑树（平衡二叉树）

# 算法

## Hash算法

把任意长度的输入，通过散列算法，变换成固定长度的输出，就是散列值；

*   特性：
    *   相同的输入一定会产生相同的结果

    *   不相同的输入也会产生相同的结果，称之为碰撞
*   常见Hash函数
    *   直接定址法：直接以关键字k或者k加上某个常数（k+c）作为哈希地址。

    *   数字分析法：提取关键字中取值比较均匀的数字作为哈希地址。

    *   除留余数法：用关键字k除以某个不大于哈希表长度m的数p，将所得余数作为哈希表地址。

    *   分段叠加法：按照哈希表地址位数将关键字分成位数相等的几部分，其中最后一部分可以比较短。然后将这几部分相加，舍弃最高进位后的结果就是该关键字的哈希地址。

    *   平方取中法：如果关键字各个部分分布都不均匀的话，可以先求出它的平方值，然后按照需求取中间的几位作为哈希地址。

    *   伪随机数法：采用一个伪随机数当作哈希函数。
*   Hash地址定位解决地址碰撞的方式
    *   开放定址法：开放定址法就是一旦发生了冲突，就去寻找下一个空的散列地址，只要散列表足够大，空的散列地址总能找到，并将记录存入。

    *   链地址法：将哈希表的每个单元作为链表的头结点，所有哈希地址为i的元素构成一个同义词链表。即发生冲突时就把该关键字链在以该单元为头结点的链表的尾部。

    *   再哈希法：当哈希地址发生冲突用其他的函数计算另一个哈希函数地址，直到冲突不再产生为止。

    *   建立公共溢出区：将哈希表分为基本表和溢出表两部分，发生冲突的元素都放入溢出表中。

# 设计模式

## 工厂

## 动态代理

```java
// 当一个类实现了Serilizable接口，这个类的属性和方法就能够被自动序列化，而被@transient标记的属性（方法和类不能够被标记）不会被序列化，并且它的的生命周期仅存于内存，不会写到磁盘进行持久化；
// 一个静态变量不管是否被transient标记都不能够被序列化。若静态变量反序列化后存在值，反序列化后类中static型变量的值为当前JVM中对应static变量的值，这个值是JVM中的不是反序列化得出的。
//但是实现Externalizable接口，并重写接口方法可以实现序列化需要的属性，无论是否被@transient 或者static修饰
@transient

```

### 注解@JsonFormat主要是后台到前台的时间格式的转换

@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH\:mm\:ss", timezone = "GMT+8")

### 注解@DateTimeFormat主要是前台到后台的时间格式的转换 "yyyy-MM-dd HH-mm-ss"

## 关于判空

ispresent()
isblank() 若为空，空字符也为空
isEmpty() 长度=0

## Http发送请求

```java
http发送请求

/**
 * wesoft.com Inc.
 * Copyright (c) 2005-2016 All Rights Reserved.
 */
package com.wesoft.util;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

import javax.validation.constraints.NotNull;

import org.apache.http.HttpEntity;
import org.apache.http.NameValuePair;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;
import com.fasterxml.jackson.databind.ObjectMapper;

public final class HttpRequest {

    private static ObjectMapper om = new ObjectMapper();

    /**
     * 通过GET方式发起http请求
     */
    public static String sendGet(String url) throws IOException {
        CloseableHttpClient httpClient = getHttpClient();
        try {
            HttpGet get = new HttpGet(url);
            CloseableHttpResponse httpResponse;
            httpResponse = httpClient.execute(get);
            HttpEntity entity = httpResponse.getEntity();
            if (entity != null) {
                return EntityUtils.toString(entity);
            } else {
                return null;
            }
        } finally {
            closeHttpClient(httpClient);
        }
    }

    public static String sendPost(String url, @NotNull Map<String, String> parameters) throws IOException {
        return sendPost(url, parameters, "UTF-8");
    }

    /**
     *
     * @param url
     * @param parameters
     * @param code 编码方式
     * @return
     */
    public static String sendPost(String url, @NotNull Map<String, String> parameters,
                                  String code) throws IOException {
        CloseableHttpClient httpClient = getHttpClient();
        try {
            HttpPost post = new HttpPost(url);
            List<NameValuePair> list = new ArrayList<>();

            parameters.forEach((key, val) -> list.add(new BasicNameValuePair(key, val)));

            UrlEncodedFormEntity uefEntity = new UrlEncodedFormEntity(list, code);
            post.setEntity(uefEntity);

            CloseableHttpResponse httpResponse = httpClient.execute(post);
            HttpEntity entity = httpResponse.getEntity();
            if (entity != null) {
                return EntityUtils.toString(entity);
            } else {
                return null;
            }

        } finally {
            closeHttpClient(httpClient);
        }

    }

    public static String sendPostByJson(String url,
                                        @NotNull Map<String, String> parameters) throws IOException {
        CloseableHttpClient httpClient = getHttpClient();
        HttpPost httpPost = new HttpPost(url);
        StringEntity stringEntity = new StringEntity(om.writeValueAsString(parameters),
            StandardCharsets.UTF_8);

        httpPost.setHeader("content-type", "application/json;charset=utf-8");
        httpPost.setEntity(stringEntity);

        try (CloseableHttpResponse httpResponse = httpClient.execute(httpPost);) {
            HttpEntity entity = httpResponse.getEntity();
            if (entity != null) {
                return EntityUtils.toString(entity);
            } else {
                return null;
            }
        } finally {
            closeHttpClient(httpClient);
        }

    }

    private static CloseableHttpClient getHttpClient() {
        return HttpClients.createDefault();
    }

    private static void closeHttpClient(CloseableHttpClient client) throws IOException {
        if (client != null) {
            client.close();
        }
    }

}

```

## 加解密 AES MD5

```java
private String encryptMd5(String phone, String content) {
    return DigestUtils.md5Hex((password + extno + phone + content).getBytes(StandardCharsets.UTF_8)).toUpperCase();
}

    private String encryptAES(String phone) {
        try {
            // 创建AES加密器
            Cipher cipher = Cipher.getInstance(TRANSFORMATION);

            // 加载密钥
            SecretKeySpec secretKeySpec = new SecretKeySpec(
                mobileEencryptionMode.getBytes(StandardCharsets.UTF_8), ALGORITHM);

            cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);

            // 加密字节数组
            byte[] encryptedBytes = cipher.doFinal(phone.getBytes(StandardCharsets.UTF_8));

            // 将密文转换为 Base64 编码字符串
            return Base64.getEncoder().encodeToString(encryptedBytes);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
```

