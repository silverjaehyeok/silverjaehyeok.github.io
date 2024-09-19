---
title: 브릿지(Bridge) 
date: 2024-09-20
categories: [Design Pattern, 구조패턴]
tags: [구조패턴]
---
### 정의
클래스의 기능 계층과 구현 계층을 분리하여 둘을 독립적으로 확장할 수 있도록 하는 패턴이다. 이 분리를 통해 클래스 간의 강한 결합을 피하고, 코드의 유연성과 재사용성을 높이는 것을 목표로 한다.  
  
### 구조
1. Abstraction(추상부분): 기능의 추상적인 부분을 정의한다. 이 클래스는 구현 부분에 대한 참조를 가지고 있으며, 이를 통해 기능을 구현한다.
2. Refined Abstraction(구체적인 추상 부분): 추상 부분을 확장한 클래스이다. 여기서 더 구체적인 기능을 정의할 수 있다.
3. Implementor(구현 부분): 기능의 실제 구현을 위한 인터페이스를 정의한다. 이는 일반적으로 추상 메서드들로 구성된다.
4. Concrete Imnplementor(구체적인 구현부분): 구현 부분 인터페이스를 실제로 구현한 클래스이다.
  
### 사례
우리는 메세지 전송 앱을 만들었다.  
메세지를 MessageSender라는 객체를 통해 메세지를 전송한다. 
```swift
class UrgentEmailMessage {
    func send(_ message: String) {
        print("긴급 이메일 전송: [긴급] \(message)")
    }
}

class RegularEmailMessage {
    func send(_ message: String) {
        print("일반 이메일 전송: \(message)")
    }
}

class UrgentSMSMessage {
    func send(_ message: String) {
        print("긴급 SMS 전송: [긴급] \(message)")
    }
}

class RegularSMSMessage {
    func send(_ message: String) {
        print("일반 SMS 전송: \(message)")
    }
```
  
위 코드는 문제가 발생하게 된다. 메세지 유형과 전송 방법이 늘어날 때마다 클래스의 수는 계속 증가하게 될 것이다. 이러한 문제에는 브릿지 패턴이 용이하게 사용될 수 있다.  

```swift
//  Implementor 
protocol MessageSender {
    func sendMessage(_ message: String)
}
// Concrete ImpleMentor
class EmailSender: MessageSender {
    func sendMessage(_ message: String) {
        print("이메일로 메시지 전송: \(message)")
    }
}

class SMSSender: MessageSender {
    func sendMessage(_ message: String) {
        print("SMS로 메시지 전송: \(message)")
    }
}

// Abstraction
class Message {
    let sender: MessageSender
    
    init(sender: MessageSender) {
        self.sender = sender
    }
    
    func send(_ message: String) {
        fatalError("이 메서드는 서브클래스에서 구현해야 합니다.")
    }
}

// Refined Abstration
class UrgentMessage: Message {
    override func send(_ message: String) {
        print("긴급 메시지 전송 준비")
        sender.sendMessage("[긴급] \(message)")
    }
}

class RegularMessage: Message {
    override func send(_ message: String) {
        print("일반 메시지 전송 준비")
        sender.sendMessage(message)
    }
}
```
이렇게 기능계층과 구현계층을 분리하게 되면 확장에서 독립적이게 된다. 즉, 둘 중 하나라도 수정이나 확장이 일어나야 하면 계속 클래스가 추가되는 기존 코드와 달리 해당 계층에서 수정과 확장을 해주면 된다는 말이다.

> 장점
{: .prompt-tip }

- 기능과 구현의 독립적 확장이 가능하다.
- 새로운 기능이나 구현을 추가할 때 기존 코드에 영향을 주지 않으므로 쉽게 확장이 가능하다.
- 코드 중복이 패턴이 적용되기 전보다 확실히 줄어든다.

> 단점 
{: .prompt-warning}

- 계층 구조가 추가되면서 코드의 구조가 다소 복잡해질 수 있다.
- 설계 단계에서 적절히 기능과 구현의 분리를 해야하기 때문에 설계 비용이 증가한다.
