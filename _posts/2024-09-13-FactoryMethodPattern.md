---
title: 팩토리 메서드(Factory Method) 
date: 2024-09-13 
categories: [Design Pattern, 생성패턴]
tags: [Design Pattern]
---

# 디자인 패턴 (Design Pattern)
디자인 패턴은 Gang Of Four에 의해 널리 알려지게 되었으며,  
객체지향 프로그래밍에서 객체 생성 및 상호 작용 시 발생하는 문제에 대한 일반적이고 재사용 가능한 솔루션이자 템플릿이다.

정리를 하기 위해 블로그를 탐방하다 보니 아주 좋은 말을 하나 발견했다.  
예전에 처음 학습하며, 내가 저질렀던 행동들을 관통하는 말이다.  

"망치를 들면, 모든 것이 못으로 보인다."  

나와 같은 뉴비 프로그래머들은 자주 경험 했을 법한 상황이다.  
새롭게 학습한 것을 어떻게든 써먹어보기 위해 튀어나온 못이 아닌데 망치를 두드리고 있는 상황이었다.  
  
제대로 튀어나온 못에 망치질을 하기 위해선 해당 패턴은 어떤 문제를 위한 솔루션인지,  
생기는 장점과 단점이 어떠한 것인지 정확히 아는 것이 중요할 것 같다.  
  
디자인 패턴을 정리하며 패턴마다 장점과 단점을 명확하게 파악해보자.  

## 팩토리 메서드(Factory Method)
>팩토리 메서드 패턴은 객체 생성을 처리하는 인터페이스를 정의하고, 이를 하위 클래스에서  
구체적인 객체 생성에 활용하는 패턴이다. 즉, 객체를 생성하기 위한 추상화된 팩토리 메서드를 정의하고,  
실제 객체 생성은 하위 클래스에서 처리한다.   
  
우리는 치킨을 너무 사랑한 나머지 단골 치킨집 사장님과 협업하여, 치킨 주문 앱을 만들었다.  
치킨 메뉴의 근본 메뉴인 후라이드 치킨과 양념 치킨을 앱을 통해 주문을 할 수 있게 되었다.  

온라인과 오프라인에서 동시에 주문을 받게 되었다.      
매출이 2배가 되어버린 사장님은 기쁨을 감추지 못하고, 신 메뉴 개발에 착수한다.    
사장님은 트렌드를 잘 읽어내는 사람이었고, 두바이 초콜릿치킨 개발에 성공한다.  
  
사장님은 앱에서도 두바이초콜릿치킨을 주문 받을 수 있게 요청하였다.  
  
## 기존의 치킨 주문 앱 코드  
```swift
class FriedChicken {
    func eat() {
        print("냠냠 맛있는 후라이드치킨.")
    }        
}

class KoreanFriedChicken {
    func eat() {
        print("냠냠 맛있는양념치킨.")
    }
}

// Client
func orderChicken() -> KoreanFriedChicken {
    return KoreanFriedChicken()
}

let chicken = orderChicken()
chicken.eat()
```          

> 문제: 우리는 각 메뉴별로 클라이언트 코드에서 직접 객체를 생성해서 주문을 받고 있다.  
이에 따라 신메뉴가 추가될 때마다 늘 클라이언트 코드는 계속해서 코드 추가와 기능이 수정되면 변경이 일어나야한다.  
이는 코드의 확장성을 저해시키며, 유지보수를 어렵게 만든다.  
{: .prompt-warning }

팩토리 메서드 패턴을 통해 코드를 리팩토링해보자.  
  
