# JAVA
>* java是一种高级编程语言，由Sun Microsystems公司于1995年推出。Java的设计目标之一是实现“一次编写，到处运行”的能力，即编写的代码可以在任何支持Java的平台上运行，而无需进行额外的修改。Java广泛应用于企业级应用、桌面应用、移动应用和Web开发等领域。
* 主要应用场景：
  1. 爬虫
  2. web服务
  3. `Android` 应用
## Java 优缺点

* 优点：
  * 跨平台性：通过在不同平台安装JVM，一套代码在不同操作系统上运行。
  * 面向对象：Java是一种面向对象编程的高级编程语言。
  * 内存管理：Java有严格的访问控制和内存管理机制，可以防止内存泄漏和程序崩溃，并提供安全性相关的特性，如安全沙箱机制和自动内存管理。
  * 多线程支持
  * 丰富的开发工具支持：Java拥有各种开发工具和框架，如集成开发环境（IDE）、调试器和性能分析工具，可以提高开发效率和代码质量。
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
    *   1byte(字节) = 8bit
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
x.getClass().equals(Base.class) true
x.getClass().equals(Derived.class) false

// 父类class typeinfo.Derived
x instanceof Base true
x instanceof Derived true
Base.isInstance(x) true
Derived.isInstance(x) true
x.getClass() == Base.class false
x.getClass() == Derived.class true
x.getClass().equals(Base.class) false
x.getClass().equals(Derived.class) true
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

## 文件流
### IO
>* 阻塞IO：在进行读取或写入操作时会阻塞线程。

* InputStream

|   类  |   功能    |   构造器参数  |   如何使用    |
| :--:  |   :--:    |   :--:    |   :--:    |
|   `ByteArrayInputStream`    |   允许将内存的缓冲区当做 `InputStream` 使用  |  `byte[]` |   作为一种数据源：与 `FilterInputStream` 对象相连以提供有用接口 |
|   `StringBufferInputStream` `已弃用` |	将 String 转换成 `InputStream`    |   字符串。底层实现实际使用 `StringBuffer`	作为一种数据源；  |将其与 `FilterInputStream` 对象相连以提供有用接口 |
`FileInputStream` |   用于从文件中读取信息    |	字符串，表示文件名、文件或 `FileDescriptor` 对象	|   作为一种数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
|   `PipedInputStream`	|   产生用于写入相关 `PipedOutputStream` 的数据。实现“管道化”概念	|`PipedOutputSteam`	|   作为多线程中的数据源：将其与 `FilterInputStream` 对象相连以提供有用接口
`SequenceInputStream`	|   将两个或多个 `InputStream` 对象转换成一个 `InputStream`	   | 两个 `InputStream` 对象或一个容纳 `InputStream` 对象的容器 `Enumeration`	作为一种数据源：    |   将其与 `FilterInputStream` 对象相连以提供有用接口   |
| `FilterInputStream`	|   抽象类，作为“装饰器”的接口。其中，“装饰器”为其它的 InputStream 类提供有用的功能。| | |

  * FilterInputStream

    |   类  |   功能	|   构造器参数	|   如何使用    |
    |   :--:    |   :---    |   :---    |   :---    |
    |   `DataInputStream` |   与 `DataOutputStream` 搭配使用，按照移植方式从流读取基本数据类型（int、char、long 等）	|   `InputStream` |   包含用于读取基本数据类型的全部接口  |
    |   `BufferedInputStream`	|   使用它可以防止每次读取时都得进行实际写操作。代表“使用缓冲区”    |	InputStream，可以指定缓冲区大小（可选）	|   本质上不提供接口，只是向进程添加缓冲功能。与接口对象搭配  |
    |   `LineNumberInputStream` |	跟踪输入流中的行号，可调用 `getLineNumber()` 和 `setLineNumber(int)`    |   `InputStream`   |   	仅增加了行号，因此可能要与接口对象搭配使用  |
    |   `PushbackInputStream`   |	具有能弹出一个字节的缓冲区，因此可以将读到的最后一个字符回退	|   `InputStream`   |   通常作为编译器的扫描器，我们可能永远也不会用到  |

* OutputStream

