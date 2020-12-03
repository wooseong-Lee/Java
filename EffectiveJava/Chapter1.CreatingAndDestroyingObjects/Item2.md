<h2>Item 2: 생성자의 파라미터가 많다면 빌더를 고려해봐라</h2>

<strong>static factory 메소드와 생성자는 공통적인 한계:</strong> 선택적인 매개변수가 많을 경우 적절한 대응이 어렵다.
<br/>
<br/>
아래와 같이 여러개의 field가 있고, 이 field중 일부만 갖는다고한다면 이런 클래스는 어떤 팩토리 메소드 / 생성자를 가져야하나?
<br/>

<h3>해결책 1: 점층적 생성자 패턴(Telescoping constructor pattern)</h3>

~~~
public cass NutritionFacts {
    private final int servingSize;  //required
    private final int servings;     //required
    private final int calories;     //optional
    private final int fat;          //optional
    private final int sodium;       //optional
    private final int carbohydrate; //optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fact) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fact, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fact, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
~~~

<strong>점층적 생성자 구조(Telescoping Constructor Structure):</strong> 이는 '필수 매개변수 + 선택 매개변수'의 형태로 생성자를 오버로딩 하는 것으로, 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다.
<br/>

~~~
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
~~~

<h5>단점</h5>
<br/>
- 위의 경우 fat은 필요가 없지만 파라미터로 넘겨야하며, 파라미터의 수가 많아질 수록 이렇게 비효율적인 상황이 더 많이 발생한다.
- 클라이언트는 코드를 작성하기 어려워지고, 코드 가독성도 떨어진다.
- 동일한 타입의 파라미터가 여러개 있다면, 사소한 버그를 불러일으킬 수 있다. 순서를 반대로 넣는다면 컴파일 에러가 발생하지 않지만 런타임에 에러가 발생할 수 있다.

<h3>해결책 2: 자바빈즈 패턴(JavaBeans Pattern</h3>

파라미터가 없는 생성자를 만들어서 객체를 생성한 뒤 각각 setter 메소드로 적절하게 초기화해준다.

~~~
// JavaBeans Pattern - allows inconsistency, mandates mutability

public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize = -1;  // Required; no default value;
    private int servings = -1;     // Required; no default value;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {}

    //Setters
    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calories = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}
~~~

이 패턴은 점층적 생성자 패턴이 가졌던 문제를 가지지 않는다. 객체를 생성하기도 쉽고 코드 가독성도 좋다.

~~~
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
~~~

<h5>단점</h5>
<br/>

- 생성 과정이 분리되어 있기 때문에 완전히 생성되기전까지는 일관성이 무너진 상태에 놓일 수 있다.
- setter로 인해 class를 immutable한 상태로 만들 수 없고, 프로그래머가 thread-safe한 코드를 위해 더 주의를 기울여야한다.

<h3>해결책 3: 빌더 패턴</h3>

<strong>Telescoping Constructor pattern의 안정성과 JavaBeans Pattern의 가독성을 모두 확보할 수 있다.</strong>

<br/>
<br/>

<h5>과정</h5>

1. 필수 파라미터를 모두 넘기면서 빌더 객체를 생성한다.
2. 선택적 필드를 초기화하기 위해 setter역할을 하는 메소드를 호출한다.
3. 마지막으로 파라미터가 없는 build() 메소드를 호출해서 객체를 생성한다.

<br/>
<br/>

**빌더는 빌더가 생성해내는 클래스 안에 있는 static inner 클래스다!**

~~~
//Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        //Required parameters
        private final int servingSize;
        private final int servings;

        //Optional parameters = initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Bilder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
~~~

<h5>[NutritionFacts]</h5>

- immutable하다.
- default value들을 모아놓을 수 있다.
- method chaining으로 유연성을 높일 수 있다.
- 클라이언트 코드를 작성하기 쉬우며, 가독성도 좋다.

~~~
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100).sodium(35).carbohydrate(27).build();
~~~
<br/>

**builder pattern의 무결성을 보장하기 위해선 build() 메소드로 생성자를 호출하는 시점에 파라미터를 검증해야한다. 만약 검증이 실패하면 IllegalArgumentException을 발생시키면 된다.(에러 메세지는 검증 실패한 파라미터에 대한 정보가 들어간다.)**
<br/>

