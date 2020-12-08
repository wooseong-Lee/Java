<h1>직렬화(Serialization)</h1>

<h2>직렬화 사용하기</h2>

<strong>외부의 다른 자바 시스템에서 활용할 수 있도록 자바 객체나 데이터를 바이트 형태로
변환하는 기술을 말한다.</strong> 이러한 문맥에서 <strong>직렬화(Serialization)
</strong>은 포맷을 변경하는 기술이라고 볼 수 있다.

<br/>
<br/>

직렬화를 하기 위해선 객체가 <strong>직렬화 가능한 클래스</strong>의 인스턴스여야 
한다는 것이다.간단하게 클래스를 선언할 때 Serializable 인터페이스를 구현하면 직렬화
 가능한 클래스가 된다.

~~~
class Article implements Serializable {
    private String title;
    private String pressName;
    private String reporterName;

    public Article(String title, String pressName, String reporterName) {
        this.title = title;
        this.pressName = pressName;
        this.reporterName = reporterName;
    }

    @Override
    public String toString() {
        return String.format("title = %s, pressName = %s, reporterName = %s",
                title, pressName, reporterName);
    }
}
~~~

**물론 직렬화 가능한 클래스를 상속하면 직렬화 가능한 클래스가 된다.**

<br/>
<br/>

객체 직렬화에는 java.io.ObjectOuputStream 클래스가 사용된다.

~~~

public class SerializeTester {
    public String serializeMethod() {
        Article article = new Article("직렬화란?", "XX일보", "XX");
        ByteArrayOutputStream bos = new ByteArrayOutputStream();

        try (bot; ObjectOutputStream oos = new ObjectOutputStream(bos)) {
            oos.writeObject(article);
        } catch (Exception e) {
            // ...생략
        }
        
        //결과가 알아보기 힘든 문자열이므로 base64로 인코딩한다.
        return Base64.getEncoder().encodeToString(bos.toByteArray());
    }

    public static void main(String[] args) {
        SerializeTester tester = new SerializeTester();
        String data = tester.serializeMethod();

        // rO0ABXNyAAdBcn... 와 같은 인코딩된 문자열 출력 
        System.out.println(data);
    }
}

~~~

위의 직렬화도니 데이터를 이용하여 반대로 <strong>역직렬화(Deserialization)</strong>
처음 선언한 객체를 얻을 수 있다.

<h2>직렬화 할 때는</h2>

객체를 직렬화할 때 특정 필드를 제외하고 싶다면 멤버 변수가 transient 키워드를 추가하면
된다. 아래와 같이 특정 필드에 선언하면 역직렬화를 하더라도 해당 값은 제외된다.

<br/>

**transient로 선언되었더라도 writeObject/readObject를 이용하면 우회할 수 있다.**

~~~
class Article implements Serializable {
    private String title;
    private String pressName;
    //기자 이름은 직렬화를 제외한다.
    private transient String reporterName;
}
~~~

<br/>

직렬화가 가능한 클래스 내부에 다른 클래스의 객체를 필드로 갖고 있을 수 있다. 이 경우
해당 클래스도 직렬화가 가능하도록 Serializable 인터페이스를 구현하고 있어야한다.

~~~

class Article implements Serialzable {
    private String title;
    private String pressName;
    private String reporterName;

    // java.time.LoacalData.Time 클래스는 Serializable을 구현하고 있다.
    private LocalDateTime articleTime;

    //개발자가 직접 만든 클래스. Serializable을 구현해야한다.
    private DetailInfo detailInfo;
}

~~~

<h2>역직렬화(Deserialization)</h2>

역직렬화에는 ObjectInputStream을 사용한다.

~~~

public class SerializeTest {
    //직렬화 메소드는 위와 동일 가정

    public Article deserializeMethod(String serializedString) {
        //앞선 직렬화에서 Base64 인코딩했으므로 다시 디코딩
        byte[] decodedData = Base64.getDecoder().decode(serializedString);
        ByteArrayInputStream bis = new ByteArrayInputStream(decodedData);
        try (bis; ObjectInputStream ois = new ObjectInputStream(bis)) {
            return (Article) ois.readObject();
        } catch (Exception e) {
            // ... 생략
        }
        return null;
    }

    public static void main(String[] args) {
        SerializeTester tester = new SerializeTester();
        String data = tester.serializeMethod();
        Article article = tester.deserializeMethod(data);

        System.out.println(article);
    }
}

~~~

<strong>역직렬화 할 때는 직렬화된 객체의 클래스가 반드시 클래스 패스(Class path)에
존재해야 하며, import 된 상태여야한다.</strong>

<h2>writeObject, readObject</h2>

기본적인 자바 직렬화 또는 역직렬화 과정에서 별도의 처리가 필요할 때는 writeObject,
readObject 메소드를 클래스 내부에 선언해주면된다.(커스터마이징) 물론 해당 클래스는
Serializable 인터페이스를 구현한 직렬화 대상 클래스여야한다. 직렬화 과정에서는 writeObject,
 역직렬화 과정에서는 readObject가 자동으로 호출된다.

 <br/>
 <br/>

