# Java 多线程




* 对于同一个Runnable实例启动的多个线程：
  * 共享的资源（线程不安全）：
    * 该Runnable对象的所有成员变量（实例变量）
    * 该对象引用的其他共享对象（如集合、文件流等外部资源）
    * 类的静态变量（属于类级别，所有实例共享）
  
  * 线程私有的资源（线程安全）：
    * 方法内的局部变量（包括参数变量）
    * 用ThreadLocal修饰的变量
    * 每个线程独立的调用栈、程序计数器等运行时数据

  * Thread类自身的成员变量（如线程名name等）

* 关键结论：
  * 成员变量共享：只要多个线程操作同一个对象实例，其成员变量就是共享的

  * 执行过程独立：每个线程会独立执行run()方法，拥有自己的方法调用栈和局部变量

  * 静态变量特殊：即使是不同Runnable实例，静态变量也是全局共享的

```java
// 线程不安全示例
public class Demo7 {
    public static void main(String[] args) {
        //线程不安全
        Runnable run = new Ticket();
        new Thread(run).start();
        new Thread(run).start();
        new Thread(run).start();
    }
     static class Ticket implements Runnable{
        //总票数
        private int count = 5;
        @Override
        public void run() {
            while (count>0){
                //卖票
                System.out.println("正在准备卖票");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                count--;
                System.out.println("卖票结束,余票："+count);
            }
        }
    }
}

```