|   类  |	功能    |   构造器参数	|   如何使用    |
|   :--:    |   :---    |   :---    |   :---    |
|   `ByteArrayOutputStream` |	在内存中创建缓冲区。所有送往“流”的数据都要放置在此缓冲区	|   缓冲区初始大小（可选）	|   用于指定数据的目的地：将其与 FilterOutputStream 对象相连以提供有用接口  |
|   `FileOutputStream`  |	用于将信息写入文件	|   字符串，表示文件名、文件或 FileDescriptor 对象	|   用于指定数据的目的地：将其与 FilterOutputStream 对象相连以提供有用接口  |
|   `PipedOutputStream` |	任何写入其中的信息都会自动作为相关 `PipedInputStream` 的输出。实现“管道化”概念  |	`PipedInputStream`  |	指定用于多线程的数据的目的地：将其与 `FilterOutputStream` 对象相连以提供有用接口  |
|   `FilterOutputStream`    |	抽象类，作为“装饰器”的接口。其中，“装饰器”为其它 OutputStream 提供有用功能。    |   |   |

  * FilterOutputStream
    | 类  |	功能    |	构造器参数  |	如何使用    |
    | :--:    |   :---    |   :---    |   :---    |
    | `DataOutputStream`  |	与 DataInputStream 搭配使用，因此可以按照移植方式向流中写入基本数据类型（int、char、long 等）	|   `OutputStream`  |	包含用于写入基本数据类型的全部接口  |
    |   `PrintStream`   |	用于产生格式化输出。其中 `DataOutputStream` 处理数据的存储，`PrintStream` 处理显示	   |   `OutputStream`，可以用 boolean 值指示是否每次换行时清空缓冲区（可选）	|   应该是对 `OutputStream` 对象的 final 封装。可能会经常用到它   |
    |   `BufferedOutputStream`	|   使用它以避免每次发送数据时都进行实际的写操作。代表“使用缓冲区”。可以调用 flush() 清空缓冲区	|   `OutputStream`，可以指定缓冲区大小（可选）	|   本质上并不提供接口，只是向进程添加缓冲功能。与接口对象搭配  |

* 标准IO流
>* 用途: 处理标准输入输出（键盘输入和屏幕输出）。
  * 关键类:
    1. System.in (标准输入流)
    2. System.out (标准输出流)
    3. System.err (标准错误输出流)

* RandomAccessFile
>* 随机读写流，独立于 `InputStream` 和 `OutputStream`。
  * 示例用法
    ```java
    import java.io.*;

    public class UsingRandomAccessFile {
        static String file = "rtest.dat";
        // 读取文件
        public static void display() {
            try (
                RandomAccessFile rf =
                    new RandomAccessFile(file, "r")
            ) {
                for (int i = 0; i < 7; i++)
                    System.out.println(
                        "Value " + i + ": " + rf.readDouble());
                System.out.println(rf.readUTF());
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }

        public static void main(String[] args) {
            try (
                // 写入文件
                RandomAccessFile rf =
                    new RandomAccessFile(file, "rw")
            ) {
                for (int i = 0; i < 7; i++)
                    rf.writeDouble(i * 1.414);
                rf.writeUTF("The end of the file");
                rf.close();
                display();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
            try (
                RandomAccessFile rf =
                    new RandomAccessFile(file, "rw")
            ) {
                // 步过 5 * 8 字节 一个Double值为8字节
                rf.seek(5 * 8);
                // 覆盖第个6个Double值，第一个Double值地址是0
                rf.writeDouble(47.0001);
                rf.close();
                display();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
    }

    ```
* Object Stream 对象流
>* 用于对象序列化、反序列化。
  * 关键类: `ObjectInputStream` 和 `ObjectOutputStream`
  * 示例用法
    ```java
    // 序列化对象到文件
    try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("object.dat"))) {
        oos.writeObject(new String("Hello, World!"));
    } catch (IOException e) {
        e.printStackTrace();
    }

    // 从文件反序列化对象
    try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("object.dat"))) {
        String message = (String) ois.readObject();
        System.out.println(message);
    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
    }
    ```


### NIO
>* 非阻塞IO：读写操作在准备好后立刻返回，而无需等待执行结束，不会阻塞线程。
>* 面向缓冲区：数据以（Buffer）的形式处理，每次处理一个缓冲区的数据。
>* 多路复用：通过选择器 (Selector) 机制，可以管理多个通道的 IO 事件。
>* 更高性能：适合高并发、高性能的网络应用。

* 关键类:
    1. FileChannel (文件通道)
    2. ByteBuffer (字节缓冲区)
    3. Selectors 和 SelectionKey (选择器和选择键)

