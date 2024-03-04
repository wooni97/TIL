# Thread와 Runnable

자바 애플리케이션을 만들어 실행하면 1개의 메인 스레드에 의해 프로그램이 실행된다. 그러나 한개의 스레드 만으로는 동시에 여러 작업을 실행할 수 없다. 동시에 여러 작업을 처리하고 싶다면 별도의 스레드를 만들어 실행해야 한다.

스레드를 생성하는 방법은 두가지가 존재한다.

- Thread 클래스를 상속받아 run 메소드를 오버라이딩 하는 것
- Runnable 인터페이스를 implements 하여 run 메소드를 정의하는 것입니다.

**Thread와 Runnable은 멀티 스레드를 위해 제공된 기술이다.**

### ☑️ Thread 클래스

스레드 생성을 위한 자바의 클래스.

**Thread 클래스로 스레드 구현**

Thread 클래스로 스레드를 구현하려면 이를 상속받는 클래스를 만들고, 내부에서 run 메소드를 구현해야 한다. 그리고 Thread의 start 메소드를 호출하면 run 메소드가 실행된다.

```java
@Test
void threadStart() {
    Thread thread = new MyThread();

    thread.start();
    System.out.println("Hello: " + Thread.currentThread().getName());
}

static class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread: " + Thread.currentThread().getName());
    }
}
```

**run 메소드를 직접 호출하는것이 아니라 start를 호출.**

### ☑️ Thread의 start(), run() 메소드

**둘의 차이는?**

- run 메소드를 호출한다면 메인 스레드에서 객체의 메소드를 호출하는 것에 불과.
- start() 메서드를 호출하는 것은 새로운 스레드가 작업을 실행하는데 필요한 호출 스택을 생성한 다음에 run을 호출해서 호출스택에 run이 올라가게 하는 것이다.

**Thread.run()**

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0747e310-4032-4588-a306-7faef82d958b/ad2b1fc0-244a-4c7d-8fa1-896adf35c895/Untitled.png)

**Thread.start()**

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0747e310-4032-4588-a306-7faef82d958b/0f845bb7-6fc8-4fb1-a7ba-d7eb802b962d/Untitled.png)

`start()` 메서드를 실행하면 Thread의 상태를 체크한다. 상태가 0이 아니라면 `IllegalThreadStateException` 예외를 발생시킨다. 그리고 native 메서드인 `start0()`을 호출한다.

native 메서드 start0() 메서드를 호출하면 JVM_StartThread라는 C++ 함수가 실행된다.

JVM_StartThread 구현 코드 요약

- 입력받은 Thread 인스턴스가 null이 아니면(이미 실행되었던 Thread 일 경우) 예외를 발생시킨다.
- Call Stack 사이즈를 계산하여 Thread를 생성하고 start한다.

**Thread 클래스의 메소드**

- sleep
    - 현재 스레드 멈추기
    - 자원을 놓아주지는 않고, 제어권을 넘겨주므로 데드락이 발생할 수 있음
- interrupt
    - 다른 스레드를 깨워서 interruptedException을 발생시킴
    - Interrupt가 발생한 스레드는 예외를 catch 하여 다른 작업을 할 수 있음
- join
    - 다른 스레드의 작업이 끝날 때 까지 기다리게 함
    - 스레드의 순서를 제어할 때 사용

**start 메소드**

- 스레드가 실행 가능한지 검사함

  스레드는 5가지 상태가 있음

  ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/666b82f7-3f39-45b3-8735-4902c1c8213f/Untitled.png)

    - New
    - Runnable
    - Waiting
    - Timed Waiting
    - Terminated

  start는 가장 처음에 해당 스레드가 실행 가능한지 상태(0)인지 확인한다. 만약 스레드가 New 상태가 아니라면 IllegalThreadStateException 예외를 발생시킨다.

- 스레드를 스레드 그룹에 추가함

  스레드 그룹에 해당 스레드를 추가. 스레드 그룹에 해당 스레드를 추가하면 스레드 그룹에 실행 준비된 스레드가 있음을 알려주고, 관련 작업들이 내부적으로 진행

  (스레드 그룹 : 서로 관련있는 스레드를 하나의 그룹으로 묶어 다루기 위한 장치, 자바에서는 ThreadGroup 클래스를 제공)

