## 1. State Pattern 이란?

State Pattern 은 상태가 변경될 때 객체의 동작도 변경될 수 있도록 하는 동작 디자인 패턴의 한 종류 입니다. <br>

> 예를 들자면, 컴퓨터가 켜져있는 상황에서는 전원 버튼의 역할은 off, 꺼져있는 상황에서는 on 이기 때문에 컴퓨터의 상태의 따라 동작(메서드)이 달라지는 것이다!

<br>

<img width="400" alt="original" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/8d4a2804-d5e3-4b1b-b913-84614cf66950">

<br>
<br>

>**Context** - 현재 상태를 갖고 있고 동작이 변경되는 객체, Protocol 의 다형성 특징을 통해 동작을 변경 <br>
>**State** - 모든 상태에 대한 동작, 필요한 Method 와 Property 를 정의 ( Protocol )<br>
>**Concrete State** - State Protocol 을 채택, Context 가 어떻게 동작하는지를 구현

<br>

## 2. 언제 써야 하나요…?

- 런타임 시점에서 객체의 상태가 바뀌면서, 동작이 바뀔 때
- 객체의 상태에 따라 동작에 대한 분기 처리가 많아질 때

<br>

## 3. NetFlix 요금제에 따른 State Pattern 예시

> **상태에 대한 동작** <br>
> 
> 
> **구독이 되어있는지?** →  영상 재생, 다운로드 불가
> **Basic, Standard, Premium 요금제 상태** → 영상 재생, 다운로드 가능, 요금제에 따른 해상도 차등

<br>

``` swift
class NetFlixApp {
    private var isSubscribed: Bool
    private var isBasic: Bool
    private var isStandard: Bool
    private var isPremium: Bool
    
    init(isSubscribed: Bool, isBasic: Bool, isStandard: Bool, isPremium: Bool) {
        self.isSubscribed = isSubscribed
        self.isBasic = isBasic
        self.isStandard = isStandard
        self.isPremium = isPremium
    }

    func changePremiumMembership() {
        self.isBasic = false
        self.isStandard = false
        self.isPremium = true
    }

    func playMovie() {
        if !isSubscribed {
            print("구독 X, 영상 재생 X")
        } else if isSubscribed && isBasic {
            print("구독 O, 영상 재생 O, 720P")
        } else if isSubscribed && isStandard {
            print("구독 O, 영상 재생 O, 1080P")
        } else if isSubscribed && isPremium {
            print("구독 O, 영상 재생 O, 1440P")
        }
    }

    func downMovie() {
        if !isSubscribed {
            print("구독 X, 영상 다운 X")
        } else {
            print("구독 O, 영상 다운 O")
        }
    }
}

let netflixApp = NetFlixApp(isSubscribed: true, isBasic: true, isStandard: false, isPremium: false)

netflixApp.playMovie()
netflixApp.downMovie()

netflixApp.changePremiumMembership()

netflixApp.playMovie()
netflixApp.downMovie()
```

<br>

<img width="652" alt="스크린샷 2024-04-19 오후 3 38 54" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/20f7dcdf-2afd-4ef0-b2dc-ef627bd47ce5">

<br>
<br>

#### ⚠️ 문제점
- 객체 지향적인 방식이 아니다
- 상태에 따른 조건 분기가 나열되어 있어 직관적으로 파악하기 어려움
- 상태를 바꾸는 부분이 노출되어 있음
- 새로운 상태가 생긴다면 모든 부분에 있어서 수정이 필요

<br>

### 3-2. Enum 으로 분기처리 

``` swift
enum MembershipType {
    case unsubscribed
    case basic
    case standard
    case premium

    func play() {
        switch self {
        case .unsubscribed:
            print("구독 X, 영상 재생 X")
        case .basic:
            print("구독 O, 영상 재생 O, 720P")
        case .standard:
            print("구독 O, 영상 재생 O, 1080P")
        case .premium:
            print("구독 O, 영상 재생 O, 1440P")
        }
    }

    func download() {
        switch self {
        case .unsubscribed:
            print("구독 X, 영상 다운 X")
        case .basic, .standard, .premium:
            print("구독 O, 영상 다운 O")
        }
    }
}

class NetFlixApp {
    var membership: MembershipType

    init(membership: MembershipType) {
        self.membership = membership
    }

    func changeMembership() {
        self.membership = .premium
    }

    func playMovie() {
        membership.play()
    }

    func downMovie() {
        membership.download()
    }
}

let netflixApp = NetFlixApp(membership: .unsubscribed)

netflixApp.playMovie()
netflixApp.downMovie()

netflixApp.changeMembership()

netflixApp.playMovie()
netflixApp.downMovie()
```

