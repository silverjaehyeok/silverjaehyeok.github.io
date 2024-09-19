---
title: 어댑터(Adapter) 
date: 2024-09-19 
categories: [Design Pattern, 구조패턴]
tags: [구조패턴]
---
### 정의
구조적 디자인 패턴 중 하나로, 호환되지 않는 인터페이스를 가진 두 클래스가 함께 동작할 수 있도록 중간에 어댑터를 만들어주는 패턴이다. 이 패턴은 기존의 코드를 변경하지 않고, 클래스나 인터페이스를 다른 형식으로 변환하여 재사용성을 높이는 데 도움을 준다.
  
### 구조
1. Target: 클라이언트가 기대하는 인터페이스이다.
2. Adaptee: 클라이언트가 사용하려고 하지만 인터페이스가 호환되지 않는 기존 클래스이다.
3. Adapter: 어댑티의 인터페이스를 타겟의 인터페이스로 변환하는 클래스이다. 이를 통해 클라이언트는 어댑티의 기능을 사용할 수 있음.
4. Client: 특정 인터페이스를 사용하여 작업을 수행하려는 코드이다.

### 사례
우리는 축구 선수들의 골 기록, 실책 등 선수들의 데이터를 사용자에게 제공해주는 앱을 만들었다.  
  
### 기존 코드
```swift
// Adaptee
class XMLFootballDataProvider {
    func getFootballPlayerData() -> String {
        // 예시로 간단한 XML 데이터 생성
        return "Messi total 999 goals"
    }
}
```
기존에는 XML 파일로 데이터를 받아와서 사용자들에게 정보를 제공했지만, 데이터를 전달하면 데이터를 분석해주는 라이브러리를 도입했다. 예를 들어, 공격수의 데이터를 전달하면 해당 선수의 데이터들을 파악해 xG(기대득점)을 도출해준다.  
  
여기서 JSON을 전달받는 라이브러리의 코드를 수정할 수도 없는 노릇이다.  
이러한 문제적 상황에서 어댑터 패턴을 사용한다.  

### 어댑터 패턴을 적용한 리팩토링 코드
```swift
// JSON 데이터를 분석하는 라이브러리
// Client
class JSONFootballAnalyzer {
    func analyzeFootballPlayerData(jsonData: String) {
        print("분석할 JSON 데이터: \(jsonData)")
    }
}

// XML 데이터를 JSON 형식으로 변환하는 어댑터
// Adapter 
class XMLToJSONAdapter {
    private let xmlProvider: XMLFootballDataProvider
    
    init(xmlProvider: XMLFootballDataProvider) {
        self.xmlProvider = xmlProvider
    }
    
    // XML 데이터를 JSON 형식으로 변환
    func getJSONData() -> String {
        let xmlData = xmlProvider.getFootballPlayerData()
        // XML 데이터를 파싱하고 JSON 형식으로 변환
        return jsonData
    }
}

// Target = JSON, XML에서 JSON 

let xmlProvider = XMLFootballDataProvider()
// 어댑터를 통해 XML 데이터를 JSON 형식으로 변환
let adapter = XMLToJSONAdapter(xmlProvider: xmlProvider)
// JSON 분석 라이브러리를 사용해 분석
let jsonAnalyzer = JSONFootballAnalyzer()
let jsonData = adapter.getJSONData()
jsonAnalyzer.analyzeFootballPlayerData(jsonData: jsonData)
```
  
코드를 살펴보면 JsonAnalyzer는 기대하는 타입이 인자값으로 전달되면 문제없이 데이터 분석을 시작하게 된다.  
우리가 흔히 쓰는 어댑터를 떠올려보자. 220V를 사용하지만, 해외 여행 시 110V가 국가 표준인 나라들이 존재한다. 어댑터를 사용하면 기존의 콘센트를 문제없이 사용이 가능하다. 포인트는 기존에 데이터를 변경하지 않고 한 가지 단계를 거친다는 것이다.  

> 장점
{: .prompt-tip }

- 기존의 코드 재사용이 가능해진다. 기존 클래스를 수정하지 않고도 새로운 환경에 맞게 사용이 가능하다.
- 새로운 인터페이스나 클래스가 추가되어도 쉽게 변환이 가능하다. 즉, 유연성이 증가한다.
  

> 단점
{: .prompt-warning}

- 어댑터 클래스가 많아지면 코드의 복잡성이 증가한다.
- 어댑터를 통해 호출하기 때문에 오버헤드로 인해 성능 저하로 이어질 수도 있다.