- 스레드를 JVM이 실행시킴

  start0 메소드 호출. 이것은 native 메소드로 선언. JVM에 의해 호출되는데 내부적으로 run을 호출. 그리고 스레드의 상태가 Runnable로 바뀜. start는 여려번 호출하는 것이 불가능하고 1번만 가능.


```java
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
        
        }
    }
}
```

### ☑️ **Thread 클래스의 `public static native Thread currentThread()`**

메서드는 현재 실행 중인 스레드를 반환하는 **`Thread`** 클래스의 정적 메서드이다. 현재 실행 중인 코드를 실행하고 있는 스레드에 대한 참조를 반환.

**`currentThread()` 메서드가 `static`으로 선언된 이유**

1. **전역적인 접근**: 모든 스레드에서 현재 스레드를 가져와야 할 때에는 인스턴스를 생성하지 않고도 접근할 수 있어야 한다. **`static`**으로 선언함으로써 객체 인스턴스를 생성하지 않고도 메서드에 접근할 수 있다.
2. **스레드 독립성 보장**: **`currentThread()`** 메서드는 현재 실행 중인 스레드에 대한 정보를 제공하기 때문에, 이 메서드를 호출하는 스레드와 실제 반환되는 스레드가 서로 다를 수 있습니다. 따라서 인스턴스 레벨이 아닌 클래스 레벨에서 접근할 수 있도록 **`static`**으로 선언된다.

### ☑️  Runnable 인터페이스

1개의 메소드만을 갖는 함수형 인터페이스

```java
@FunctionalInterface
public interface Runnable {

    public abstract void run();
    
}
```

스레드를 구현하기 위한 템플릿이다. 해당 인터페이스의 구현체를 만들고 Thread 객체 생성시에 넘겨주면 실행 가능하다.

```java
import java.util.Random;

public class MyRunnable implements Runnable {

  private static final Random random = new Random();

  @Override
  public void run() {
    String threadName = Thread.currentThread().getName();
    System.out.println("- " + threadName + " has been started");
    int delay = 1000 + random.nextInt(4000);
    try {
      Thread.sleep(delay);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.println("- " + threadName + " has been ended (" + delay + "ms)");
  }

}

```

### ☑️ Thread vs Runnable

2가지 방법으로 작성한 스레드의 실행 방법은 약간 다르다.

두가지 방법 모두 Thread 클래스의 start() 메소드를 통해서 실행시킬 수 있는데, Thread를 확장하여 구현한 경우 해당 객체의 start() 메소드를 직접 호출할 수 있다. 반면에 Runnable을 구현한 경우 Runnable 형 인자를 받는 생성자를 통해 별도의 Thread 객체를 생성 후 start() 메소드를 호출해야 한다.

```java
public class ThreadRunner {

  public static void main(String[] args) {
    // create thread objects
    Thread thread1 = new MyThread();
    thread1.setName("Thread #1");
    Thread thread2 = new MyThread();
    thread2.setName("Thread #2");

    // create runnable objects
    Runnable runnable1 = new MyRunnable();
    Runnable runnable2 = new MyRunnable();

    Thread thread3 = new Thread(runnable1);
    thread3.setName("Thread #3");
    Thread thread4 = new Thread(runnable2);
    thread4.setName("Thread #4");

    // start all threads
    thread1.start();
    thread2.start();
    thread3.start();
    thread4.start();
  }

}
```

Thread와 Runnable 둘 중 무엇이 더 번거로운지는 개인차가 있는것 같다.

**장/단점**

- 자바에서는 다중 상속을 허용하지 않기 때문에, Thread 클래스를 확장하는 클래스는 다른 클래스를 상속 받을 수 없다.
- 클래스의 확장성이 중요한 상황이라면 Runnable 인터페이스를 구현하는 것이 더 바람직할 것이다.
- Thread 관련 기능의 확장이 필요한 경우에는 Thread 클래스를 상속받아 구현해야 할 때도 있다. 하지만 거의 대부분의 경우에는 Runnable 인터페이스를 사용하면 해결 가능하다.

### ☑️ 새로 생성된 스레드의 호출 스택은 언제 사라질까?

새로 생성된 스레드의 호출 스택은 해당 스레드가 실행을 마치고 종료될 때 함께 사라진다. 스레드가 작업으 ㄹ오나료하거나 run() 메서드의 실행이 종료되면 해당 스레드는 종료된다. 이후에는 해당 스레드의 호출 스택도 메모리에서 해제 된다.