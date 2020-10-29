<h1>스레드 생성 방법</h1>

<h2>java.lang.Thread 클래스로 생성</h2>

Runnable을 매개값으로 갖는 생성자를 호출한다.

~~~
Thead thread = new Thread(Runnable target);
~~~

<strong>Runnable:</strong> 작업 스레드가 실행할 수 있는 코드를 가지고 있는 객체라는 의미이다. 작업 내용을 가지고 있는 객체이며 실제 스레드는 아니다.

~~~
1. Runnable 구현 객체를 생성한다.
class Task implements Runnable {
    public void run() {
        //스레드가 실행할 코드
    }
}

2. Runnable 객체를 구현한 클래스를 매개값으로 해서 Thread 생성자를 호출하면 작업 스레드가 생성된다.
Thread thread = new Thread(task);

3. 작업 스레드는 생성 즉시 실행되는 것이 아니라, start() 메소드를 다음과 같이 호출해야 실행된다.
thread.start();
~~~

<h3>비프음 발생과 프린팅을 메인스레드에서만 하는 경우</h3>

~~~
public class BeepPrintExample1 {
    public static void main(String[] args) {
        Toolkit toolkit = Toolkit.getDefaultToolkit();
        for (int i = 0; i < 5; ++i) {
            toolkit.beep();
            try {
                Thread.sleep(500);
            } catch (Exception e) {}
        }

        for (int i = 0; i < 5; ++i) {
            System.out.println("띵");
            try {
                Thread.sleep(500);
            } catch (Exception e) {}
        }
    }
}
~~~

<h3>작업 스레드가 담당하도록 나눠보기</h3>

~~~
public class BeepTask implements Runnable {
    public void run() {
        Toolkit toolkit = Toolkit.getDefaultToolkit();
        for (int i = 0; i < 5; ++i) {
            toolkit.beep();
            try {
                Thread.sleep(500);
            } catch (Exception e) {}
        }
    }d
}

public class BeepPrintExample2 {
    public static void main(String[] args) {
        Runnable beepTask = new BeepTask();
        Thread thread = new Thread(beepTask);
        thread.start();

        for (int i = 0;i < 5; ++i) {
            System.out.println("띵");
            try {
                Thread.sleep(500);
            } catch (Exception e) {}
        }
    }
}
~~~

<h2>Thread 하위 클래스로부터 생성</h2>

~~~
public class WorkerThread extends Thread {
    @Override
    public void run() {
        //스레드가 실행할 코드
    }
}
~~~