* 常用通道 (Channels)
>* 通道是双向的，可以用来读、写或同时进行读写。
  1. FileChannel
  2. DatagramChannel
  3. SocketChannel
  4. ServerSocketChannel

* 缓冲区 (Buffers)
>* 缓冲区是一个容器对象，用于存储从通道读取的数据或即将写入通道的数据。
  * 常用缓冲区
    1. ByteBuffer
    2. CharBuffer
    3. IntBuffer
    4. FloatBuffer
  * 缓冲区主要属性：
    1. capacity：缓冲区的容量，缓冲区中能容纳的数据元素的最大数量。
    2. limit：第一个不能被读或写的元素的索引，缓冲区的界限。
    3. position：缓存区指针，指向下一个将要被读取的数据。
    4. mark：在缓存区标记一个位置。

* 选择器 (Selectors)
>* 选择器用于实现非阻塞 I/O。选择器会注册一个或多个通道，并可以监控这些通道的多个事件。
  1. Selector 类是选择器的核心。
  2. SelectionKey 用于标识哪些通道已经准备好进行 I/O 操作。

* 读写文件示例
  ```java
    import java.io.IOException;
    import java.nio.ByteBuffer;
    import java.nio.channels.FileChannel;
    import java.nio.file.Path;
    import java.nio.file.Paths;
    import java.nio.file.StandardOpenOption;

    public class FileReadExample {
        public static void main(String[] args) {
            Path path = Paths.get("example.txt");

            // 读取文件
            try (FileChannel fileChannel = FileChannel.open(path, StandardOpenOption.READ)) {
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                while (fileChannel.read(buffer) > 0) {
                    buffer.flip(); // Switch buffer from writing mode to reading mode
                    while (buffer.hasRemaining()) {
                        System.out.print((char) buffer.get());
                    }
                    buffer.clear(); // Clear buffer for the next read
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            //写入文件
            try (FileChannel fileChannel = FileChannel.open(path, StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                buffer.put("Hello, World!".getBytes());
                buffer.flip(); // Switch buffer from writing mode to reading mode
                fileChannel.write(buffer);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
  ```
* 网络I/O示例（服务器端）阻塞式
  ```java
    import java.io.IOException;
    import java.net.InetSocketAddress;
    import java.nio.ByteBuffer;
    import java.nio.channels.ServerSocketChannel;
    import java.nio.channels.SocketChannel;

    public class NioServer {
        public static void main(String[] args) {
            try (ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()) {
                serverSocketChannel.bind(new InetSocketAddress(8080));
                while (true) {
                    // serverSocketChannel.accept() 是一个阻塞操作，等待客户端连接。一旦接收到连接请求，它会返回一个 SocketChannel 用于与客户端通信。
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    int bytesRead = socketChannel.read(buffer);
                    while (bytesRead != -1) {
                        //将缓冲区从写模式切换到读模式 buffer.flip()，重置position到流启示点供后续读取。
                        buffer.flip();
                        while (buffer.hasRemaining()) {
                            System.out.print((char) buffer.get());
                        }
                        buffer.clear();
                        bytesRead = socketChannel.read(buffer);
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
  ```
* 网络I/O示例（客户端）
  ```java
    import java.io.IOException;
    import java.net.InetSocketAddress;
    import java.nio.ByteBuffer;
    import java.nio.channels.SocketChannel;

    public class NioClient {
        public static void main(String[] args) {
            // 开启通道用于与服务器通信
            try (SocketChannel socketChannel = SocketChannel.open()) {
                // 链接 目标地址
                socketChannel.connect(new InetSocketAddress("localhost", 8080));
                // 初始化缓冲区，将数据放入缓冲区
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                buffer.put("Hello, NIO!".getBytes());
                // 重置指针 写入到通道
                buffer.flip();
                socketChannel.write(buffer);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
  ```
