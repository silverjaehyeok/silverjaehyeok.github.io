---
title: 싱글톤(Singleton) 
date: 2024-09-19 
categories: [Design Pattern, 생성패턴]
tags: [생성패턴]
---
### 정의
싱글톤 패턴은 클래스의 인스턴스가 오직 하나만 생성되도록 보장하고, 그 인스턴스에 접근할 수 있는 전역 접근점을 제공하는 디자인 패턴이다.  이 패턴은 공유 자원 관리나 글로벌 상태 관리가 필요한 경우 유용하게 사용된다.  
  
### 특징
1. 인스턴스의 라이프 사이클이 앱의 라이프 사이클과 동일하다.
2. 클래스 외부에서 인스턴스에 접근할 수 있도록 전역 메서드나 인스턴스를 제공한다.
3. 생성자를 private로 정의하여 외부에서의 인스턴스 생성을 막고, 내부적으로만 관리한다.  

### 사례 
우리는 신선한 과일을 판매하는 앱을 만들었다. 현재 앱에서 재고를 관리하는 FruitStockManager가 존재한다. 하지만, 재고를 조정해야 하는 부분에서 계속 객체를 생성해서 재고를 조정하고 있다.  
  
### 기존 코드
```swift
enum Fruit {
    case apple(Int)
    case orange(Int)
    case banana(Int)
    
    func name() -> String {
        switch self {
        case .apple:
            return "Apple"
        case .orange:
            return "Orange"
        case .banana:
            return "Banana"
        }
    }
    
    var quantity: Int {
            switch self {
            case .apple(let count), .orange(let count), .banana(let count):
                return count
            }
        }
}

// 지속적으로 생성되고 있는 객체 
final class FruitStockManager {
    private var fruitList = ["Apple": 20, "Banan": 10, "Orange": 5]
    
    func changeStock(fruits: [Fruit]) {
        for fruit in fruits {
            var stock = fruitList[fruit.name()]
            guard let nowQuantity = stock else { return print("존재하지 않는 과일입니다.") }
            let changedQuantity = nowQuantity + fruit.quantity
            fruitList[fruit.name()] = changedQuantity
        }
    }
}

// Client
func sell() {
    // ... 재고 판매에 관련된 로직
    let fruitStockManager = FruitStockManager()
    fruitStockManager.changeStock(fruits: [.apple(-5), .banana(-2), .orange(-4)])
}

func buy() {
    // ... 재고 입고에 관련된 로직
    let fruitStockManager = FruitStockManager()
    fruitStockManager.changeStock(fruits: [.apple(3), .banana(10), .orange(0)])
}
```
  
기존 코드를 살펴보면 2가지 문제가 생긴다. FruitStockManager는 앱이 종료될 때까지 존재해야하며, 여러 곳에서 생성해서 다른 상태를 가지고 있으면 안된다. FruitStockManager의 라이프 사이클은 앱의 라이프 사이클과 동일 해야하며, 전역 접근점을 제공해 어느 곳에서 접근해도 동일한 상태를 유지해야 한다는 말이다. 여기서 싱글톤 패턴을 적용하게 되면 위 두 문제는 해결된다.

```swift
final class FruitStockManager {
    // 전역 접근점 제공
    static let shared = FruitStockManager()
    private var fruitList: [String: Int]
    
    //생성자를 private로 정의
    private init(fruitList: [String : Int] = ["Apple": 20, "Banan": 10, "Orange": 5]) {
        self.fruitList = fruitList
    }
    
    func changeStock(fruits: [Fruit]) {
        for fruit in fruits {
            var stock = fruitList[fruit.name()]
            guard let nowQuantity = stock else { return print("존재하지 않는 과일입니다.") }
            let changedQuantity = nowQuantity + fruit.quantity
            fruitList[fruit.name()] = changedQuantity
        }
    }
}

func sell() {
    // ... 재고 판매에 관련된 로직
    FruitStockManager.shared.changeStock(fruits: [.apple(-5), .banana(-2), .orange(-4)])
}

func buy() {
    // ... 재고 입고에 관련된 로직
    FruitStockManager.shared.changeStock(fruits: [.apple(3), .banana(10), .orange(0)])
}
```
  
이로써 하나의 상태를 공유하게 되며, FruitStockManager는 앱의 라이프 사이클동안 메모리에 단 하나만 존재하게 된다.

>장점
{: .prompt-tip}
- 객체를 여러 번 생성하지 않고 단 하나의 인스턴스를 재사용하므로, 메모리와 자원을 절약할 수 있다.
- 전역에 존재하기 때문에 접근이 매우 용이하다.
  
>단점
{: .prompt-warning}
- 테스트의 어려움이 존재한다 동일 한 상태를 공유하기 때문에 원하는 테스트가 어렵고, 모의 객체 주입을 통해 테스트 코드를 작성하는데 싱글톤에 의존하고 있는 코드는 의존성 주입이 힘들다.
- 다른 객체들이 싱글톤에 강하게 의존하게 되면, 코드의 유연성이 떨어지고 결합도가 높아진다.
- 멀티스레드 환경에서 동시성 문제가 발생할 수 있으므로 주의해야 한다.
