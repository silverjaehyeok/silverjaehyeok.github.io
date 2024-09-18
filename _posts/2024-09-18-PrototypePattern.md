---
title: 프로토타입(Prototype) 
date: 2024-09-18 
categories: [Design Pattern, 생성패턴]
tags: [생성패턴]
---
### 정의
프로토 타입은 코드를 그들의 클래스에 의존시키지 않고 기존 객체들을 복사할 수 있도록 하는 생성 디자인 패턴이다.   
  
### 구조
1. Prototype(원형, 프로토 타입): 복제를 지원하는 인터페이스를 정의한다.
2. ConCrete Prototype(구체적인 프로토타입): 프로토타입 인터페이스를 구현하는 클래스이다. 이 클래스는 객체 복제를 위해 copy() 메서드를 구현한다.
3. Client(클라이언트): 프로토타입 객체를 복제하여 새로운 객체를 생성하는 코드이다. 클라이언트는 복제 과정에서 구체적인 클래스에 의존하지 않으며, 기존의 객체를 복제하여 사용할 수 있다.  
  
### 사례
객체가 있고 그 객체의 정확한 복사본을 만들고 싶다고 가정한다면, 어떻게 해야할까? 먼저 같은 클래스의 새 객체를 생성해야 한다. 그런 다음 원본 객체의 모든 필드들을 살펴본 후 해당 값들을 새 객체에 복사해야한다. 하지만, 객체의 필드들 중 일부가 접근 제어가 private라면, 객체 외부에서 볼 수 없을 수 있으므로 모든 객체는 이러한 방법으로 복사할 수 없다.  
  
직접 접근 방식에는 또 하나의 문제가 존재한다. 복제본을 만드려면 객체의 클래스를 알아야 하므로 코드가 해당 클래스에 종속이 되어버리게 된다. 만약 DIP를 통해 구체적인 타입이 아닌 인터페이스에 의존을 하게 되는 경우 구체적인 클래스를 모르기 때문에 어려움이 생기게 된다.  
  
### 기존코드
```swift
class Monster {
    var name: String
    var attackPower: Int
    private var health: Int
    
    init(name: String, health: Int, attackPower: Int) {
        self.name = name
        self.health = health
        self.attackPower = attackPower
    }
    
}

var monster1 = Monster(name: "goblin", health: 30, attackPower: 15)
var monster2 = Monster(name: monster1.name, health: monster1.health, attackPower: monster1.attackPower)
```
  
기존 코드는 직접 객체를 생성하고 객체를 직접 복제하고 있다. 또한 private로 접근제어가 되어있는 경우 위 코드는 오류를 발생하게 된다. 또한, 구체적인 타입이 아닌 인터페이스를 통해 사용하고 있다면 구체적인 클래스를 알 수 없어 복제하기가 어려워진다.

### 프로토타입 패턴을 적용한 코드
```swift
// Prototype
protocol Prototype {
    init(prototype: Self)
}

extension Prototype {
    func copy() -> Self {
        return type(of: self).init(prototype: self)
    }
}

// Concrete Prototype
class Monster: Prototype {
    var name: String
    var attackPower: Int
    private var health: Int
    
    init(name: String, health: Int, attackPower: Int) {
        self.name = name
        self.health = health
        self.attackPower = attackPower
    }
    
    required convenience init(prototype: Monster) {
        self.init(name: prototype.name, health: prototype.health, attackPower: prototype.attackPower)
    }
}

// Client
var monster1 = Monster(name: "goblin", health: 30, attackPower: 15)
var monster2 = monster1.copy()
```  
  
이렇게 프로토타입 패턴을 적용한 경우 복제가 쉬워지게 된다. 현재 익스텐션으로 기본 구현을 해두었고, 클라이언트 부분에서 그냥 copy() 메서드를 호출하면 객체 복제가 완료되게 된다. 하지만 이렇게 직접적인 타입으로 생성하게 복제하게 되면 클라이언트 코드에서 특정 타입에 대한 의존성이 생기게 되니까 아래 코드처럼 인터페이스로 의존성을 역전시킬 수도 있을 것 같다.  
  
```swift
// Prototype
protocol Prototype {
    func copy() -> Prototype
    func attack()
}

// Concrete Prototype
class Monster: Prototype {
    var name: String
    var attackPower: Int
    private var health: Int
    
    init(name: String, health: Int, attackPower: Int) {
        self.name = name
        self.health = health
        self.attackPower = attackPower
    }
    
    // Prototype 프로토콜의 복제 메서드 구현
    func copy() -> Prototype {
        return Monster(name: self.name, health: self.health, attackPower: self.attackPower)
    }
    
    // 공격 메서드 구현
    func attack() {
        print("\(name)이 공격을 가합니다! 공격력: \(attackPower)")
    }
}

// 객체 생성 및 복제
var monster1 = Monster(name: "Goblin", health: 30, attackPower: 15)
var monster2 = monster1.copy()

// 클라이언트 코드에서 구체적인 클래스를 몰라도 Prototype을 통해 복제된 객체 사용 가능
monster1.attack()
monster2.attack()
```
  
>장점
{: .prompt-tip}
- 패턴의 활용으로 객체 복제가 매우 손쉽게 이루어지게 된다.
- 대량으로 객체를 생성할 때 용이하다. 기존 객체에서 사용하던 것을 그대로 복제해서 사용하기 때문에 효율적일 수 있으며, 성능 향상을 기대할 수도 있다.
- 객체 복제 생성 로직이 캡슐화되어,유지 보수가 용이해진다.  
  
>단점
{: .prompt-warning}
- copy()메서드를 통해 복제될 때 객체가 중첩되어 있거나 복잡한 경우 Deep Copy와 Shallow Copy를 신경써야한다. 위에서 언급하지 않았는데 Swift로 예시를 들자면 구조체처럼 값 타입인 경우 자동으로 복사가 되어 Deep Copy가 일어나지만 클래스나 레퍼런스 타입 같은 경우 Shallow Copy가 일어나게 된다. 따라서 레퍼런스 타입 경우에는 Deep Copy가 일어나야 한다면 신경써서 구현해주어야 한다.
- 복잡한 경우 객체의 관계 파악이 어려워진다.