* 使用选择器进行非阻塞网络I/O（服务器端）
  ```java
    import java.io.IOException;
    import java.net.InetSocketAddress;
    import java.nio.ByteBuffer;
    import java.nio.channels.SelectionKey;
    import java.nio.channels.Selector;
    import java.nio.channels.ServerSocketChannel;
    import java.nio.channels.SocketChannel;
    import java.util.Iterator;

    public class NioServer {
        public static void main(String[] args) throws IOException {
            // 打开选择器
            Selector selector = Selector.open();

            // 打开服务器通道
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.bind(new InetSocketAddress(8080));
            serverSocketChannel.configureBlocking(false);

            // 将通道注册到选择器上，并注册接收事件
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

            while (true) {
                // 选择键
                selector.select();

                // 获取选择键集合
                Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();

                while (keyIterator.hasNext()) {
                    SelectionKey key = keyIterator.next();

                    if (key.isAcceptable()) {
                        // 接受客户端连接 并将当前连接注册到 选择器
                        ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel();
                        SocketChannel socketChannel = serverChannel.accept();
                        socketChannel.configureBlocking(false);
                        socketChannel.register(selector, SelectionKey.OP_READ);
                    } else if (key.isReadable()) {
                        // 读取客户端数据
                        SocketChannel socketChannel = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(256);
                        int bytesRead = socketChannel.read(buffer);
                        if (bytesRead == -1) {
                            socketChannel.close();
                        } else {
                            System.out.println("Received: " + new String(buffer.array()).trim());
                        }
                    }

                    // 移除处理过的键
                    keyIterator.remove();
                }
            }
        }
    }
  ```

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

### volatile
> 变量修饰符，使变量在多线程可见

* volatile 在java内存模型`jmm` 中封装的8个交互操作。
  * read：把主内存中的值传递到工作内存，为load动作做准备。
  * load：将read动作从主内存传递的内存变量赋值为工作区内存的变量副本。
  * use：将工作内存中的变量传递给执行引擎。当虚拟机需要使用变量的字节码指令时执行此操作。
  * assign：将从执行引擎接受到的值赋给工作内存的变量，当虚拟机接收到给变量赋值的字节码指令时执行操作。
  * store：把工作内存中的一个变量的值传送到主内存中，为随后的write操作做准备。
  * write：把store操作从工作内存得到的变量写入到主内存中。
  * lock：作用于主内存的变量，把一个变量标示为一条线程独占的状态。
  * Unlock：作用于主内存，把一个处于锁定状态的变量释放出来。释放后的变量才能够被其他线程锁定。

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
* 基础用法
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
* 同步调用 `thenApply()` 和异步调用 `thenApplyAsync()`
* 异常
  * 任务完成或出现异常都会使CompletableFuture处于完成状态；当出现异常时，CompletableFuture并不会立即跳出异常，而是当你获取get()结果时才会看到抛出异常；
  * 查看`CompletableFuture`状态
  ```java
  // 是否发生异常
  .isCompletedExceptionally();
  // 是否执行完毕
  .isDone();
  ```
  * 链式异常处理
      * `exceptionally()`: 参数仅在出现异常时才运行。exceptionally() 局限性在于，该函数只能返回输入类型相同的值。通过将一个好的对象插入到流中来恢复到一个可行的状态。
      * `handle()`: 一致被调用来查看是否发生异常（必须检查fail是否为true）。但是 handle() 可以生成任何新类型，所以它允许执行处理，而不是像使用 exceptionally()那样简单地恢复。
      * `whenComplete()`: 类似于handle()，同样必须测试它是否失败，但是参数是一个消费者，并且不修改传递给它的结果对象。
        ```java
        CompletableExceptions
        .test("exceptionally", failcount)
        .exceptionally((ex) -> { // Function
            if (ex == null)
                System.out.println("I don't get it yet");
            return new Breakable(ex.getMessage(), 0);
        })
        .thenAccept(str ->
                        System.out.println("result: " + str));

        // Create a new result (recover):
        CompletableExceptions
                .test("handle", failcount)
                .handle((result, fail) -> { // BiFunction
                    if (fail != null)
                        return "Failure recovery object";
                    else
                        return result + " is good";
                })
                .thenAccept(str ->
                        System.out.println("result: " + str));

        // Do something but pass the same result through:
        CompletableExceptions
                .test("whenComplete", failcount)
                .whenComplete((result, fail) -> { // BiConsumer
                    if (fail != null)
                        System.out.println("It failed");
                    else
                        System.out.println(result + " OK");
                })
                .thenAccept(r ->
                        System.out.println("result: " + r));
        ```
  * 流异常
    CompletableFuture 和 parallel Stream 都不支持包含检查性异常的操作。相反，你必须在调用操作时处理检查到的异常：
    ```java
    public class ThrowsChecked {
        class Checked extends Exception {}

        static ThrowsChecked nochecked(ThrowsChecked tc) {
            return tc;
        }

        static ThrowsChecked withchecked(ThrowsChecked tc) throws Checked {
            return tc;
        }

        static void testStream() {
            Stream.of(new ThrowsChecked())
                    .map(ThrowsChecked::nochecked)
                    // .map(ThrowsChecked::withchecked); // [1]
                    .map(
                            tc -> {
                                try {
                                    return withchecked(tc);
                                } catch (Checked e) {
                                    throw new RuntimeException(e);
                                }
                            });
        }

        static void testCompletableFuture() {
            CompletableFuture
                    .completedFuture(new ThrowsChecked())
                    .thenApply(ThrowsChecked::nochecked)
                    // .thenApply(ThrowsChecked::withchecked); // [2]
                    .thenApply(
                            tc -> {
                                try {
                                    return withchecked(tc);
                                } catch (Checked e) {
                                    throw new RuntimeException(e);
                                }
                            });
        }
    }
    ```
