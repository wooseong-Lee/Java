<h2>private 생성자로 객체의 생성을 막아라.</h2>

- static 메소드/필드만으로 이루어진 클래스의 생성을 막기 위해 사용.

<strong>e.g:</strong> java.lang.Math, java.util.Arrays

- 특정 인터페이스(Collection<T>)를 구현하는 클래스(SynchronizedCollection)를 위한 static 메소드(synchronizedCollection)등을 모아놓기 위해 사용

<strong>e.g:</strong> java.util.Collections

- 상속을 막기 위해 사용할 수 있다. 상속한 경우 명시적이든 암묵적이든 상위 클래스의 생성자를 호출해야 하는데, 이 클래스의 생성자가 private인 경우 상속을 할 수 없다.

<br/>
<br/>

생성자가 없으면 컴파일러가 자동으로 디폴트 생성자를 생성해주므로, 명시적으로 객체의 생성을 막기 위해선 private 생성자를 선언해야한다.
<br/>
단순히 클래스를 abstract로 지정하면, 해당 클래스를 상속함으로써 생성자를 호출해서 간접적으로 생성할 수 있으므로 객체 생성을 막는 완전한 방법이 아니다. 또한 사용자로 하여금 상속을 위한 클래스로 오해하게 할 수 있다.

~~~

//Noninstantiable utility class
public class UtilityClass {
    //Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ...
}

~~~

생성자 내부의 에러 처리는 필수는 아니지만, 클래스 내부에서 생성자가 의도치 않게 호출된 경우 감지할 수 있다. 즉 어떤 상황에서도 해당 클래스의 객체가 생성될 수 없음을 보장한다.
<br/>

하지만 생성자가 코드상에 보임에도 객체를 생성할 수 없다는건 비 직관적이므로 위의 예제처럼 주석을 함께 달아놓는 것도 나쁘지 않다.
