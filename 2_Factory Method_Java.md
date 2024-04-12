# 팩토리 메서드 패턴
https://cyclic-basket-9b5.notion.site/1c1eeedb4edc4f388897822b04237ee9?pvs=4

## Factory Method - 시작

객체 생성을 **공장(Factory) 클래스로 캡슐화 처리해서 대신 생성하게 하는** 생성 디자인 패턴이다.

좀 더 자세하게 설명하자면, **클라이언트에서 직접 new 연산자를 통해 제품 객체를 생성하는 것이 아니라 대신 제품 객체를 생성할 공장 클래스를 만들고, 이를 상속하는 서브 공장 클래스의 메서드에서 여러가지 제품 객체 생성을 각각 책임지는 것**을 말한다.

이 패턴을 이용하는 이유는 객체간의 결합도를 낮출 수 있고, 유지보수에 용이해지기 때문이다.

<aside>
💡 그래 공장한테 요청하면 서브 공장 클래스의 메서드에서 제품을 생성한다는 말을 표현한건 알겠는데,  **그냥 한 공장에서 다 만들면 안돼? 왜 서브 공장이 필요한데?**라는 물음이 생길 수 있다.

</aside>

이를 이해하기 위해서는 먼저 **Factory Method 패턴과 Abstract Factory 패턴**의 베이스인 **Simple Factory**를 알아야한다.

참고로 Simple Factory는 디자인 ‘패턴’이라고는 할 수 없고, OOP에서 자주 쓰이는 관용구라고 보면 된다.

## Simple Factory?

Simple Factory는 객체를 생성해내는 공장을 따로 두는 것이다.

**즉, 객체 생성 부분을 전담하는 공장 클래스가 따로 있는 것이다.**

객체는 여러 곳에서 생성될 수 있다.
하지만 호출하는 쪽이 객체의 생성자에 직접 의존하고 있다면, 나중에 수정되어야 하는 코드가 많이 발생한다. (밑에서 예시를 통해 확인해보자!)

따라서 생성자 호출(new)을 별도의 클래스(Factory)에서 담당하고 클라이언트 코드에서는 이 Factory를 통해 객체를 생성해야 한다.

### Example - 구현 클래스 의존 O

```java
public interface Sopt {
    void printLanguage();
}

public class Server implements Sopt {
	  @Override
    public void printLanguage() {
        System.out.println("Java");
    }
}

public class Ios implements Sopt {
	  @Override
    public void printLanguage() {
        System.out.println("Swift");
    }
}
```

Sopt를 한번 예시로 들어보자.

공통 인터페이스인 `Sopt` 을 정의하고 이를 구현하는 `Server`, `Ios` 클래스이다.

이제 클라이언트 코드에서 `Server`, `Ios` 을 사용하기 위해 객체를 생성할 수 있다.

```java
public class Client {
    public static void main(String[] args) {
	    Sopt server = new Server();
	    Sopt ios = new Ios();
	    
	    server.printlanguage();
	    ios.printlanguage();
    }
}
```

일반적인 사용법은 new를 사용해 구현 클래스를 생성한 후 호출하는 것이다.
하지만 이렇게 되면 클라이언트와 클래스들 사이에 다음과 같은 의존관계가 생긴다.