### 死锁
> 以下四个条件同时满足时就会触发死锁。
* 互斥条件。任务使用的资源中至少有一个不能共享的。
* 至少有一个任务它必须持有一个资源且正在等待获取一个被当前别的任务持有的资源。
* 资源不能被任务抢占， 任务必须把资源释放当作普通事件。
* 必须有循环等待， 这时，一个任务等待其它任务所持有的资源， 后者又在等待另一个任务所持有的资源， 这样一直下去，知道有一个任务在等待第一个任务所持有的资源， 使得大家都被锁住。

### 线程异常
* 主线程无法捕获子线程内发生的异常,可以通过静态方法设置 `Thread` 异常捕获，也可以单独调用线程示例的setUncaughtExceptionHandler方法设置异常捕获。
```java
  Thread.setDefaultUncaughtExceptionHandler(
      new MyUncaughtExceptionHandler());
```
### volatile
* 使用该关键字标记的变量，线程直接读写主内存，从而使多个线程可见，保证数据一致性。
* 无法保证操作原子性，在并发操作、多线程环境下，需要额外使用 `Synchronized` 上锁避免资源竞争问题。
* 关键字仅能保证变量所有线程可见以及指令重排，。
* 禁止指令重排序：编译器和 `CPU` 不会对 `volatile` 变量的读/写操作进行重排序。
  * 确保被标记 `volatile` 变量前面的写操作，依旧在 `volatile` 变量前运行，不关心前面的写操作是否指令排。`volatile` 变量后的操作同上。
* 字分裂
  * 当 `Java` 数据类型字节数较大时（如 `double`、`long` 64位），JVM允许将64位数量的读写分为两个32位单独操作执行，写入变量的过程将分为两步进行。
  * 在读取变量时存在脏读的情况。（读取时在第一步执行结束，和第二步开始前）。
  * 使用 `volatile` 修饰符可以组织字分裂的情况。
  * 但如果使用 `synchronized` 或 `java.util.concurrent.atomic` 类之一保护这些变量，`volatile` 也可被替代。
* volatile 适用于 仅有一个线程对变量存在写操作，而其他线程都为读取的多线程环境。
* 最后尽量不要使用 `volatile` ，选择使用 `Atomic`。
### Atomic
>* 原子性可以应用于除 `long` 和 `double` 之外的所有基本类型之上的 “简单操作”。(字分裂问题)

### Synchronized
>* 在使用并发时，将字段设为 private 特别重要；否则，synchronized 关键字不能阻止其他任务直接访问字段。

* 资源竞争
  * 一个线程可以获取对象的锁多次。
  * `JVM` 会追踪对象被锁定的次数，若对象未被线程锁定，则计数为 `0` 。当一个线程首次获得锁时，计数为 `1`，每次同一个线程在同一个对象上获取另一个锁时（线程在未释放对象某一方法锁的同时请求对象其他需获取锁的方法时）计数器会增加。只有首先获取到锁的线程才允许多次获取同一对象上的多个锁。当线程离开 `Synchronized` 方法时，计数递减，直到计数器标记为 `0` 。
  * 当某个方法正在读写某一会被其他线程读写的变量时，需要使用相同的监视器锁同步。

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




## 对象序列化
>* 可以对数据对象进行持久化操作。
>* 可以使用 `Web Socket`将数据对象进行网络传输。
>* 序列化和反序列化数据读取顺序必须一致。
>* 静态变量无法序列化。

### Serializable
>* 自动序列化接口。
>* 默认将对像所有属性数据进行序列化操作，可以通过使用 `@transient`注解标注不需要进行序列化的属性。

