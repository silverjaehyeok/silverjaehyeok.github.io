---
title: 빌더(Builder) 
date: 2024-09-17 
categories: [Design Pattern, 생성패턴]
tags: [Builder Pattern]
---
### 정의
> 빌더 패턴은 객체들을 단계별로 생성할 수 있도록 하는 생성 디자인 패턴이다. 빌더 패턴의 사용 목적은 복잡한 객체를 단계별로 생성하는 데 있다. 이 패턴은 객체 생성을 일련의 단계들로 정리하며, 객체를 생성하고 싶으면 위 단계들을 builder(빌더) 객체를 사용하면 된다. 중요한 점은 모든 단계를 호출할 필요가 없으며, 객체의 특정 설정을 제작하는 데 필요한 단계들만 호출하면 된다.

### 구조
1. Product(제품): 빌더가 생성할 복합 객체를 나타내는 클래스이다.
2. Builder(빌더): 제품을 구성하는 인터페이스로, 제품 생성에 필요한 메서드를 정의한다.
3. Concrete Builder(구체적인 빌더): Builder 인터페이스를 구현하며, 제품의 구성과 생성 과정을 구체화한다. 단계별로 제품을 만들기 위한 메서드를 구현한다.
4. Director(감독자): Builder 객체를 이용해 제품의 생성 과정을 지시하는 클래스이다. 감독자는 특정한 순서와 방식으로 빌더의 메서드를 호출하여 제품을 완성한다. 필요에 의해 Director는 사용할 수도 있고, 생략할 수도 있다.

### 사례
우리는 서버와 통신을 하기 위해 Request를 만들고 URLSession을 활용해서 통신을 하게 된다.  
  
### 기존코드  

```swift
// 네트워크 요청을 보내는 함수
func fetchData(from url: String, headers: [String: String]?, queryParameters: [String: String]?, completion: @escaping (Data?, URLResponse?, Error?) -> Void) {
    // 기본 URL 설정
    var urlComponents = URLComponents(string: url)
    
    // 쿼리 파라미터 추가
    if let queryParameters = queryParameters {
        urlComponents?.queryItems = queryParameters.map { URLQueryItem(name: $0.key, value: $0.value) }
    }
    
    // URL 생성
    guard let finalURL = urlComponents?.url else {
        completion(nil, nil, NSError(domain: "Invalid URL", code: 0, userInfo: nil))
        return
    }
    
    var request = URLRequest(url: finalURL)
    request.httpMethod = "GET"
    
    // 헤더 추가
    if let headers = headers {
        for (key, value) in headers {
            request.addValue(value, forHTTPHeaderField: key)
        }
    }
    
    // 네트워크 요청
    URLSession.shared.dataTask(with: request) { data, response, error in
        completion(data, response, error)
    }.resume()
}

// 사용 예제
fetchData(from: "https://api.example.com/data", headers: ["Authorization": "Bearer token"], queryParameters: ["page": "1"]) { data, response, error in
    // 데이터 처리
}
```
  
기존 코드를 살펴보면 현재 통신을 하는 함수 내부에서 복잡한 Request를 하나하나 다 설정해서 Request를 생성하고 있다. 또 Request를 설정하기 위한 파라미터들을 설정하고 인자값으로 전달 받고 있다. 이렇게 작성할 경우 유지보수가 상당히 어려워지는 단점이 생긴다.  
  
수정이 일어난다면, 해당 메서드를 호출하는 클라이언트 코드와 fetchData 메서드 두 곳 다 수정을 해야하며, HttpMethod 부분만 확인 해봐도 'GET'으로 고정이 되어있기 때문에 네트워킹 코드의 재사용성이 정말 좋지 않다는 것을 느낄 수 있다.  

이러한 복잡한 객체 생성에 활용하는 빌더 패턴을 적용해 리팩토링 해보자.

### 빌더패턴 적용 리팩토링 코드
```swift 
// 네트워크 빌더 인터페이스
protocol NetworkBuilderProtocol {
    var url: String? { get }
    var headers: [String: String] { get }
    var queryParameters: [String: String] { get }
    var httpMethod: String { get }
}

// 네트워크 요청을 위한 빌더 클래스
struct NetworkBuilder: NetworkBuilderProtocol {
    var url: String?
    var headers: [String: String] = [:]
    var queryParameters: [String: String] = [:]
    var httpMethod: String
}

class NetworkManager {
    // 빌더 패턴을 이용한 네트워크 요청 함수
    func fetchData(using builder: NetworkBuilder, completion: @escaping (Data?, URLResponse?, Error?) -> Void) {
        guard let request = makeRequest(builder) else {
            completion(nil, nil, NSError(domain: "Invalid Request", code: 0, userInfo: nil))
            return
        }
        
        // 네트워크 요청
        URLSession.shared.dataTask(with: request) { data, response, error in
            completion(data, response, error)
        }.resume()
    }
    
    func makeRequest(_ builder: NetworkBuilderProtocol) -> URLRequest? {
        guard let urlString = builder.url, var urlComponents = URLComponents(string: urlString) else {
            return nil
        }
        
        // 쿼리 파라미터 추가
        if !builder.queryParameters.isEmpty {
            urlComponents.queryItems = builder.queryParameters.map { URLQueryItem(name: $0.key, value: $0.value) }
        }
        
        // 최종 URL 생성
        guard let finalURL = urlComponents.url else {
            return nil
        }
        
        // URLRequest 생성
        var request = URLRequest(url: finalURL)
        request.httpMethod = builder.httpMethod
        
        // 헤더 추가
        for (key, value) in builder.headers {
            request.addValue(value, forHTTPHeaderField: key)
        }
        
        return request
    }
}
```
  
위처럼 빌더 패턴을 사용하게 되는 경우에는 복잡한 Request의 객체의 생성 과정을 단계적으로 나눈 것을 빌더에게 위임하고 있다. NetworkManager에서 Request를 만드는 부분을 살펴보면 빌더를 이용해서 복잡한 Request를 단지 필요한 부분만을 가져와서 손쉽게 조립해서 사용하고 있는 것을 확인할 수 있다. 빌더 내부의 BaseURL이 수정 되어도 NetworkManager 쪽 코드의 수정은 일어나지 않게 된다.

>장점
{: .prompt-tip }
- 복잡한 객체의 생성 과정을 나누어 설정할 수 있어 유연성이 높아진다. 또한 필요한 설정만 선택적으로 적용할 수 있다.
- 가독성과 유지보수성이 높아지는데, 객체 생성 코드가 흐름에 따라 작성되는 경우가 많기 때문에 가독성이 높다, 생성 과정이 분리 되어 있으므로, 유지보수가 용이하다.
- 단일 책임 원칙을 준수한다. 객체 생성을 분리함으로써, 하나의 책임만을 가지게 되어 코드 구조가 깔끔해짐.

>단점
{: .prompt-warning}
- 빌더가 만들어 질 때마다 인터페이스와 구체적인 빌더 파일이 계속 생성되므로 코드 복잡성이 증가한다.
- 빌더 패턴은 객체를 생성하기 전에 빌더 인스턴스를 먼저 생성해야 하므로, 객체 생성의 초기 비용이 증가 할 수도 있다.
- 객체 생성 과정이 빌더로 고정이 되기 때문에 오히려 유연성을 제한 할 수 도 있다.
