<h2>Item 1: 생성자대신 static factory 메소드를 고려하라</h2>

<strong>primitive 타입을 받아서 박싱된 Boolean object reference를 반환해주는 예시</strong>

~~~
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE:
} 
~~~

<strong>팩토리 메소드는 팩토리 메소드 패턴와 동의어가 아니다!</strong>

<h3>Advantages of static factory method</h3>

---------------------------------------------

<h4>1. 이름이 있다.</h4>

- 사용하는 코드에서 용도를 확인하기 쉽다.
- 생성자의 경우 동일한 메소드 시그니처를 갖는 생성자를 생성할 수 없다. 파라미터의 순서를 변경해서 동일한 타입 리스트를 갖는 생성자를 만들어 사용한다면, 어떤 생성자를 언제 사용해야하는지 알 수 없게된다. 결과적으로 class document를 보기전까지 코드만으로는 해당 생성자의 역할을 알 수 없게된다. <strong>따라서 클래스가 동일한 메소드 시그니처의 생성자 여러개를 필요로 한다면 팩토리 메소드를 사용한다.</strong>

<h4>2. 불릴때마다 새 객체를 생성하지 않아도된다.</h4>

- 따라서 불변 클래스같은 경우 미리 생성해놨던 객체를 리턴할 수 있다.(싱글톤 패턴) 따라서 동일한 객체가 자주 요구되는 경우, 특히 객체 생성에 시간이 많이 소요되는 경우 유용하다.
- 위의 Boolean의 valueOf가 이 경우다.
- Flyweight pattern과 비슷하다고 볼 수 있다.
- 특정 시점에 어떤 객체만 존재하게 할지 강제할 수 있다. 이런 클래스를 instance-controlled 클래스라고 한다.

<h5>Instance-controlled 클래스를 사용하는 이유?</h5>

**싱글톤을 보장해줄 수 있으며, 객체화를 금지함으로써 동일한 두 개의 인스턴스가 없음을 보장할 수 있다. 따라서 a == b인 경우만 a.equals(b)가 성립하는 것이 보장된다. Enum도 이런 방식이다.**

<h4>3. 리턴 타입의 서브타입을 반환할 수 있다.(생성자는 이거 안됨.)</h4>

- 유연하게 반환할 객체를 선택할 수 있다.
- 클래스 접근지정자를 public으로하지않고 해당 클래스의 객체를 반환할 수 있다. 이는 interface-based 프레임워크를 가능하게한다.
- 전통적으로 Type(아래 코드에선 Collection)을 생성하기 위한 static factory method는 객체를 생성할 수 없는 Types(아래 코드에서 Collections 클래스) 클래스 아래에 있다.
- 아래 실제 자바 Collection API의 코드에서 리턴되는 타입은 접근지정자가 public이 아니다.
- 자바의 Collections Framework API는 45개의 유틸리티 구현을 제공하는데, 거의 모든 구현이 하나의 객체 생성 불가 클래스에서 static factory 메소드를 통해 제공된다.(이는 45개의 분리된 public class를 제공할때보다 API가 훨씬 가벼워진거다.)
- 프로그래머가 API를 사용하기위해 추가로 봐야하는 문서 및 어려움 또한 줄였다.
- 또한 이 방법으로 반환된 객체를 구현 클래스가 아닌 인터페이스로 참조할 수 있다.
- 자바 8에서부터 인터페이스에 static 메소드를 사용할 수 있으므로, noninstantiable companion class 를 사용할 이유는 줄었지만, 여전히 여러가지 구현 코드를 별도의 패키지 접근지정자를 가진 클래스로 뺴는게 좋다.

~~~
public class Collections {
    //Suppresses default constructor, ensuring non-instantiability
    private Collections() {}

    ...

    public static <T> Collection<T> synchronizedCollection(Collection<T> c) {
        return new SynchronizedCollection<>(c);
    }

    static <T> Collection<T> synchronizedCollection(Collection<T> c, Object mutex) {
        return new SynchronizedCollection<>(c, mutex);
    }

    static class SynchronizedCollection<E> implements Collection<E>, Serializable {
        private static final long serialVersionUID = 3053995032091335093;

        final Collection<E> c;
        final Object mutex;

        SynchronizedCollection(Collection<E> c) {
            this.c = Objects.requireNonNull(c);
            mutex = this;
        }
    }
    ...
}
~~~

<h4>4. 리턴 타입이 파라미터에 따라 바뀔 수 있다.</h4>

- 리턴 타입으로 명시해놓은 타입의 하위 타입은 모두 리턴할 수 있다.
- EnumSet의 경우 public 생성자가 없고 factory method만 있다.
- EnumSet은 64개 이하의 원소가 있다면 팩토리 메소드는 RegularEnumSet 객체를 반환하며, 이는 내부적으로 long 타입 하나를 사용한다.
- 만약 65개이상의 원소가 있다면, 팩토리 메소드는 JumboEnumSet 객체를 반환하며, 이는 내부적으로 long array를 사용한다.
- 이 두개의 구현체는 클라이언트는 알 수 없으며, 추후에 라이브러리에서 변경이 일어나더라도 클라이언트 사이드에서는 고칠게 없다.
- 또한 RegularEnumSet / JumboEnumSet 이외에 추가적으로 더 반환하더다도 클라이언트는 신경쓰지 않아도 된다.

<h4>리턴되는 객체의 클래스는 해당 팩토리 메소드를 포함하는 클래스가 작성되는 시점에 존재하지 않아도 된다.</h4>
<br/>

<strong>이는 Java Databse Connectivity API(JDBC)의 근간이되는 Service Provider Framework의 핵심이다.</strong>

<br/>
<br/>