직렬화 가능 클래스인 Article을 아래과 같이 수정할 수 있다.

~~~

public class Article implements Serializable {
    private transient Integer id;
    private String title;
    private String pressName;
    private String reporterName;

    /**
    * 직렬화 할 때 자동으로 호출된다.
    * 반드시 private으로 선언해야 한다.
    */
    private void writeObject(ObjectOutputStream oos) {
        try {
            oos.defaultWriteObject();
            oos.writeObject(this.id);
            oos.writeObject(this.title);
            oos.writeObject(this.pressName);
            oos.writeObject(this.reporterName);
            System.out.println("writeObject method called");
        } catch (IOException e) {
            // ... 생략
        }
    }

    /**
    * 역직렬화 할 때 자동으로 호출된다.
    * 반드시 private으로 선언해야 한다.
    */
    private void readObject(ObjectInputStream ois) {
        try {
            ois.defaultReadObject();
            this.id = (Integer)ois.readObject();
            this.title = (String)ois.readObject();
            this.pressName = (String)ois.readObject();
            this.reporterName = (String)ois.readObject();
            System.out.println("readObject method called");
        } catch (IOException | ClassNotFoundException e) {
            // ... 생략
        }
    }
}

~~~

<strong>중요하게 봐야하는 것은 writeObject와 readObject 메소드의 접근지정자를
private으로 선언한 것이다! 다른 접근 지정자로 선언 시 자동으로 호출되지 않는다.</strong>

- writeObject: ObjectOutputStream의 defaultWriteObject 메소드를 가장 먼저
호출해야하며, 이어서 클래스의 직렬화할 필드를 writeObject 메소드의 인수로 넘기면된다.(예제에서 transient
로 선언된 필드도 포함되있다.)

- readObject: ObjectInputStream의 defaultReadObject 메소드가 가장 먼저 선언되야 하며,
직렬화한 필드 순서대로 ObjectInputStream의 readObject 메소드를 수행하여 클래스의 멤버 필드에 대입한다.

<strong>유심히 봐야하는 것은 transient가 선언된 키워드가 직렬화 대상에 포함됐다는 것이다.</strong>

<h3>왜 private인가?</h3>

private 으로 선언되었다는 것은 이 클래스를 상속한 서브 클래스에서 메서드를 재정의(override)
를 하지 못하게 한다는 것이다. 다른 객체는 호출할 수 없기 때문에 클래스의 무결성이 유지되며
수퍼 클래스와 서브 클래스는 독립적으로 직렬화 방식을 유지하며 확장될 수 있다.
직렬화 과정에서는 리플렉션(reflection)을 통해 메서드를 호출하기 때문에 접근 지정자는
문제가 되지 않는다.

<h2>SerialVersionUID(SUID)</h2>

직렬화 할 때 SUID 선언이 없다면 내부에서 자동으로 유니크한 번호를 생성해서 관리한다. SUID는
직렬화와 역직렬화 과정에서 값이 서로 맞는지 확인한 후에 처리하기 때문에 이 값이 맞지 않다면
InvalidClassException 예외가 발생한다.

<br/>
<br/>

**자바 직렬화 스펙 정의를 보면 SUID값은 필수가 아니며, 선언되어있지 않으면 클래스의 기본 해시값을 사용한다.**

<br/>

따라서 <strong>직접 SUID를 명시하지 않아도 내부에서 자동으로 값이 추가되며</strong>, 이 값들은
클래스의 이름, 생성자 등과 같이 클래스 구조를 이용해서 생성한다. 앞선 예제에서도 직렬화 가능한 클래스를
선언할 때(Article) SUID를 생략헀지만, 내부적으로 정보가 생성되어 있음을 유추할 수 있다.

<br/>

따라서 Article 클래스를 직렬화 한 뒤, Article 클래스에 필드를 추가하고 이를 역직렬화하면
아래와 같은 예외가 발생한다.

~~~
java.io.InvalidClassException: Article;
    local class incompatible: stream classdesc serialVersionUID = 6824395829496368166,
    local class serialVersionUID = 1162379196231584967
~~~

<h3>어떡해야 하나...?</h3>

자바에서는 SUID를 개발자가 선언하고 관리하는 방식을 권장한다.

~~~

class Article implements Serializale {
    private static final long serialVersionUID = 1;

    private String title;
    private String pressName;
    private String reporterName;

    // ... 생략
}

~~~

<strong>이렇게 SUID를 직접 관리하면 Article 클래스에 멤버를 추가해도 직렬화 
과정에서 오류가 발생하지 않는다.</strong> 따라서 이런 관점에서, 직렬화를 사용할 때는
자주 변경될 소지가 있는 클래스의 객체는 사용하지 않는 것이 좋다. 프레임워크 또는 라이브러리에서
제공하는 클래스의 객체도 버전없을 통해 SerialVersionUID가 변경될 여지가 있으므로
예상치못한 오류가 발생할 수 있다.