## 패턴을 적용한 리팩토링 코드
```swift 

// Product Interface
protocol Chicken {
    func eat()
}

// Creator 
protocol ChickenFactory {
    func createChicken() -> Chicken
}

// Concrete Product
class FriedChicken: Chicken {
    func eat() {
        print("맛있는 후라이드치킨이 완성 되었습니다.")
    }
}

class KoreanFriedChicken: Chicken {
    func eat() {
        print("맛있는 양념치킨이 완성 되었습니다.")
    }
}

class DubaiChocolateChicken: Chicken {
    func eat() {
        print("신상 두바이초콜렛치킨이 완성 되었습니다.")
    }
}

// Factory (Concrete Creator)
class FriedChickenFactory: ChickenFactory {
    func createChicken() -> Chicken {
        return FriedChicken()
    }
}

class KoreanFriedChickenFactory: ChickenFactory {
    func createChicken() -> Chicken {
        return KoreanFriedChicken()
    }
}

class DubaiChocolateChickenFactory: ChickenFactory {
    func createChicken() -> Chicken {
        return DubaiChocolateChicken()
    }
}

// Client
func orderChicken(factory: ChickenFactory) -> Chicken {
    return factory.createChicken()
}

// 기존의 메뉴 주문
let friedChickenFactory = FriedChickenFactory()
let friedChicken = orderChicken(factory: friedChickenFactory)

let koreanFriedChickenFactory = KoreanFriedChickenFactory()
let koreanFriedChicken = orderChicken(factory: koreanFriedChickenFactory)

// 추가된 신메뉴 주문
let dubaiChocolateChickenFactory = DubaiChocolateChickenFactory()
let dubaiChocolateChicken = orderChicken(factory: dubaiChocolateChickenFactory)
dubaiChocolateChicken.eat()
```

클라이언트쪽에서 사용할 때 직접적인 객체 생성을 하지 않고 팩토리 메서드 패턴을 적용하여   
팩토리 클래스에 위임함으로써, 새로운 메뉴를 추가할 때 클라이언트의 코드의 변경없이 새로운 팩토리 클래스를 추가하는 방식으로   문제를 해결할 수 있게 되었다.  
  
하지만 현재 팩토리 별로 동일한 코드가 반복되기 때문에 또 한번 아래처럼 enum을 활용해 리팩토링을 할 수 있을 것이다.  
enum을 활용하게 되면 case 추가로만 확장을 쉽게 할 수도 있을 것 같다.  

```swift
// ProductType
enum ChickenType {
    case fried
    case koreanFried
    case dubaiChocolate
}

// Factory (Concrete Creator)
class MainChickenFactory: ChickenFactory {
    func createChicken(type: ChickenType) -> Chicken {
        switch type {
        case .fried:
            return FriedChicken()
        case .koreanFried:
            return KoreanFriedChicken()
        case .dubaiChocolate:
            return DubaiChocolateChicken()
        }
    }
}

// Client
func orderChicken(factory: ChickenFactory, type: ChickenType) -> Chicken {
    return factory.createChicken(type: type)
}

let chickenFactory = MainChickenFactory()
let friedChicken = orderChicken(factory: chickenFactory, type: .fried)
let koreanFriedChicken = orderChicken(factory: chickenFactory, type: .koreanFried)
let dubaiChocolateChicken = orderChicken(factory: chickenFactory, type: .dubaiChocolate)
dubaiChocolateChicken.eat()
```

팩토리 메서드 패턴의 핵심은 클라이언트 코드에서 직접 객체를 생성하지 않고, 하위 클래스를 통해 객체를 생성한다는 것이다.  
또한 하위 클래스를 통해 객체를 생성하기 때문에 객체 생성이 캡슐화되어 있기 때문에 동일한 주문은 공장에서 찍어낸 제품(Product)처럼 제품의 불변성을 보장한다는 것이다.   

## 패턴의 장점과 단점
  
> 장점
{: .prompt-tip }

- 클라이언트에선 객체의 구체적인 생성 로직이 분리되어 캡슐화된다. 객체가 어떻게 생성되는 지 신경 쓸 필요 없이,
인터페이스만을 통해 객체를 생성할 수 있다.
- 새로운 객체 생성 로직이 필요할 때 기존의 코드를 변경하지 않고도 새로운 하위 클래스또는 케이스만 추가하면 쉽게 확장할 수 있다. 기존의 코드의 수정없이 확장이 가능하기 때문에 유지 보수성이 높아진다.
- 프로토콜을 활용함으로써, 객체 생성 로직 사이의 결합도를 낮추어 유연성을 증가 시킨다. 클라이언트 코드에는 구체적인 객체에 대한 정보를 알 필요가 없기 때문에, 변화에 덜 민감해진다.

> 단점
{: .prompt-warning}

- 클래스 수와 프로토콜의 수가 증가하게 된다.이로 인해 관리 해야할 코드의 양이 증가하게 된다.
- 새로운 제품군이 추가되거나 기능이 추가되면 클라이언트 코드에서도 많은 수정이 일어날 수도 있다. 미리 확장에 용이하게 초반부터 확장에 대비해 코드를 설계하지 않을 시에 복잡성이 증가할 수 있다. 

문제에 올바른 패턴 사용으로 유연하게 대처할 수 있기를 바란다.
