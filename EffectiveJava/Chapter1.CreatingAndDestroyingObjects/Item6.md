<h1>불필요한 객체 생성을 피하라</h1>

<h2>String 인스턴스</h2>

때로는 필요할 때마다 새로 생성하기보단, 생성했던 하나의 객체를 재사용하는게 나을 수 있다. 객체가 불변 객체라면 재사용할 수 있다.
<br/>
다음은 안좋은 코드 예시다.

~~~

String s = new String("bikini");

~~~

위의 선언문은 실행될 때마다 새로운 String 인스턴스를 생성하는데 불필요한 객체 생성이다. 만약 위의 선언이 루프 안에서 사용되거나, 자주 호출되는 메소드안에 존재한다면 매우 많은 갯수의 (불필요한)String 객체가 생성될 수 있다.

<br/>
위의 개선된 버전은 아래와 같다.

~~~

String s = "bikini";

~~~

위의 코드는 매번 새로운 객체를 생성하기보다 하나의 String 인스턴스를 사용하며, 동일한 JVM에서 동작하는 코드 어디에서라도 String literal이 똑같다면 동일한 인스턴스의 사용을 보장해준다.

<h2>Static factory method</h2>

static factory 메소드와 생성자를 모두 제공하는 클래스에서 생성자대신 static factory 메소드를 사용하면 불필요한 객체 생성을 피할 수 있다.

<br/>
<br/>

<strong>Boolean.valueOf(String)</strong>이 <strong>Boolean(String)</strong>(자바9에서 deprecated됨)보다 우선시된다.

<h2>캐싱(Caching)</h2>

몇몇 객체 생성은 더 많은 자원(메모리, 시간)이 소모될 수 있다. 이런 비싼 객체를 반복적으로 필요로한다면, 재사용을 위해 이 객체를 캐싱할 수도 있다. 문자열이 로마 숫자인지 아닌지 판단하는 메소드를 작성하다고 할 때, 정규식을 이용해서 다음과 같이 쉽게 작성할 수 있다.

~~~

static boolean isRomanNumeral(String s) {
    return s.matches("^("^("=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}

~~~

<br/>
<br/>

문제는 이 메소드는 String.matches에 의존한다는 것이다. String.matches는 내부적으로 정규식 매칭을 위한 Pattern 인스턴스를 생성한 뒤 한번만 사용한다. 그리고 Pattern 객체는 가비지 컬렉션의 대상이된다. Pattern 객체는 정규 표현식을 유한 상태 머신으로 컴파일하기 때문에 객체 생성 비용이 비싸다. 따라서 String.matches를 반복적으로 사용하는 경우 성능에 안좋은 영향을 미칠 수 있다.

<br/>
<br/>

성능을 높이려면 클래스 초기화 시점(static)에 명시적으로 정규 표현식을 final한 Pattern 객체에 컴파일하는 것이 좋다. 즉 캐싱해놓고 isRomanNumeral 메소드가 호출될 때마다 동일한 인스턴스를 계속 사용하는 것이다.

~~~

public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
        "^("^("=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}

~~~

위의 개선된 RomanNumerals 메소드는 자주 호출될 수록 이전 버전에 비해 성능적으로 큰 우위를 가진다. 원래는 보이지 않던 Pattern 인스턴스를 static final 필드로 만드는 것은 첫번째로 이름을 부여할 수 있고, 따라서 정규식 자체보다 가독성이 좋다.

<br/>
<br/>

새로운 버전의 isRomanNumerals 메소드를 포함하는 클래스가 초기화됐는데, 메소드가 한번도 호출되지 않는다면 ROMAN 필드는 쓸데없이 초기화된 것이다. isRomanNumerals 메소드가 호출되는 시점에 ROMAN 필드를 초기화하는 lazily initializing으로 방지 할 수 있다. 하지만 이 방법은 구현을 복잡하게 할 뿐아니라 뚜렷한 성능 향상도 없어서 추천되지 않는 방법이다.

<br/>
<br/>

<h2>Adapter pattern</h2>

Adapter pattern(a.k.a view)은 다른 객체에 위임해주는, 인터페이스 역할을 하는 객체다. Adapter의 경우 위임받는 객체이상의 상태를 갖지 않으므로 backing object당 하나 이상의 Adapter가 생성될 필요가 없다.

<h3>keySet</h3>

<strong>Map.keySet:</strong> Map object의 Set view(adapter)를 반환한다. 동일한 객체에 대해서 keySet을 호출할 때마다 mutable한 동일 객체를 반환해준다. keySet view 객체를 여러개 만들 수 있겠지만, 아무 쓸모 없는 짓이다.

<h2>Auto-boxing/Auto-unboxing</h2>

오토박싱/언박싱은 자동으로 처리되지만, primitive 타입과 박싱된 primitive 타입의 특성까지 없애진 않는다. primitive type과 박싱된 primitive 타입엔 약간의 의미적 차이가 있고, 꽤 큰 성능적 차이가 있다. 다음 예시를 보자

~~~

private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; ++i) {
        sum += i;
    }

    return sum;
}

~~~

위의 코드는 알파벳 단 한글자 때문에 실제로 기대되는 속도보다 느리게 동작한다. i가 sum에 더해지는 매순간마다 Long 객체가 생성된다.

<br/>
<br/>

**되도록 primitive 타입을 사용하고, 의도치않은 boxing을 피해라**

<h2>정리</h2>

<strong>오해하지 말 것: </strong> 객체 생성은 비싸며, 피해야하는 일이란게 아니다!
<br/>

**오히려 작은 객체(생성자도 별 일을 안하는)의 생성과 재선언은 현대 JVM에서 매우 싸다.**

<br/>

Object Pool을 만들어놓고 객체 생성을 피하는 것은 Object Pool안의 객체가 Pool안에 둘 만큼 무겁지 않은 이상 좋지 않을 수 있다.

<br/>
<br/>

Object Pool의 사용이 적절한 예시는 DB 커넥션이다. 커넥션을 설정하는 비용은 매우 높으므로 Object Pool(Connection Pool)에 있는 객체를 다시 사용하는 것이 좋다. 쓸데없는 Object Pool을 만들면 코드도 더러워지고, 메모리 점유량이 커지며, 성능에 좋지 않을 수 있다. 최근의 JVM은 매우 최적화된 성능의 GC를 가지고 있으므로 쓸데없는 Object Pool을 만드는 것보다 JVM에 맞기는게 나을 수 있다.

<br/>
<br/>

Item50과 대조되는 주제다!
<br/>

- <strong>Item6: </strong> 지켜지지 않으면 코드가 더러워지고, 성능 문제 야기

- <strong>Item50: </strong> 지켜지지 않으면 발견하기 힘든 버그를 발생시킬 수 있고, 보안문제를 일으킬 수 있다!(지키지 않으면 더 치명적인 결함을 야기할 수 있다.)