* @transient
  * @transient修饰的变量不能被序列化；
  * @transient只作用于实现 Serializable 接口；
  * @transient只能用来修饰普通成员变量字段；
  * 不管有没有 @transient 修饰，静态变量都不能被序列化；

### Externalizable
>* 手动序列化接口。
>* 可以将静态变量进行序列化。

```java
public class Blip3 implements Externalizable {
    private int i;
    private String s; // No initialization
    public Blip3() {
        System.out.println("Blip3 Constructor");
// s, i not initialized
    }
    public Blip3(String x, int a) {
        System.out.println("Blip3(String x, int a)");
        s = x;
        i = a;
// s & i initialized only in non-no-arg constructor.
    }
    @Override
    public String toString() { return s + i; }
    @Override
    public void writeExternal(ObjectOutput out)
            throws IOException {
        System.out.println("Blip3.writeExternal");
// You must do this: 手动序列化属性，不关心是否被@transient修饰
        out.writeObject(s);
        out.writeInt(i);
    }
    @Override
    public void readExternal(ObjectInput in)
            throws IOException, ClassNotFoundException {
        System.out.println("Blip3.readExternal");
// You must do this: 读取序列化属性
        s = (String)in.readObject();
        i = in.readInt();
    }
// 文件流读写示例
    public static void main(String[] args) {
        System.out.println("Constructing objects:");
        Blip3 b3 = new Blip3("A String ", 47);
        System.out.println(b3);
        try(
                ObjectOutputStream o = new ObjectOutputStream(
                        new FileOutputStream("Blip3.serialized"))
        ) {
            System.out.println("Saving object:");
            o.writeObject(b3);
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
// Now get it back:
        System.out.println("Recovering b3:");
        try(
                ObjectInputStream in = new ObjectInputStream(
                        new FileInputStream("Blip3.serialized"))
        ) {
            b3 = (Blip3)in.readObject();
        } catch(IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
        System.out.println(b3);
    }
}
```

### 静态变量序列化
* 自动序列化无法对静态变量生效，可以手动对静态变量单独操作进行序列化。
* 静态变量反序列化后，若对应类的静态属性已存在值，则静态反序列化值会覆盖当前JVM中对应的静态变量的值。
* 实现Externalizable接口，并重写接口方法可以实现序列化需要的属性，无论是否被@transient 或者static修饰
* 若一个输入流重复读取一个序列化对象，则多次获取到的对象为同一对象，内存地址相同。若另创建新的流读取，则获取到的对象为新创建的对象，与其他流得到的对象地址不相同。


### XML
> XML标签序列化数据接口
```java
import nu.xom.*;
import java.io.*;
import java.util.*;
public class APerson {
    private String first, last;
    public APerson(String first, String last) {
        this.first = first;
        this.last = last;
    }
    // Produce an XML Element from this APerson object:
    public Element getXML() {
        Element person = new Element("person");
        Element firstName = new Element("first");
        firstName.appendChild(first);
        Element lastName = new Element("last");
        lastName.appendChild(last);
        person.appendChild(firstName);
        person.appendChild(lastName);
        return person;
    }
    // Constructor restores a APerson from XML:
    public APerson(Element person) {
        first = person
                .getFirstChildElement("first").getValue();
        last = person
                .getFirstChildElement("last").getValue();
    }
    @Override
    public String toString() {
        return first + " " + last;
    }
    // Make it human-readable:
    public static void
    format(OutputStream os, Document doc)
            throws Exception {
        Serializer serializer =
                new Serializer(os,"ISO-8859-1");
        serializer.setIndent(4);
        serializer.setMaxLength(60);
        serializer.write(doc);
        serializer.flush();
    }
    
    // 写入XML对象
    public static void main(String[] args) throws Exception {
        List<APerson> people = Arrays.asList(
                new APerson("Dr. Bunsen", "Honeydew"),
                new APerson("Gonzo", "The Great"),
                new APerson("Phillip J.", "Fry"));
        System.out.println(people);
        Element root = new Element("people");
        for(APerson p : people)
            root.appendChild(p.getXML());
        Document doc = new Document(root);
        format(System.out, doc);
        format(new BufferedOutputStream(
                new FileOutputStream("People.xml")), doc);



        // 读取 XML对象
        public List readPeople(String fileName) throws Exception {
                List list = new ArrayList();
            Document doc =
                    new Builder().build(new File(fileName));
            Elements elements =
                    doc.getRootElement().getChildElements();
            for(int i = 0; i < elements.size(); i++)
                list.add(new APerson(elements.get(i)));
            return list;
        }

    }
}
```

