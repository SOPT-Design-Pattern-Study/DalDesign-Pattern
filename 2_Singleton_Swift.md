# 싱글톤 패턴이란?

> **특정 용도로 객체를 하나만 생성하여 공용으로 사용하고 싶을 때 사용하는 디자인 패턴입니다.**
> 

클래스가 여러차례 호출되더라도, 인스턴스는 딱 한번만 생성되게 하는 것입니다. 

이렇게 되면 전역변수처럼 코드의 어느 곳에서 접근하던 간에 같은 메모리에 접근이 가능해집니다.

## 햄버거

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2a65dd92-1694-460a-a843-42f41adf38d8/9480db00-b1fb-4662-a468-224f67e2d648/Untitled.png)

스장님처럼 햄버거에 대하여 한 번 비유를 해보겠습니다.

기존에 햄버거 모델이

```swift
class BuggerInfo {
    var bun: String?
    var patty: String?
    var vegatable: String?
}
```

다음과 같이 있고, 3명의 햄버거 제작자가 햄버거를 완성 시키려고 합니다.

---

```swift
let userInfo = BuggerInfo()
userInfo.bun = "모카번"

//bunVC
```

```swift
let userInfo = BuggerInfo()
userInfo.patty = "불고기"

//pattyVC
```

```swift
let userInfo = BuggerInfo()
userInfo.vegatable = "피클"

//vegetableVC
```

- 각각의 담당자들이 인스턴스를 각각 선언하고, 정보를 하나씩 저장하고 있어요!

```swift
class BuggerInfo {
    var bun = "모카번"
    var patty = nil
    var vegatable = nil
}

class BuggerInfo {
    var bun = nil
    var patty = "불고기"
    var vegatable = nil
}

class BuggerInfo {
    var bun = nil
    var patty = nil
    var vegatable = "피클"
}
```

## 하지만 우리가 원하는 건 한 인스턴스에 저장된 햄버거의 정보에요

- 물론 각각의 재료담당자들이 서로 참조하여 데이터를 전달할 수 있지만, 너무나 불편합니다..
(BunVC → PattyVC → VegetableVC)

## 그래서 햄버거매니저를 고용하기로 합니다.

```swift
class BuggerManager {
    static let shared = BugerInfo()
    // 1개만 만들거야

    var bun: String?
    var patty: String?
    var vegatable: String?
    
    private init() {}
    // 나 빼고는 만들지마
}
```

- 이렇게 중앙에 버거매니저를 놓는다면, 각각의 재료 담당자는 서로 다른 인스턴스를 선언할 필요없이 바로바로 정보를 한곳에다 바꿔줄 수 있어요!

---

## 싱글톤 패턴 장단점

### 장점

- **인스턴스가 1번만 생성되기 때문에 메모리 낭비를 방지할 수 있다**

→ 각각 담당자가 BuggerInfo를 생성하는 것 보다, BuggerManager가 1번만 생성해서 Good!

- **전역 인스턴스로 사용할 수 있기 때문에 메모리 및 자원 관리가 쉽다**

→ BuggerManager만 관리해주면 된답니다!

- **메모리를 할당하고 초기화하는 시간이 줄어들어 객체 접근 시간이 줄어든다**

→ 각각 재료 담당자들이 정보를 만들어 주는 것 보다, 바로 버거매니저가 이미 한번 만든 메모리에 다시 접근하면 빠릅니다!

### 단점

- Singleton Instance가 너무 많은 일을 하거나, 많은 데이터를 공유시킬 경우 다른 클래스의 Instance들 간 결합도가 높아져  "개방=폐쇄" 원칙을 위배함 

**→ 싱글턴 인스턴스가 너무 많은 일을 하며 여러 부분과 묶여 있다면?, 새로운 기능을 추가하거나 기존 기능을 수정할 때 다른 부분에도 영향을 미칠 수 있습니다. 

→ 옆동네 롯데리아, 맥도날드, 버거킹이 다 이 매니저를 쓴다면?

→ 맥도날드가 패티 잘못 구우면 줄폐업이다.**
- 멀티 스레드 환경에서 객체가 2개 생성되는 위험이 발생할수 있다

→ 위의 단점에서 언급한것과 같이, 싱글톤 객체는 딱 한번만 생성되어야 하는데 멀티스레드 환경에서 클래스에 동시에 접근할 시 객체가 2개 생성되는 위험이 발생할 수 있습니다. 이는 Thread-Safe 를 보장하지 않습니다.

→ 하지만 Swift에서는 
`static let shared = BugerInfo()`

`static let`을 통하여 1회 생성을 보장 받을 수 있다!

## 사용 예제

1. `UIScreen.main`
2. `URLSession.shared`
3. `NotificationCenter.default`
4. `UserDefaults.standard`

- **사용 이유**
    1. 애플리케이션의 런타임 동안에는 단 하나의 주 화면이 존재하므로, **`UIScreen.main`**은 애플리케이션에서 전역적으로 공유되는 단일 인스턴스입니다.
    2. 여러 곳에서 동일한 네트워크 구성을 사용해야 하므로, 애플리케이션 전체에서 단일 인스턴스를 공유하는 것이 효율적입니다.
    3.  애플리케이션에서 이벤트를 통신하는 데 사용되며, 단일 인스턴스를 통해 모든 이벤트를 중앙 집중식으로 관리하는 것이 효율적입니다.
    4. 애플리케이션의 설정이나 사용자 기본 설정과 같은 간단한 데이터를 저장하고 검색하는 데 사용됩니다. 사용자의 설정을 애플리케이션 전체에서 공유하기 때문에 단일 인스턴스를 사용하는 것이 유용합니다.
    

## 사용 했던 프로젝트

```swift

class SignUpManager {
    static let shared = SignUpManager()
    
    var averageUseTime: String = "1"
    var problem: [String] = ["12"]
    var period: Int = 3
    var goalTime: Int = 3
    var appCode: [Apps] = [Apps(appCode: "#25393", goalTime: 76000),
                           Apps(appCode: "#25391", goalTime: 89000)]
}
```

- 온보딩 과정에서 여러 VC로 부터의 데이터를 저장하기 위해서 SignUpManager라는 싱글톤 객체를 사용했었습니다.

```swift
   func didTapButton() {
        let nextViewController = ProblemSurveyViewController()
        self.navigationController?.pushViewController(nextViewController, animated: false)
        **SignUpManager.shared.averageUseTime = selectedText**
    }
```

- 다음과 같이 접근하여 사용할 수 있었어요!

# 느낀점

- 장단점에 대해 처음으로 생각해볼 수 있었다! 왜 쓰는지 알아서 좋았다.