<h5>Service Provider Framework Pattern의 구성 요소</h5>

1. Service Interface: Service 사용자에게 제공하기 위해 표준으로 정한 API가 정의된 Interface
~~~
interface Connection {...} // JDBC에서의 Service Interface
~~~

2. Service Registration API: Provider Interface의 구현체를 등록하는 API
~~~
DriverManager.registerDriver(); // Provider Register API

/**
* registerDriver() 메소드는 Driver가 로드되는 static 시점에 호출된다. 따라서 * getConnection이 호출되기전 미리 등록된 제공자가 있다고 확신하고 그에 맞는 
* Connection 서비스 구현체를 반환하도록 약속되어 있다.
*/
~~~

3. Service Access API: Service Interface의 구현체를 얻어오는 API(<strong>static factory method</strong>).
<br/>

- 클라이언트가 서비스의 인스턴스를 얻을 때 사용.
- 클라이언트는 서비스 접근 API를 사용할 때 원하는 구현체의 조건을 명시할 수 있다. 조건을 명시하지 않으면 기본 구현체를 반환하거나 지원하는 구현체들을 하나씩 돌아가며 반환한다.

~~~
DriverManager.getConnection(); //Service Access API

/**
* getConnection()은 Connection이라는 서비스 인터페이스를 반환하는데, 제공자 
* 등록 API 역할을 하는 DriverManager.registerDriver() 메소드로 등록한 제공자
* (Driver)에 맞는 Connection 서비스를 반환한다. 즉 mySql Driver, Oracle 
* Driver등 DB에 따라 다른 Connection을 제공한다는 뜻이다.
* 
*/
~~~

4. Provider Interface: Service Interface의 하위 객체를 생성해주는 API가 정의된 Interface

- Service Provider 인터페이스가 없다면 리플렉션을 이용해서 각 구현체의 인스턴스를 생성한다.
<strong>Service Provider Framework Pattern 구현 예시</strong>

~~~
// 1). Service Interface
public interface Service {...}

// 4). Provider Interface
public interface Provider {
    Service newService();
}

//팩토리 클래스, 객체생성 불가능 클래스
public cass Services {
    private static final Map<String, Provider> providersMap = new ConcurrentHashMap<String, Provider>();

    // 객체 생성 방지
    private Services() {}

    // 2).Service Registration API
    public static void registerProvider(String name, Provider provider) {
        providersMap.put(name, provider);
    }

    // 3). Service Access API (정적 팩토리 메소드)
    public static Service getMyService(String name) {
        Provider provider = providersmap.get(name);

        if (provider != null) {
            return provider.newService();
        } else {
            throw new IllegalArgumentException("No provider");
        }
    }

}

~~~

**DI Framework도 Service Provider로 볼 수 있다!**
<br/>

<h3>Disadvantages of static factory method</h3>

---------------------------------------------

<h4>public/protected 생성자가 없는 경우 서브클래스를 만들 수 없다.</h4>

예를 들어 Collections Framework에 있는 클래스들의 서브 클래스를 만들 수 없다. 하지만 단점이 아닐 수도...? Inheritance보다 Composition을 사용하도록 강제하는 효과!

<h4>프로그래머가 찾기 힘들다.</h4>

- 생성자는 API docs 상단에 모아놔서 찾기 쉬운반면 팩토리 메소드는 다른 메소드와 구분없이 보여준다. 따라서 사용자가 정적 팩토리 메소드 방식 클래스를 사용할 때 인스턴스화할 방법을 알아서 찾아야한다.
- 일반적으로 공통 명명 규칙을 준수함으로써 이 문제를 어느정도 해결할 수 있다.

---------------------------------------------

- <strong>from:</strong> 파라미터를 하나 받아서 해당 타입(static factory method를 포함하는 클래스)의 인스턴스를 생성해주는 타입 변환 메소드

~~~
Date d = Date.from(instant);
~~~
<br/>

- <strong>of:</strong> 여러개의 파라미터를 받아 그들을 포함하는 인스턴스를 생성해주는 집계 메소드

~~~
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
~~~
<br/>

- <strong>valueOf:</strong> from, of의 더 구체적인 버전

~~~
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
~~~
<br/>

- <strong>instance / getInstance:</strong> 매개변수를 받을 경우, 매개변수로 명시한 인스턴스를 반환(동일한 인스턴스를 보장하지는 않는다.)

~~~
StackWalker luke = StackWalker.getInstance(options);
~~~
<br/>

- <strong>create / newInstance:</strong> instance / getInstance와 같으나, 항상 새로운 인스턴스를 생성해서 반환해주는 것을 보장한다.

~~~
Object newArray = Array.newInstance(classObject, arrayLen);
~~~
<br/>

- <strong>getType:</strong> getInstance와 동일하지만, 팩토리 메소드가 다른 클래스에 있는 경우 사용된다. getType에서의 Type은 리턴되는 객체의 클래스를 의미한다.

~~~
FileStore fs = Files.getFileStore(path);
~~~
<br/>

- <strong>newType:</strong> newInstance와 동일하지만, 팩토리 메소드가 다른 클래스에 있는 경우 사용된다. getType에서의 Type은 리턴되는 객체의 클래스를 의미한다.

~~~
BufferedReader br = Files.newBufferedReader(path);
~~~
<br/>

- <strong>type:</strong> getType / newType의 간결한 버전

~~~
List<Complaint> litany = Collections.list(lagacyLitany);
~~~
<br/>

<h3>정리</h3>
static factory 메소드와 생성자는 모두 장단점이 있다. 하지만 대부분 static factory 메소드가 우위를 가지므로, 생성자를 만들기전에 static factory 메소드를 고려해보자.