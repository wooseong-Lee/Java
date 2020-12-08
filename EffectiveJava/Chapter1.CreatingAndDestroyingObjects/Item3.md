<h2>Item 3: private 생성자 혹은 enum 타입을 이용해서 싱글톤 속성을 강제하라.</h2>
<br/>

<h3><strong>싱글톤이란? </strong></h3>
상태를 갖지 않는 객체 혹은 내부적으로 유일한, 정확히 한번만 생성되는 객체를 의미한다.
<strong>싱글톤을 사용하는 클라이언트 코드는 테스트하기 어렵다. 싱글톤이 인터페이스를 구현하지 않은 이상 mock으로 교체하는게 어렵기 때문이다.</strong>

<br/>
<br/>

일반적으로 싱글톤으로 만드는데 두 가지 방법이 있는데, 두 가지 방법 모두 다
<br/>

1. private 생성자를 갖으며
2. 단일 인스턴스에 대한 접근을 허용하기 위해 public static 멤버 클래스를 갖는다.

<h4>첫 번째 방법</h4>

~~~

//Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}

~~~

- private 생성자는 public static final 필드인 INSTANCE 변수를 초기화하는 순간에 한번만 호출된다.
- public / protected 생성자가 없음으로써 단 한개의 객체 생성을 보장해준다. <strong>즉, Elvis 클래스가 초기화되는 순간에(링킹 시) 생성되는 Elvis 인스턴스하나만 존재한다.

**AccessibleObject.setAccessible 메소드로 리플렉션을 이용해서 private 생성자에 접근하는 방법이 있긴하다. 이 또한 막기 위해서는 private 생성자가 2번 호출될 때 예외를 던지도록 수정한다.**
<br/>

<h5>public field 방법의 장점</h5>

- API를 통해 해당 클래스가 싱글톤임을 알 수 있다.
<br/> **public static 필드가 final이니깐 INSTANCE 변수는 언제나 동일한 객체를 가리키고 있겠구나!**

- 단순하다.(작성하기 쉽다.)

<h4>두 번째 방법</h4>

~~~

//Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance( return INSTANCE; )

    public void leaveTheBuilding() { ... }
}

~~~

- getInstance 메소드를 호출할 때마다 매번 같은 객체를 참조하는 참조 변수가 반환된다.

<h5>static factory 방법의 장점</h5>

- API를 바꾸지 않고 싱글톤이 아닌 객체를 반환하도록 바꿀 수 있다.(유연하다.)

**기존에는 단일 인스턴스를 반환하다가, 클라이언트의 코드 수정없이 자신을 호출한 스레드마다 다른 인스턴스를 반환하도록 변경될 수 있다.**

- generic singleton factory를 작성할 수 있다.
- getInstance등을 메소드 참조에 담을 수 있다.

~~~
Supplier<Elvis> ex = Elvis::getInstance;
~~~

<strong>만약 하나라도 해당되지 않는다면, public field 방식이 나을 수 있다.</strong>

<h5>직렬화 (Serialization)</h5>

- 위의 두 방식을 사용하는 싱글톤 클래스를 serializable하게 만들려면 단순히 Serializable 인터페이스를 구현하는 것만으로는 불충분하다.

- 역직렬화 할 때 클래스가 로딩되면서 INSTANCE 변수가 또 생길 수 있으므로, 같은 타입의 인스턴스가 여러개 생길 수 있다.

- <strong>이 문제를 해결하려면 모든 인스턴스 필드에 transient를 추가(직렬화하지 않겠다는 뜻)하고, readResolve 메소드를 다음과 같이 구현하면 된다.</strong>

~~~
private Object readResolve() {
    return INSTANCE;
}
~~~

serialization은 readResolve()라는 private method를 통해서 instantiation을 진행한다.

readResolve, writeReplace는 직렬화과정에서 proxy 역할을 한다. 반면, readObject, writeObject는 프록시 과정이 동반되지 않는 일반적인 직렬화/역직렬화 과정에 사용된다.

~~~
//readObject가 반환된 후 readResolve가 호출된다.
readObject -> readResolve

//반대로 writeReplace가 호출된 후 writeObject가 호출된다.
wrtieReplace -> writeObject
~~~
<br/>
<br/>

deserialization될 때, readObject 객체는 내부적으로 해당 클래스가 readResolve 객체를 가지고 있는지 확인한 뒤, readResolve 객체가 있다면 readResolve 객체가 반환하는 객체를 deserialize한다. 즉 readResolve는 역직렬화되는 객체를 프록시하는 역할을 한다.

<br/>

**즉, readResolve와 writeReplace는 동일한 객체/혹은 다른 객체를 리턴할 수 있다.필드가 final인 경우, 혹은 이전과 데이터 정합성이 중요시 되는 경우 동일한 객체를 반환하도록 할 수 있다.**


<h4>Enum</h4>

직렬화 / 역직렬화 할 때, 코딩으로 문제를 해결할 필요도 없고, 리플렉션으로 호출되는 문제를 고민할 필요도 없는 방법이 있다.

~~~
public Enum Evlis {
    INSTANCE;
}
~~~

하지만 이 방법은 Enum말고 다른 상위 클래스를 상속해야한다면, 사용할 수 없다.(하지만 Enum도 인터페이스는 구현할 수 있다.)