![simple_factory](https://github.com/NOW-SOPT-SERVER/hoonyworld/assets/125895298/768d57bb-2734-414d-8455-5cab8660b35f)

이렇게 구현 클래스(Server, Ios)를 의존하고 있으면, 해당 클래스의 생성자 or 전처리 코드가 변경되었을 때 사용하는 모든 클라이언트 코드를 변경해야 한다.

그러면 이제 객체의 생성만을 담당하는 별도의 Factory 클래스를 만들어보자!

---

### Example - 구현 클래스 의존 X

```java
public interface Sopt {
    enum Part { // 추가
        SERVER, IOS    
    }
    
    void printLanguage();
}

public class Server implements Sopt {
	  @Override
    public void printLanguage() {
        System.out.println("Java");
    }
}

public class Ios implements Sopt {
	  @Override
    public void printLanguage() {
        System.out.println("Swift");
    }
}
```

우선 Sopt 인터페이스에 enum 타입으로 선언을 해준다.

```java
public class SoptFactory {

    public Sopt createSopt(Sopt.Part soptPart) {
        switch (soptPart) {
            case SERVER:
                return new Server();
            case IOS:
                return new Ios();
            default:
                throw new IllegalArgumentException("Sopt에 없는 파트입니다");
        }
    }
}
```

SoptFactory라는 팩토리 클래스를 만들고 Sopt.Part에 따라 다른 객체를 생성해서 반환해준다.

```java
public class Client {
    public static void main(String[] args) {
        SoptFactory soptFactory = new SoptFactory();
        Sopt server = soptFactory.createSopt(Sopt.Part.SERVER);
        Sopt ios = soptFactory.createSopt(Sopt.Part.IOS);

        server.printLanguage();
        ios.printLanguage();
    }
}
```

soptFactory를 선언하고 생성 메서드인 create만 호출하면 실제 구현 클래스인 Server, Ios에 의존하지 않은 코드를 작성할 수 있게 된다.

![factory_2](https://github.com/NOW-SOPT-SERVER/hoonyworld/assets/125895298/fad7cd8b-bcfb-4b72-870f-bc0a40bd0048)

이제 의존관계는 다음과 같이 바뀌게 된다. 

클라이언트에서 구현클래스(Server, Ios)에 직접 의존하지 않기 때문에, 클래스 이름이 변경되거나, 생성자가 변경되어도 SoptFactory 내부만 수정하면 된다!!

### Simple Factory의 한계

우와 완벽한데 앞으로 Simple Factory로 구현해야징~

놉! 🙅‍♂️

아쉽지만, Simple Factory는 객체 생성의 역할을 팩토리에 담당시켜 확장이 용이하다는 장점이 있지만, **변경에 닫혀 있어야 한다는 OCP 원칙에 위배된다.**

만약 새로운 Sopt 인터페이스의 구현 클래스로 Android가 추가되었다고 가정해보자.

```java
public class Client {
    public static void main(String[] args) {
        SoptFactory soptFactory = new SoptFactory();
        Sopt server = soptFactory.createSopt(Sopt.Part.SERVER);
        Sopt ios = soptFactory.createSopt(Sopt.Part.IOS);
        Sopt android = soptFactory.createSopt(Sopt.Part.ANDROID); // 추가

        server.printLanguage();
        ios.printLanguage();         
        android.printLanguage(); // 추가
    }
}
```

문제 없어보이는데 어디서 OCP를 위반한다는거지?

```java
public interface Sopt {
    enum Part {
        SERVER, IOS, ANDROID   // 추기
    }
    
    void printLanguage();
}

public class Server implements Sopt {
	  @Override
    public void printLanguage() {
        System.out.println("Java");
    }
}

public class Ios implements Sopt {
	  @Override
    public void printLanguage() {
        System.out.println("Swift");
    }
}

public class Android implements Sopt { // 추가
    @Override
    public void printLanguage() {
        System.out.println("Kotlin");
    }
}
```

Android 클래스를 새롭게 추가했지만,

기존 **`Sopt.Part`** 열거형에 새로운 타입을 추가해야 하네 😭

```java
public class SoptFactory {
    public Sopt createSopt(Sopt.Part soptPart) {
        switch (soptPart) {
            case SERVER:
                return new Server();
            case IOS:
                return new Ios();
            case ANDROID:
                return new Android(); 
            default:
                throw new IllegalArgumentException("Sopt에 없는 파트입니다");
        }
    }
} 
```

**`SoptFactory`**의 **`create`** 메서드에 새로운 **`case`**를 추가해야 하네  😭

이렇듯 OCP를 위반하게 된다. 

그러면 OCP를 위반하지 않고 팩토리 패턴을 사용하는 방법이 없을까?

팩토리 메서드 패턴이나 추상 팩토리 패턴을 활용한다면 기존 클래스에 영향을 주지 않고 확장이 가능하다 👍
본론으로 들어가보자 👨🏻‍💻

## Factory Method - 본론

다시 한 번 리마인드 하자면, Factory Method 패턴이란 **클라이언트에서 직접 new 연산자를 통해 제품 객체를 생성하는 것이 아니라 대신 제품 객체를 생성할 공장 클래스를 만들고(여기까진 Simple Factory 와 동일)**, **이를 상속하는 서브 공장 클래스의 메서드에서 여러가지 제품 객체 생성을 각각 책임지는 것을 말한다.(이 부분이 차이점!)**

### Factory Method 구조

<img width="708" alt="123" src="https://github.com/NOW-SOPT-SERVER/hoonyworld/assets/125895298/49d9f8bc-67cd-4c63-8843-d59999a6cd3e">

- Creator : 최상위 공장 클래스로서, 팩토리 메서드를 추상화하여 서브 클래스로 하여금 구현하도록 함.
    - 객체 생성 처리 메서드(someOperation) : 객체 생성에 관한 전처리, 후 처리를 템플릿화한 메서드
    - 팩토리 메서드(createProduct) : 서브 공장 클래스에서 재정의할 객체 생성 추상 메서드
- ConcreteCreator : 각 서브 공장 클래스들은 이에 맞는 제품 객체를 반환하도록 생성 추상 메서드를 재정의한다. 즉, 제품 객체 하나당 그에 걸맞는 생산 공장 객체가 위치된다.
- Product : 제품 구현체를 추상화한 것
- ConcreateProduct : 제품 구현체

아까 보았던 Simple Factory의 예시에 대입해 보자면 다음과 같다.

- Product는 Sopt
- ConcreateProduct는 Server, Ios
- Creator는 SoptFactory(서브 클래스로 하여금 구현하도록 하지 않았을 때)

이제 할 일은 팩토리 메서드를 추상화하여 서브 클래스로 하여금 구현하는 것이다.

### Example - 팩토리 패턴

```java
public interface Sopt {
    void printLanguage();
}

public class Server implements Sopt {
    @Override
    public void printLanguage() {
        System.out.println("Java");
    }
}

public class Ios implements Sopt {
    @Override
    public void printLanguage() {
        System.out.println("Swift");
    }
}
```

인터페이스 Product(Sopt)와 구현 클래스 ConcreateProduct(Server, Ios)이다.
Simple Factory와 동일하다.

```java
// 공장 객체 추상화 (추상 클래스)
public abstract class SoptFactory {

    // 객체 생성 처리 메서드(final로 오버라이딩 방지)
    public final Sopt createOperation() {
        Sopt sopt = createSopt(); // 서브 클래스에서 구체화한 팩토리 메서드 실행
        sopt.setup();  // .. 이밖의 객체 생성에 가미할 로직 실행
        return sopt;  // 완성된 객체 반환
    }

    // 팩토리 메소드 : 구체적인 객체 생성 종류는 각 서브 클래스에 위임
    // protected 이기 때문에 같은 패키지 내의 클래스거나, 해당 클래스를 상속받은 자식클래스에서만 접근 가능
    protected abstract Sopt createSopt();
}

// 서버를 생성하는 구체 팩토리
public class ServerFactory extends SoptFactory {
    @Override
    public Sopt createSopt() {
        return new Server();
    }
}

// iOS를 생성하는 구체 팩토리
public class IosFactory extends SoptFactory {
    @Override
    public Sopt createSopt() {
        return new Ios();
    }
}
```

~~우와 너무 어려운걸…?~~

차근차근 어떤 점이 변했는지 살펴보자!

1. `SoptFactory`가 추상 클래스로 변환되었다.
2. 객체 생성 처리 메서드(`createOperation`**)**가 생겼다.
3. 팩토리 메서드인 `createSopt`가 추상 메서드로 그리고 `protected`로 접근제어자가 바뀌었다.

첫번째, SoptFactory가 추상 클래스로 변환된 이유는

구체적인 객체 생성 로직을 책임을 서브 클래스로 넘기기 위해서 입니다. 이러한 접근 방식은 객체 생성의 구체적인 세부사항을 숨기면서도 필요에 따라 확장할 수 있는 유연성을 제공한다.

두번째, 객체 생성 처리 메서드(createOperation)가 생긴 이유는 

객체 생성 과정에서 공통된 사항(ex. 객체 생성 후 설정, 위의 sopt.setup())을 처리하고, 실제 객체 생성은 createSopt 추상 메서드(팩토리 메서드)에 위임하기 위해서이다.

세번째, 팩토리 메서드인 createSopt가 추상 메서드이자 protected로 접근제어자가 설정된 이유는

추상 메서드로 선언함으로써 모든 서브 클래스가 이 팩토리 메서드를 구현하도록 요구할 수 있고, 

protected로 설정함으로서 서브 클래스와 같은 패키지의 클래스에 의해서만 접근될 수 있으며, API 사용자는 createSopt의 존재 자체를 알려주지 않고 createOperation 공개 메서드를 통해 객체를 요청하게 된다는 장점이 있다. 

```java
public class Client {
    public static void main(String[] args) {
        // 각 타입의 팩토리 인스턴스 생성
        SoptFactory serverFactory = new ServerFactory();
        SoptFactory iosFactory = new IosFactory();

        // 서버 객체 생성 및 사용
        Sopt server = serverFactory.createOperation();
        server.printLanguage();

        // iOS 객체 생성 및 사용
        Sopt ios = iosFactory.createOperation();
        ios.printLanguage();
    }
}
```

클라이언트 코드이다.

클라이언트 코드에서 Server와 Ios 클래스에 대한 의존성 없이 사용이 가능하다.

의존성 주입을 사용해서 외부에서 Factory 클래스를 받아온다면 ServerFactory(), IosFactory() 에 대한 의존성도 제거가 가능하다.

### Simple Factory 한계 극복

팩토리 메서드 패턴의 장점은 확장할 때 기존 코드의 변경이 없어도 된다는 점이라고 앞서 말했다.

이번에도 Android 파트가 추가된다고 가정하자

Simple Factory에서는 SoptFactory의 create 메서드를 수정해야 했다.

깜빡하고 switch-case를 넣지 않았다면, 코드에 오류가 발생하고 Enum에 설정해 놓았던 IllegalArgumentException로 어느정도 방어할 수는 있지만, 수정에도 열려있다는 단점은 변하지 않는다.

**하지만 우리는 Factory Method 패턴을 적용했기 때문에, 기존 SoptFactory의 수정 없이 새로운 코드를 추가만 하면 된다!!**

```java
public interface Sopt {
    void printLanguage();
}

public class Server implements Sopt {
    @Override
    public void printLanguage() {
        System.out.println("Java");
    }
}

public class Ios implements Sopt {
    @Override
    public void printLanguage() {
        System.out.println("Swift");
    }
}

public class Android implements Sopt { // 추가
    @Override
    public void printLanguage() {
        System.out.println("Kotlin");
    }
}
```

Sopt 인터페이스를 구현하는 Android 클래스 추가

```java
public abstract class SoptFactory {

    public final Sopt createOperation() {
        Sopt sopt = createSopt(); // 서브 클래스에서 구체화한 팩토리 메서드 실행
        sopt.setup();  // .. 이밖의 객체 생성에 가미할 로직 실행
        return sopt;  // 완성된 객체 반환
    }

    protected abstract Sopt createSopt();
}

public class ServerFactory extends SoptFactory {
    @Override
    public Sopt createSopt() {
        return new Server();
    }
}

public class IosFactory extends SoptFactory {
    @Override
    public Sopt createSopt() {
        return new Ios();
    }
}

public class AndroidFactory extends SoptFactory { // 추가
    @Override
    protected Sopt createSopt() {
        return new Android();
    }
}
```

Android 객체를 생성하는 AndroidFactory 정의

```java
public class Client {
    public static void main(String[] args) {
        // 각 타입의 팩토리 인스턴스 생성
        SoptFactory serverFactory = new ServerFactory();
        SoptFactory iosFactory = new IosFactory();
        SoptFactory androidFactory = new AndroidFactory();

        // 서버 객체 생성 및 사용
        Sopt server = serverFactory.createOperation();
        server.printLanguage();

        // iOS 객체 생성 및 사용
        Sopt ios = iosFactory.createOperation();
        ios.printLanguage();
        
        // Andorid 객체 생성 및 사용
        Sopt android = androidFactory.createOperation();
        android.printLanguage();
    }
}
```

이렇게 최상위 공장 클래스의 코드 수정 없이 새로운 객체 타입을 추가하기 위해 새로운 팩토리 클래스만 추가하면 되고 기존 코드를 변경할 필요가 없다. 이는 OCP를 잘 준수한다고 볼 수 있다.

![factory_3](https://github.com/NOW-SOPT-SERVER/hoonyworld/assets/125895298/23e69b15-e565-4e17-9698-e66135b8667d)

이렇게 의존관계를 갖게 된다.

## Factory Method - 장단점

- 장점: Factory Method 패턴의 가장 큰 장점은 지금까지 본 것처럼 **수정에 닫혀있고 확장에는 열려있는 OCP 원칙을 지킬 수 있다는 점이다.**
- 단점: 간단한 기능을 사용할 때보다 많은 클래스를 정의해야 하기 때문에 코드량이 증가한다.
