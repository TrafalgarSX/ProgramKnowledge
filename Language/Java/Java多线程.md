### Defining and Starting a Thread
- Provide a Runnable object. The Runnable interface defines a single method, run, meant to contain the code executed in the thread. The Runnable object is passed to the Thread constructor, as in the HelloRunnable example:
```java
public class HelloRunnable implements Runnable {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new Thread(new HelloRunnable())).start();
    }

}
```
- Subclass Thread. The Thread class itself implements Runnable, though its run method does nothing. An application can subclass Thread, providing its own implementation of run, as in the HelloThread example
```java
public class HelloThread extends Thread {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new HelloThread()).start();
    }

}
```
some simplify:
```java
    Thread t1 = new Thread(new Runnable() {
    public void run()
    {
         // code goes here.
    }});  
    t1.start();
```

```java
new Thread(){
          public void run() {
              try{
                  System.out.println("Does it work?");
                  Thread.sleep(1000);
                  System.out.println("Nope, it doesnt...again.");
              } catch(InterruptedException v)
              {
              System.out.println(v);
              }
          }
      }.start();
```

```java
new Thread(() ->{
        System.out.println("Does it work?");
        Thread.sleep(1000);
        System.out.println("Nope, it doesnt...again.");       
}){{start();}}.join();
```

thread-pool
```java
 public static void main(String[] args) {
        int threadCount = 100;

        // create a thread-pool excute test
        ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
        for (int i = 0; i < threadCount; i++) {
            executorService.execute(new TestRunnable());
        }
        executorService.shutdown();
    }

    // 实现Runnable接口，用于执行Test函数
    static class TestRunnable implements Runnable {
        @Override
        public void run() {
            SM9Test sm9Test = new SM9Test();
            sm9Test.Test();
        }
    }
```