## 数据压缩
>* Java I/O 类库提供了可以读写压缩格式流的类。你可以将其他 I/O 类包装起来用于提供压缩功能。
>* 这些类不是从 Reader 和 Writer 类派生的，而是 InputStream 和 OutputStream 层级结构的一部分。这是由于压缩库处理的是字节，而不是字符。
>* 但是，你可能会被迫混合使用两种类型的流（请记住，你可以使用 InputStreamReader 和 OutputStreamWriter，这两个类可以在字节类型和字符类型之间轻松转换）。

* `ZIP` 和 `GZIP` 使用较多
|   压缩类  |	功能    |
|   :--:    |   :---    |
|   CheckedInputStream	|    getCheckSum() 可以对任意 InputStream 计算校验和（而不只是解压）    |
|   CheckedOutputStream |   getCheckSum() 可以对任意 OutputStream 计算校验和（而不只是压缩）    |
|   DeflaterOutputStream    |   压缩类的基类    |
|   ZipOutputStream	|   DeflaterOutputStream 类的一种，用于压缩数据到 Zip 文件结构  |
|   GZIPOutputStream	|   DeflaterOutputStream 类的一种，用于压缩数据到 GZIP 文件结构 |
|   InflaterInputStream	|   解压类的基类    |
|   ZipInputStream	|   InflaterInputStream 类的一种，用于解压 Zip 文件结构的数据   |
|   GZIPInputStream	|   InflaterInputStream 类的一种，用于解压 GZIP 文件结构的数据  |

* GZIP解压缩单文件简单示例
```java
public class GZIPcompress {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println(
                    "Usage: \nGZIPcompress file\n" +
                            "\tUses GZIP compression to compress " +
                            "the file to test.gz");
            System.exit(1);
        }
        try (
                // 获取输入文件流
                InputStream in = new BufferedInputStream(
                        new FileInputStream(args[0]));
                // 获取输出文件流并使用GZIP包装
                BufferedOutputStream out =
                        new BufferedOutputStream(
                                new GZIPOutputStream(
                                        new FileOutputStream("test.gz")))
        ) {
            System.out.println("Writing file");
            int c;
            while ((c = in.read()) != -1)
                out.write(c);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        System.out.println("Reading file");
        try (
            // 解压缩 读取
                BufferedReader in2 = new BufferedReader(
                        new InputStreamReader(new GZIPInputStream(
                                new FileInputStream("test.gz"))))
        ) {
            in2.lines().forEach(System.out::println);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}

```
* ZIP多文件存储
```java
import java.io.*;
import java.util.*;
import java.util.zip.*;

public class ZipCompress {
    public static void main(String[] args) {
        // 检查是否提供了命令行参数，如果没有提供则输出用法并退出
        if (args.length == 0) {
            System.out.println("Usage: java ZipCompress file1 file2 ...");
            System.exit(1);
        }

        // 压缩文件部分
        try (
            // 创建文件输出流，指向压缩后的文件
            FileOutputStream f = new FileOutputStream("test.zip");
            // 创建一个带校验和的输出流，使用Adler32算法
            CheckedOutputStream csum = new CheckedOutputStream(f, new Adler32());
            // 创建ZIP输出流
            ZipOutputStream zos = new ZipOutputStream(csum);
            // 创建缓冲输出流包装ZIP输出流，提高写入效率
            BufferedOutputStream out = new BufferedOutputStream(zos)
        ) {
            // 设置ZIP文件的注释
            zos.setComment("A test of Java Zipping");
            // 遍历所有的输入文件
            for (String arg : args) {
                System.out.println("Writing file " + arg);
                // 打开输入文件流
                try (
                    InputStream in = new BufferedInputStream(new FileInputStream(arg))
                ) {
                    // 创建一个新的ZIP条目，并放入ZIP输出流中
                    zos.putNextEntry(new ZipEntry(arg));
                    int c;
                    // 读取输入文件的内容，并写入到ZIP输出流中
                    while ((c = in.read()) != -1)
                        out.write(c);
                }
                // 刷新缓冲输出流，确保所有数据都被写入
                out.flush();
            }
            // 打印校验和，校验和在文件关闭后才有效
            System.out.println("Checksum: " + csum.getChecksum().getValue());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        // 解压文件部分
        System.out.println("Reading file");
        try (
            // 创建文件输入流，指向压缩文件
            FileInputStream fi = new FileInputStream("test.zip");
            // 创建一个带校验和的输入流，使用Adler32算法
            CheckedInputStream csumi = new CheckedInputStream(fi, new Adler32());
            // 创建ZIP输入流
            ZipInputStream in2 = new ZipInputStream(csumi);
            // 创建缓冲输入流包装ZIP输入流，提高读取效率
            BufferedInputStream bis = new BufferedInputStream(in2)
        ) {
            ZipEntry ze;
            // 遍历ZIP文件中的所有条目
            while ((ze = in2.getNextEntry()) != null) {
                System.out.println("Reading file " + ze);
                int x;
                // 读取ZIP条目的内容，并输出到控制台
                while ((x = bis.read()) != -1)
                    System.out.write(x);
            }
            // 如果只有一个文件，打印其校验和
            if (args.length == 1)
                System.out.println("Checksum: " + csumi.getChecksum().getValue());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        // 另一种打开和读取ZIP文件的方式
        try (
            // 创建一个ZipFile对象，用于读取ZIP文件
            ZipFile zf = new ZipFile("test.zip")
        ) {
            // 获取ZIP文件中的所有条目
            Enumeration<? extends ZipEntry> e = zf.entries();
            while (e.hasMoreElements()) {
                ZipEntry ze2 = e.nextElement();
                System.out.println("File: " + ze2);
                // 可以像之前那样提取数据
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}

```