- 빌더 패턴은 클래스 계층화에도 적절하게 사용할 수 있다. 추상 클래스는 추상 빌더를 가지며, 구체 클래스는 구체적인(Concrete) 빌더를 갖는다.

~~~
// Builder pattern for class hierarchies
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            returns self();
        }

        abstract Pizza build();

        //Subclasses must override this method to return "this" protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
~~~

<strong>Pizza 클래스의 Builder는 재귀적 타입 파라미터을 갖는 제네릭 타입이며, 이는 self 메소드와 더불어서 서브 클래스에서 타입 캐스팅 없이 메소드 체이닝이 적절하게 동작할 수 있도록 도와준다.</strong>

<br/>

<strong>자바에서 self type이 없어서 사용하게 된 이 형태는 simulated self-type idiom으로 알려져 있다.</strong>

<br/>
<br/>

~~~
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
~~~

<br/>

~~~
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; //default

        public Builder sauceInsize() {
            sauceInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
~~~

- 각 서브 클래스의 빌더 메소드는 각각의 서브 클래스를 반환해준다. <strong>서브 클래스의 메소드가 수퍼 타입에 선언된 타입의 서브 클래스를 리턴하는 것을 공변 반환 타입(Covariant return typing)이라고 한다.</strong>
- 클라이언트는 이 계층적 builder를 기존의 NutritionFacts 빌더를 사용할 때처럼 동일하게 사용할 수 있다.

~~~
NyPizza pizza = new NyPizza.Builder(SMALL).addTopping(SAUSAGE).addTopping(ONION).build();

Calzone calzone = new Calzone().builder().addTopping(HAM).sauceInsize().build();
~~~

<h5>빌더 패턴의 단점</h5>

- 빌더 객체를 생성해야 한다. 성능이 중요시 되는 경우 문제가 될 수 있다.
- 점층적 생성자 패턴보다 장황하다. 따라서 충분히 많은 수의 파라미터가 있는 경우에 사용하는게 좋다.(보통 4개이상인 경우 사용)

<h4>정리</h4>

- 생성자나 팩토리 메소드의 파라미터 수가 많아질 경우 빌더 패턴을 고려해볼 수 있다. <strong>특히 대다수의 파라미터가 선택적이거나(필수가 아님), 동일한 타입을 갖는 경우 유용하다.</strong> 
- 클라이언트의 코드 작성 측면에서 빌더는 점층적 구조보다 작성하기 쉬우며, 가독성이 좋고, 자바빈즈 패턴보다 안전하다.


<br/>
<br/>

<h5>Covariant Return Type</h5>

**OOP에서 메소드의 공변 반환 타입이란 그 메소드가 오버라이딩될 때 더 좁은(narrower)타입으로 교체할 수 있다는 것이다.**

~~~
class Foo {
    public A Foo() {
        return new A();
    }
}

class Bar extends Foo {
    @Override
    public B foo() {
        return new B(); //here
    }
}

class A {}

class B extends A {}
~~~

<h5>Covariant Return Type이 실제로 쓰이는 곳: Cloneable interface</h5>

- Cloneable, Serializable은 마커 인터페이스이다. Object 클래스에 있는 clone()는 기본적으로 protected로 지정되있다. 모든 클래스는 기본적으로 Object를 상속하므로 일반적은 클래스들의 clone 메소드는 자신과 서브클래스 내부에서만 사용 가능하다. Cloneable 인터페이스는 이 메소드를 public으로 쓸 수 있게 해준다.
<br/>

- 아래 코드에서 Namecard는 Object의 clone()을 오버라이딩하지만 반환형이 Namecard이다. 따라서 Namecard 오브젝트의 clone을 사용할 때에는 캐스팅 할 필요가 없다.
~~~
class Namecard implements Cloneable {
    private String name;
    private int age;

    //생성자 및 getter, setter

    @Override
    public Namecard clone() throws CloneNotSupportedException {
        return (Namecard) super.clone();
    }
}
~~~
<br/>

~~~
Point copy = (Point)original.clone(); //공변반환타입 사용안한 경우

Point copy = original.clone(); //공변반환타입 사용한 경우.
~~~

**공변 반환 타입을 사용하면 조상의 타입이 아닌, 실제로 반환되는 자손 객체의 타입으로 반환할 수 있어 번거로운 형변환이 줄었다.**
