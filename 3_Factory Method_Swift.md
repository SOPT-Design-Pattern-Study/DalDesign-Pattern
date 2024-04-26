# Factory Pattern

<aside>
📌 **팩토리 패턴**

> 객체 생성을 하기 위한 인터페이스를 만들고, 실제 인스턴스를 생성하는 구체적인 생성과정은 그 안에 구현하는 패턴
> 

→ **객체의 생성을 팩토리로 모듈화**하여 객체 생성 시 **구체적인 부분이 아닌 추상적인 부분에 의존**할 수 있다.

</aside>

<br>

**사용 목적**

- 객체의 생성을 **캡슐화**하기 위해서
- 객체 생성의 구체적인 구현을 generic한 객체를 사용해 분리

→ 인스턴스의 생성에 비즈니스 로직이 끼어있을 때, 이 로직을 의존성을 가지는 객체로부터 분리할 수 있다

→ 다형성으로 인해 인스턴스를 생성하는 팩토리를 언제든지 유연하게 변경할 수 있다

즉, 인스턴스를 만드는 과정이 변경되어도 의존성을 가진 객체는 수정되지 않고 인스턴스를 만드는 시나리오만 수정하면 된다.

<br>

## Factory Method Pattern

> 객체를 생성하기 위한 인터페이스를 정의하는 과정에서 어떤 클래스의 인스턴스를 만들지 서브클래스에서 결정하는 패턴으로 다시 말해, 클래스의 **인스턴스를 만드는 일을 서브클래스에게 맡긴다.**


<div align="center">
  
<img width="574" alt="image" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/65678579/a1152ef1-806a-44da-8f68-4bdb21d7a850">

<img width="469" alt="image" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/65678579/560977f8-6f27-4370-a647-0e9cf210ecad">

→ 각각의 공장이 단일 제품을 생성하고 있는 팩토리 메소드
</div>

<br>

<div align="center">
<img width="484" alt="image" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/65678579/d14417c3-6899-4b84-9cec-b62d28904a4c">
</div>

```swift
// 공장: 객체를 생성하기 위한 인터페이스
protocol TrumpFactory {
		func makeTrump(with: 문양) -> Trump
}
```

```swift
class PatternTrumpFactory: TrumpFactory {         // TrumpFacotry 채택
		func makeTrump(with: 문양) -> Trump {
				switch(문양) {
				case heart: return HeartCard()
				case diamond: return DiamondCard()
				case clover: return CloverCard()
				case spade: return SpadeCard()
				}
		}
}
```

```swift
protocol Trump { 
   var name: String
}
```

```swift
// Trump 프로토콜 채택

class HeartCard: Trump {...}
class DiamondCard: Trump {...}
class CloverCard: Trump {...}
class SpadeCard: Trump {...}
```

```swift
let patternTrumpFactory = PatternTrumpFactory()
let anyTrump = patternTrumpFactory.makeTrump(with: 만들고 싶은 문양)

```

<br>
<br>


## 구조
<div align="center">
    <img width="602" alt="image" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/65678579/4c7bbfd2-d7cf-4202-b313-ec7c202d449a">
    <img width="603" alt="image" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/65678579/74583475-843a-480f-b03a-b964e6f6b6c5">
    <img width="631" alt="image" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/65678579/45485bc5-0b2f-46f9-a6bf-020c87df1ac5">
</div>

**Abstract Creator**: 팩토리에 대한 프로토콜 제공

**Concrete Creator**: 실제 팩토리 역할

**Concrete Product**: 팩토리에서 만들어진 제품들

**Abstract Product**: 제품들이 채택하고 있는 프로토콜

→ 선언부와 구현부 분리


<br>
<br>
<br>


### 싱글톤 패턴

> **하나의 객체는 하나의 인스턴스만** 가질 수 있도록 보장하고, 이에 대한 **전역적인 접근을 보장**
타입 프로퍼티인 `static` 키워드 사용