<img width="652" alt="스크린샷 2024-04-19 오후 3 47 37" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/7dfbf16b-4910-44af-860b-e10b41a4fd28">

<br>
<br>

#### ⚠️ **문제점**

- enum 으로 모든 상태를 관리하는 것은 좋아보이지만 여전히 분기처리 필요
- 상태가 추가될 경우, 기능 수정이 발생하는 경우 수정에 용이하지 않다
- OCP 개방 폐쇄 원칙을 준수하지 못함 ( 확장에는 열려있고 변경에는 닫혀 있어야 한다 )


<br>

### 3-3. State Pattern 적용

``` swift
// State
protocol State {
    func play()
    func download()
}
``` 

``` swift
// ConcreteState
class unSubscribeState: State {
    func play() {
        print("구독 X, 영상 재생 X")
    }
    
    func download() {
        print("구독 X, 영상 다운 X")
    }
}

class BasicState: State {
    func play() {
        print("구독 O, 영상 재생 O, 720P")
    }
    
    func download() {
        print("구독 O, 영상 다운 O, 720P")
    }
}

class StandardState: State {
    func play() {
        print("구독 O, 영상 재생 O, 1080P")
    }
    
    func download() {
        print("구독 O, 영상 다운 O, 1080P")
    }
}

class PremiumState: State {
    func play() {
        print("구독 O, 영상 재생 O, 1440P")
    }
    
    func download() {
        print("구독 O, 영상 다운 O, 1440P")
    }
}
```

``` swift
// Context
class NetFlixApp {
    var membership: State
    
    init(membership: State) {
        self.membership = membership
    }
    
    func changeMembership() {
        self.membership = PremiumState()
    }
    
    func playMovie() {
        membership.play()
    }
    
    func downMovie() {
        membership.download()
    }
}

var myNetFlixApp = NetFlixApp(membership: unSubscribeState())

myNetFlixApp.playMovie()
myNetFlixApp.downMovie()

myNetFlixApp.changeMembership()

myNetFlixApp.playMovie()
myNetFlixApp.downMovie()
```

<img width="652" alt="스크린샷 2024-04-19 오후 3 47 37" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/2a13ab3c-3635-4a54-aa53-ee6824ea9d0d">

<br>

## 4. 정리해보자면~

#### 💡 장점

- 상태에 따른 동작을 개별 객체로 관리 가능 → 복잡도 감소, 응집도 향상
- 단일 책임 원칙 준수 ( 각 상태에 관한 동작 분리 )
- 개방 폐쇄 원칙 준수 ( 기존 코드를 변경하지 않고 새로운 상태를 추가 할 수 있음 )


#### ⚠️ 단점

- 개별 객체를 만들기 때문에 관리해야할 클래스 증가
- 객체가 갖고 있는 상태의 수가 적다면 패턴을 적용하는 것이 비효율적일 수 있음

<br>

### 4-2. 전략 패턴과 차이점

- 두 패턴 모두 동작을 캡슐화하지만 의도가 다르다.
- 전략 패턴은 런타임에 특정 알고리즘을 선택하는 데 중점을 두는 반면, 상태 패턴은 객체의 상태 간 전환을 처리한다.

<br>

### 4-3. State 가 변경될 가능성 vs 관련 동작이 변경될 가능성

<br>

> 💡 **넷플릭스 기능 중 공유하기가 생겼다고 가정해봅시다!**

<br>

- 과연 Enum 방식이 나쁜 것인가..? 라는 고민

<br>

<img width="650" alt="스크린샷 2024-04-19 오후 4 58 58" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/a835095d-b7e8-42d7-8f7a-9e9995c2e0fa">

#### ⚠️ 문제점

- Concrete State 가 많고 State 의 동작 메서드가 적을 경우 수정할 Class 가 많아진다…

<br>

<img width="793" alt="스크린샷 2024-04-19 오후 4 59 49" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/864982a7-f18f-4ee0-9e70-7749a80feb9a">

#### ⚠️ 문제점

- 단일 책임 원칙을 준수할 순 없다….
- 수정할 파일이 나눠져 있지 않고 수정할 부분이 적다!



