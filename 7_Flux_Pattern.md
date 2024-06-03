## 1. Flux 패턴이 뭔가요?

FaceBook 이 React 를 구축할때 소개한 아키텍처로 데이터의 흐름을 예측 가능하고 이해하기 쉽게 만들어주는 디자인 패턴

<br>

## 2. 왜 등장했을까요?

대규모 어플리케이션에서 데이터 흐름을 파악하기가 어렵다!
데이터 흐름을 일관성 있게 관리하여 예측 가능성을 높이기 위함!

<br>

### 2-1-1. 기존 방식 MVC

<img width="700" alt="original" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/529215f2-e64d-4694-b419-a4e105a6c6a2">

- Model 의 데이터가 변경되면 View 로 전달하여 화면에 출력
- View 에서 데이터를 입력받아 Model 을 업데이트

<br>

<img width="700" alt="original" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/7f57c9e1-c40e-4882-8c8a-109693fa96ba">

- iOS 의 UIKit 은 ViewController 를 중심으로 한 패턴 ( MVC )
- ViewController 가 View lifeCycle 을 직접 관리
- ViewController 가 너무 할일이 많아진다… ( 네트워크 요청, DB 데이터 요청, delegate, datasource )

<br>

> 💡 즉, 데이터가 양방향으로 흐르게 된다!

<br>

<img width="700" alt="original" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/35424aa1-7393-4859-9270-bb7be712103a">

- 하나의 ViewController 이 많은 Model 과 View 를 관리

- **복잡한 예시 가정**
    - Model 데이터 변경 → View 업데이트 → 또 다른 Model 을 업데이트
    - 서로 복잡하고 강한 의존성을 가지게 될 가능성 높아짐
    - 이러한 의존성이 많아질수록 어떠한 이벤트가 발생했을 때 어떠한 상황이 벌어질지 예측 불가
 
<br>

### 2-1-2. 기존 방식 MVVM

<img width="700" alt="original" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/4ced8781-cc4a-4c28-9635-77e84d230583">

- MVC 의 단점이였던 ViewController 의 역할을 ViewModel 로 분리 ( 네트워크 요청 등 )
- ViewModel 은 View 의 이벤트를 감지하여 알맞는 비즈니스 로직을 수행
- Model 의 데이터를 가져오거나 업데이트 하고 View 에 데이터를 업데이트 하여 화면 출력

<br>

> 💡 MVVM 도 여전히 View 와 ViewModel 이 서로 데이터 바인딩이 되어있어 양방향 흐름을 갖고 있다!

<br>

## 3. 단방향 데이터 흐름, Flux

<img width="700" alt="original" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/b699f7e4-26c0-478d-accd-af26732c5dc2">
<img width="700" alt="original" src="https://github.com/SOPT-Design-Pattern-Study/DalDesign-Pattern-Study/assets/45564605/7b80d1e9-7ae7-4100-9ba8-dcc54fcd3389">

- 사용자의 입력을 기반으로 Action 을 구현
- Action 을 Dispatcher 에 전달하여 Store (Model) 을 변경
- View 에 변경된 데이터를 반영

<br>

- **Dispatcher**
    - Store 가 등록해 놓은 Action 타입마다 호출할 콜백 함수들이 존재
    - Action 을 전달받으면 해당하는 Store 가 Action 타입에 맞는 콜백 함수를 실행
    - Store 의 데이터를 변경하는 작업은 오직 Dispatcher 를 통해 가능

- **Store (Model)**
    - 상태 저장소로 상태를 변경할 수 있는 메서드를 갖고 있음
    - 어떤 Action 이 발생했는지에 따라 해당하는 데이터 변경을 수행하는 콜백함수를 Dispatcher 에 등록
    - 만약 데이터가 변경되면 View 에 데이터가 변경되었음을 알림

<br>

## 4.  Swift 에서는 TCA!

> Ex. 증감 버튼이 있는 Counter App

- TCA: Reducer , Flux: Dispatcher
``` swift
import ComposableArchitecture

@Reducer
struct CounterFeature {
  //...
  
  enum Action {
    case decrementButtonTapped
    case incrementButtonTapped
  }
  
  var body: some ReducerOf<Self> {
    Reduce { state, action in
      switch action {
      case .decrementButtonTapped:
        state.count -= 1
        return .none
        
      case .incrementButtonTapped:
        state.count += 1
        return .none
			}
		}
  }
}
```

<br>

- TCA, Flux: Store

```swift
import ComposableArchitecture

@Reducer
struct CounterFeature {

  @ObservableState
  struct State {
    var count = 0
  }
  
  // ...
  
  var body: some ReducerOf<Self> {
  // ...
  }
}
```

<br>

- TCA, Flux: View

```swift
struct CounterView: View {
  let store: StoreOf<CounterFeature>
  
  var body: some View {
    VStack {
      Text("\(store.count)")
			
      HStack {
        Button("-") {
          store.send(.decrementButtonTapped)
        }
        
        Button("+") {
          store.send(.incrementButtonTapped)
        }
      }
    }
  }
}
```
