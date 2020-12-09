<h1>DI를 사용하라</h1>

예를 들어, 스펠링 체크 클래스는 사전에 의존하는데 이 경우 스펠링 체크 클래스는 static utility 클래스로 구현될 수 있다.

~~~

public class SpellChecker {
    private static final Lexicon lexicon dictionary = ...;

    private SpellChecker() {} // 객체화할 수 없음.

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}

~~~

마찬가지로 이들은 싱글톤으로 구현될 수도 있다.

~~~

public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}

~~~

이 두 가지 방법의 문제는 dictionary 클래스가 하나만 존재한다고 가정하는 것이다. 각 언어는 각자의 dictionary를 가질 수 있다. 또한 테스트는 별개의 dictionary로 해야할 수도 있다.

<br/>

dictionary 필드를 non-final로 만든 뒤 setter를 이용해서 dictionary 클래스의 변경을 허용하면 이상하고, 에러가 발생하기 쉬우며, 동시성 문제를 일으킬 수 있다.

<br/>
<br/>

**클래스의 동작이 의존하는 클래스마다 변경되야하는 경우, 위와 같이static utility 클래스 혹은 싱글톤으로 만드는 것은 좋지않다.**

<br/>

<strong>클라이언트의 요구에 따라 다양한 객체에 대한 의존성을 갖는 클래스를 만들어야 한다.</strong>

<h2>Method of DI 1: 객체를 생성할 때 생성자를 통해 넘기기</h2>

~~~

//DI는 유연함을 제공하며 테스트하기 쉽고 재사용성이 높다..
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}

~~~

DI는 임의의 갯수의 자원에 대해서도 이뤄질 수 있으며, 임의의 의존성 그래프 형태를 띌수도 있다. 또한 DI는 immutability를 보장하므로 여러 객체가 자원을 공유할 수 있다. DI는 생성자, static factory 메소드, 빌더 등을 이용해서 사용할 수 있다.

<br/>
<br/>

또 다른 방법으로 생성자에 factory를 넘길 수 있다.

~~~

Mosaic create(Supply<? extends Tile> tileFactory) { ... }

~~~

DI는 유연하며, 테스트하기 용이하지만 프로젝트가 수천 개의 의존성을 포함하며 프로젝트의 크기 너무 커질 수 있다. 하지만 이러한 복잡함으로 DI framework를 통해서 해결될 수 있다. 