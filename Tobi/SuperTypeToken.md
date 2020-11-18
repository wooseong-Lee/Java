<h1>Type token</h1>
<strong>about Type Erasure:</strong> https://medium.com/asuraiv/java-type-erasure%EC%9D%98-%ED%95%A8%EC%A0%95-ba9205e120a3

~~~

public class TypeToken {
    //자바의 type eraser에 의해 컴파일되면 사라진다.모두 Object 타입으로 된다.
    //컴파일러가 체크를 하고 캐스팅하는 코드를 모두 넣어준다.

    static <T> T create(Class<T> clazz) throws Exception {
        return clazz.newInstance();
    }

    public static void main(String[] args) throws Exception {
        String o = create(String.class);
        System.out.println(o.getClass());
    }
}

~~~

아래 처럼 만들면 사용하는 쪽에서 캐스팅을 해야한다.

~~~
Map<String, Object> map = new HashMap<>();
~~~

아래의 문제점은 주석 친 부분처럼 사용할 수 있다.

~~~
class TypeSafeMap {
    Map<Class<?>, Object> map = new HashMap<>();

    void put(Class<?> clazz, Object value) {
        map.put(clazz, value);
    }
}

public static void main(String[] args) throws Exception {
    TypeSafeMap m = new TypeSafeMap();
    //m.put(Integer.class, "value");
    m.put(String.class, "value");
}
~~~

따라서 TypeSafeMap 의 파라미터를 맞춰주는게 좋다. 컴파일러가 체크할 수 있도록 하는게 핵심

~~~
class TypeSafeMap {
    Map<Class<?>, Object> map = new HashMap<>();

    <T> void put(Class<T> clazz, T value) {
        map.put(clazz, value);
    }

    <T> T get(Clazz<T> clazz) {
        //cast 메소드 : Object객체를 upcasting 해준다.
        return clazz.cast(map.get(clazz));

        //return (T)clazz.cast(map.get(clazz)); 이렇게 하면 타입 세이프한게 아니다
    }
}
~~~

<strong>Type Token: </strong> 타입에 대한 정보를 값으로 넘기겠다는 의미.<br/>

<strong>Type Token의 한계</strong><br/>
아래의 경우에서 List<Integer>가 있던 상태에서 List<String>을 넣으면 덮어씌워진다.

~~~
m.put(List.class, List.of(1, 2, 3));
m.put(List.class, List.of("a", "b", "c"));
~~~

그래서 아래와 같이 변경하면 오류가 발생한다.<br/>
클래스 리터럴로 오브젝트를 가져올 때, 해당 클래스에 대한 오브젝트를 가져올 때 타입 정보를 가지고 가져올 수 없기 때문이다.<br/>
왜냐하면 해당 클래스에는 제네릭 타입에 대한 정보가 없기 때문이다.

~~~
m.put(List<Integer>.class, List.of(1, 2, 3));
~~~

<strong>따라서 이런 방식으로는 제네릭 타입이 있는 타입 토큰을 사용할 수 없다.</strong><br/>

<strong>언어에서 Generic을 지원하는 두 가지 방법</strong>

- ERASURE(java): 컴파일 시점에 제네릭에 대한 정보를 다 지워버림
- REFICATION: FEFIABLE하게 만드는 것(c#) -> 구체화 하는 것. 실제로 바이트 코드에 제네릭에 대한 정보를 남긴다.

<h1>Super Type Token</h1>

<h3>우리가 원하는 것</h3>
<strong>List<String>.class 와 List<Integer>.class를 구분하고 싶다</strong><br/>

아래에서 출력의 결과는 String이 아닌 Object다. 컴파일 시점에 타입에 대한 정보가 다 지워지므로
리플랙션을 이용해도 Sup.value의 타입을 알 수 없다.

~~~
public class SuperTypeToken {
    static class Sup<T> {
        T value;
    }

    public static void main(String[] args) throws NoSuchFieldException {
        Sup<String> s = new Sup<>();

        System.out.println(s.getClass().getDeclaredField("value").getType());
    }
}
~~~

특정 경우에는 해당 정보를 가져올 수 있다.
아래와 같은 경우에는 String 타입을 얻을 수 있다. 즉 런타임에도 알 수 있는 특정한 상황이 존재한다.
gGg
~~~
public class SuperTypeToken {
    static class Sup<T> {
        T value;
    }

    static class Sub extends Sup<String> {

    }

    public static void main(String[] args) throws NoSuchFieldException {
        Sub b = new Sub();
        Type t = b.getClass().getGenericSuperclass();
        ParameterizedType ptype = (ParameterizedType)t;
        System.out.println(ptype.getActualTypeArguments()[0]);
    }
}
~~~