```swift

let patternTrumpFactory = PatternTrumpFactory.shared
let anyTrump = patternTrumpFactory.makeTrump(with: 만들고 싶은 문양)

class PatternTrumpFactory: TrumpFactory {         // TrumpFacotry 채택
		
		static let shared = PatternTrumpFactory()     // 싱글톤 객체 생성
		private init() {}                             // init함수 접근 제한자 private
		
		func makeTrump(with: 문양) -> Trump {           // makeTrump 오버라이딩
				switch(문양) {
						case heart: return HeartCard()
						case diamond: return DiamondCard()
						case clover: return CloverCard()
						case spade: return SpadeCard()
				}
		}
}
```

> **카드의 종류가 여러개로 늘어난다면?**
> 
> 
> Trump 프로토콜의 구현체 클래스 생성 + makeTrump()의 분기처리만으로 확장이 가능한 구조
> 
> → SOLID원칙 중 확장에는 열려있어야하나 수정에는 닫혀있어야 하는 **개방-폐쇄 원칙(OCP)를 만족**한다.
> 


<br>
<br>

## 트럼프카드가 알맞은 도구를 들고 있게 하려면?

```swift
let patternTrumpFactory = PatternTrumpFactory.shared
let patternSwordFactory = PatternSwordFactory.shared
let patternPaintFacotry = PatternPaintFactory.shared

let anyTrump = patternTrumpFactory.makeTrump(with: 만들고 싶은 문양)
let anySword = patternSwordFactory.makeSword(with: 알맞은 문양)
let anyPaint = patternPaintFactory.makePaint(with: 알맞은 문양)
```

```swift
class PatternSwordFactory: SwordFactory {         // TrumpFacotry 채택
		
		static let shared = PatternSwordFactory()     // 싱글톤 객체 생성
		private init() {}                             // init함수 접근 제한자 private
		
		func makeSword(with: 문양) -> Sword {           // makeTrump 오버라이딩
				switch(문양) {
						case heart: return HeartSword()
						case diamond: return DiamondSword()
						case clover: return CloverSword()
						case spade: return SpadeSword()
				}
		}
}

protocol SwordFactory {
		func makeSword(with: 문양) -> Sword
}

protocol Sword {}

class HeartSword: Sword {...}

// 마찬가지로 Paint와 관련해서도 protocol과 class가 필요함
```

<br>
<br>

## Abstract Factory Pattern

> 관련된 객체를 여러 개 생성
> 

<div align ="center">
<img width="631" alt="image" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/65678579/425cdb7c-0826-443f-a77c-b4a1696281ba">
</div>

같은 특성을 가지는 객체끼리 생산할 수 있는 팩토리에 대한 메소드를 정의하는 걸 **추상 팩토리**라고 한다!

```swift
protocol AbstractTrumpWithStuffFactory {
		func makeTrump() -> Trump
		func makeSword() -> Sword
		func makePaint() -> Paint
}
```

```swift
class HeartFactory: AbstractTrumpWithStuffFactory {
		func makeTrump() -> Trump {
				return HeartTrump()
		}

		func makeSword() -> Sword {
				return HeartSword()
		}

		func makePaint() -> Paint {
				return HeartPaint()
		}
}

class DiaFactory: AbstractTrumpWithStuffFactory {
		func makeTrump() -> Trump {
				return DiaTrump()
		}

		func makeSword() -> Sword {
				return DiaSword()
		}

		func makePaint() -> Paint {
				return DiaPaint()
		}
}
```

```swift
class TrumpWithStuffFactory {
		static let factory: AbstractTrumpWithStuffFactory {
				// if 문양 == 하트 {
						
				}
				// 카드 무늬별 분기처리
				// return HeartFactory, return DiaFactory ...
		}
}
```

```swift
let trumpFactory = TrumpWithStuffFactory.factory
		
let trump = trumpFactory.trump
let sword = trumpFactory.sword
let paint = trumpFactory.paint

```

<br>
<br>

### 추상 팩토리 vs 팩토리 메소드

<추상 메소드 패턴>: 팩토리 메서드가 여러 개의 다른 객체를 생성

트럼프와 물건 팩토리 → 하트 트럼프와 물건 → 하트 트럼프, 하트 검, 하트 물감

---

<팩토리 메소드 패턴>

트럼프 팩토리 → 트럼프

검 팩토리 → 검

물감 팩토리 → 물감
