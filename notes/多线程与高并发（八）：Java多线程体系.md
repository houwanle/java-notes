## 多线程与高并发（八）：Java多线程体系

### 1.线程的创建

#### 1.1 继承Thread
1. 定义Thread类的子类，并重写该类的run方法，该方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体；
2. 创建Thread子类的实例，即创建了线程对象；
3. 调用线程对象的start()方法来启动线程；

  ```java
  public class DemoThread extends Thread {
    public void run() {
      System.out.println("hello");
    }

    public static void main(String[] args) {
      new DemoThread().start();
    }
  }
  ```
