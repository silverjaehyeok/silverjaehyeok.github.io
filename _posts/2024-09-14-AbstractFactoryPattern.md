---
title: 추상 팩토리 (Abstract Factory) 
date: 2024-09-14 
categories: [Design Pattern, 생성패턴]
tags: [Design Pattern]
---
> 추상 팩토리(Abstract Factory 패턴은 객체 생성 디자인 패턴 중 하나로, 관련된 객체들의 군(제품군)을 생성하는   
인터페이스를 제공한다. 이 패턴은 구체적은 클래스의 인스턴스를 직접 만들지 않고, 인터페이스를 통해 객체를  
생성할 수 있게 해준다. 추상 팩토리 패턴은 팩토리 메서드 패턴의 확장된 형태이다. 서로 연관된 또는 의존하는  
객체들을 일관되게 생성해야 할 때 유용하다.  
  
### 추상 팩토리의 구조 
1. Abstract Factory: 관련된 객체들을 생성하는 인터페이스를 정의한다.
2. Concrete Factory: 추상 팩토리를 구현하며, 각 제품군에 대한 객체를 생성한다.
3. Abstract Product: 생성될 객체들이 따라야 할 인터페이스 또는 추상 클래스이다.
4. Concrete Product: 추상 제품 인터페이스를 구현하며, 구체적인 객체들을 정의한다.

### 사례 
우리가 E북 리더앱을 하나 만들었는데 UX/UI를 생각해서 라이트 모드와 다크 모드를 지원을 하려고한다.  

### 기존 코드
```swift
// 1. 구체적인 제품 정의
class LightButton {
    func press() {
        print("Light Button Pressed!")
    }
}

class DarkButton {
    func press() {
        print("Dark Button Pressed!")
    }
}

class LightLabel {
    func display() {
        print("Light Label Displayed!")
    }
}

class DarkLabel {
    func display() {
        print("Dark Label Displayed!")
    }
}

// 2. 클라이언트 코드
class Application {
    private var isDarkTheme: Bool

    init(isDarkTheme: Bool) {
        self.isDarkTheme = isDarkTheme
    }

    func createUI() {
        if isDarkTheme {
            let button = DarkButton()
            let label = DarkLabel()

            button.press()
            label.display()
        } else {
            let button = LightButton()
            let label = LightLabel()

            button.press()
            label.display()
        }
    }
}

// 라이트 테마 UI 생성
let lightApp = Application(isDarkTheme: false)
lightApp.createUI()

// 다크 테마 UI 생성
let darkApp = Application(isDarkTheme: true)
darkApp.createUI()
```

앱을 이용하던 고객들이 눈 건강이 걱정된다고 블루라이트 모드 지원을 요청한다.  
기존 코드에서는 조건문을 통해 Bool값에 따라 라이트 모드와 다크모드가 분기가 된다.  
  
> 문제: 새로운 블루 라이트 모드를 도입하려고 하는 경우 클라이언트 코드의 수정이 필요하며 구체적인 객체에 의존하고 있어 유지보수성이 매우 떨어진다.
{: .prompt-warning }  

### 추상 팩토리 패턴을 적용한 코드
```swift
// Abstarct Product
protocol Button {
    func press()
}

protocol Label {
    func display()
}

// Concrete Product
class LightButton: Button {
    func press() {
        print("Light Button Pressed!")
    }
}

class DarkButton: Button {
    func press() {
        print("Dark Button Pressed!")
    }
}

class LightLabel: Label {
    func display() {
        print("Light Label Displayed!")
    }
}

class DarkLabel: Label {
    func display() {
        print("Dark Label Displayed!")
    }
}

// New Theme(Concrete Product)
class BlueLightButton: Button {
    func press() {
        print("Blue Light Button Pressed!")
    }
}

class BlueLightLabel: Label {
    func display() {
        print("Blue Light Label Displayed!")
    }
}

// Abstract Factory
protocol ThemeFactory {
    func createButton() -> Button
    func createLabel() -> Label
}

// Concrete Factory
class LightThemeFactory: ThemeFactory {
    func createButton() -> Button {
        return LightButton()
    }

    func createLabel() -> Label {
        return LightLabel()
    }
}

class DarkThemeFactory: ThemeFactory {
    func createButton() -> Button {
        return DarkButton()
    }

    func createLabel() -> Label {
        return DarkLabel()
    }
}

// New Theme(Factory)
class BlueLightThemeFactory: ThemeFactory {
    func createButton() -> Button {
        return BlueLightButton()
    }

    func createLabel() -> Label {
        return BlueLightLabel()
    }
}

// Client
class Application {
    private var themeFactory: ThemeFactory

    init(factory: ThemeFactory) {
        self.themeFactory = factory
    }

    func createUI() {
        let button = themeFactory.createButton()
        let label = themeFactory.createLabel()

        button.press()
        label.display()
    }
}

// 라이트 테마 UI 생성
let lightFactory = LightThemeFactory()
let lightApp = Application(factory: lightFactory)
lightApp.createUI()

// 다크 테마 UI 생성
let darkFactory = DarkThemeFactory()
let darkApp = Application(factory: darkFactory)
darkApp.createUI()

// 새로운 블루라이트 테마 UI 생성
let blueLightFactory = BlueLightThemeFactory()
let blueLightApp = Application(factory: blueLightFactory)
blueLightApp.createUI()
```
  
위처럼 추상 팩토리 패턴을 적용하게 되면 클라이언트 코드에 더 이상 복잡한 생성 로직이 존재하지 않게 되며, 새로운 테마를 추가하더라도 새로운 팩토리와 연관된 제품들을 추가해주기만 하면 된다. 그리고 클라이언트에서 구체적인 클래스에 의존하지 않기 때문에 결합도가 낮아지며, 코드의 유지보수성을 높이는데 기여할 수 있다.  

### 패턴의 장점과 단점
>장점
{: .prompt-tip }
- 팩토리 메서드의 확장 패턴이며, 일관성 있는 객체를 생성할 수 있게 된다.
- 새로운 팩토리를 추가할 때 클라이언트 코드를 수정할 필요가 없기 때문에 코드가 유연해진다.
- 구체적인 클래스를 의존하지 않고 인터페이스에 의존하기 때문에 유지보수성을 높일 수 있다.  


>단점
{: .prompt-warning }
- 여러 개의 팩토리 메서드와 인터페이스를 도입하기 때문에, 클래스의 수와 인터페이스가 증가하고 코드의 복잡성이
증가한다.
- 팩토리를 통해 객체를 생성하기 때문에 생성 로직이 팩토리 클래스로 고정이 되기 때문에 팩토리에 의존하는 경향이 생긴다.