### Java的Jar包
>* `Zip` 格式也用于 JAR（Java ARchive）文件格式，这是一种将一组文件收集到单个压缩文件中的方法，就像 `Zip` 一样。与 `Java` 中的其他所有内容一样，`JAR` 文件是跨平台的，因此你不必担心平台问题。你还可以将音频和图像文件像类文件一样包含在其中。
>* jar 工具不像 Zip 实用程序那样通用。例如，你无法将文件添加或更新到现有 JAR 文件；只能从头开始创建 JAR 文件。
>* 此外，你无法将文件移动到 JAR 文件中，在移动文件时将其删除。

* JDK 附带的 jar 工具会自动压缩你选择的文件。你可以在命令行上调用它：
```java
jar [options] destination [manifest] inputfile(s)
```

* `java` 关于jar的相关指令

|   选项	|   功能    |
|   :--:    |   :---    |
|   c	    |   创建一个新的或者空的归档文件    |
|   t   |	列出内容目录    |
|   x   |	提取所有文件    |
|   x   | file	提取指定的文件  |
|   f   |	这代表着，“传递文件的名称。”如果你不使用它，jar 假定它的输入将来自标准输入，或者，如果它正在创建一个文件，它的输出将转到标准输出。  |
|   m   |	代表第一个参数是用户创建的清单文件的名称。  |
|   v   |	生成详细的输出用于表述 jar 所作的事情   |
|   0   |	仅存储文件;不压缩文件（用于创建放在类路径中的 JAR 文件）。  |
|   M   |	不要自动创建清单文件    |

* 示例java代码
命令创建名为 myJarFile 的 JAR 文件。 jar 包含当前目录中的所有类文件，以及自动生成的清单文件：
```java
jar cf myJarFile.jar *.class
```

下一个命令与前面的示例类似，但它添加了一个名为 myManifestFile.mf 的用户创建的清单文件。 ：
```java
jar cmf myJarFile.jar myManifestFile.mf *.class
```

这个命令输出了 myJarFile.jar 中的文件目录：
```java
jar tf myJarFile.jar
```

如下添加了一个“verbose”的标志，用于生成更多关于 myJarFile.jar 中文件的详细信息：
```java
jar tvf myJarFile.jar
```

假设 audio，classes 和 image 都是子目录，它将所有子目录组合到文件 myApp.jar 中。还包括“verbose”标志，以便在 jar 程序工作时提供额外的反馈：
```java
jar cvf myApp.jar audio classes image
```

如果你在创建 JAR 文件时使用了 0（零） 选项，该文件将会被替换在你的类路径（CLASSPATH）中：然后 Java 可以搜索到 lib1.jar 和 lib2.jar 的类文件。
```java
CLASSPATH="lib1.jar;lib2.jar;"
```
