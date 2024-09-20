---
title: 컴포짓(Composite) 
date: 2024-09-21
categories: [Design Pattern, 구조패턴]
tags: [구조패턴]
---
### 정의
컴포짓 패턴은 트리 구조를 사용하여 객체들을 구성할 때 유용한 패턴이다. 이 패턴은 클라이언트가 개별 객체와 복합 객체를 동일하게 취급할 수 있도록 설계한다. 즉, 단일 객체와 복합 객체를 같은 방식으로 다룰 수 있도록 하는 패턴이다.  
  
### 구조
1. Component(구성요소): 공통 인터페이스를 정의하여 Leaf와 Composite가 동일한 방식으로 다뤄질 수 있게 한다.
2. Leaf(잎): 트리 구조의 가장 밑단에 있는 개별 객체를 의미함.(기존의 이산수학 내용처럼 자식 노드가 없는 것을 의미.)
3. Composite(복합체): 자신이 자식 요소들을 포함할 수 있는 객체로, 트리의 가지 역할을 한다. 

### 사례
파일 시스템의 디렉토리 구조를 생각해보자. 우리는 디렉토리와 파일을 가지고 있으면, 디렉토리에 파일을 포함시켜 상하 구조를 가지게 만들었다. 디렉토리에는 파일이 들어있고 디렉토리에 파일의 크기를 알아낼 수 있다.  

### 기존 코드
```swift
class File {
    var size: Int
    
    init(size: Int) {
        self.size = size
    }
}

class Directory {
    private var files = [File]()
    
    func getSize() -> Int {
        var size = 0
        
        for file in files {
            size += file.size
        }
        return size
    }
    
    func add(file: File) {
        files.append(file)
    }
}

let file1 = File(size: 2)
let file2 = File(size: 3)
let directory1 = Directory()
directory1.add(file: file1)
directory1.add(file: file2)
print(directory1.getSize())


```
  
하지만 여기서 디렉토리들을 모두 포함한 최상위 디렉토리의 파일 크기를 알아내려면 문제가 발생한다. 지금 구조가 파일과 디렉토리에서 디렉토리와 파일만의 구조를 갖게 된다. 하지만, 디렉토리는 그보다 더 상위 디렉토리가 존재할 수도있다. 따라서 디렉토리는 자식 요소들을 포함 수 있는 컴포짓이 될 수도 있지만 리프가 될 수도 있다는 의미이다. 이러한 상황에선 컴포짓 패턴을 적용하면 디렉토리는 하위 디렉토리가 있던 파일이 있던 동일하게 처리를 한다는 점이다.
  
### 컴포짓 패턴을 적용한 리팩토링 코드
```swift 
// Component
protocol FileSystemItem {
    var name: String { get }
    var size: Int { get }
    
    func getSize() -> Int
}

// Leaf
class File: FileSystemItem {
    var name: String
    var size: Int

    init(name: String, size: Int) {
        self.name = name
        self.size = size
    }
    
    func getSize() -> Int{
        return size
    }
    
}

// Composite 
class Directory: FileSystemItem {
    var name: String
    var size: Int
    private var items = [FileSystemItem]() // 자식 요소들을 관리하는 리스트

    init(name: String, size: Int = 0) {
        self.name = name
        self.size = size
    }

    func add(item: FileSystemItem) {
        items.append(item)
    }

    func remove(item: FileSystemItem) {
        items.removeAll { $0.name == item.name }
    }
    
    func getSize() -> Int {
        var totalSize = 0
        for item in items {
            totalSize += item.getSize()
        }
        return totalSize
    }
}

let rootDirectory = Directory(name: "root")
let file1 = File(name: "file1.txt", size: 1)
let file2 = File(name: "file2.txt", size: 2)

let subDirectory = Directory(name: "sub")
let file3 = File(name: "file3.txt", size: 3)

rootDirectory.add(item: file1)
rootDirectory.add(item: subDirectory)
subDirectory.add(item: file2)
subDirectory.add(item: file3)
print(rootDirectory.getSize())

```
  
디렉토리는 리프가 될 수도 있고, 컴포짓이 될 수도 있게 되었지만, 하위 아이템들은 디렉토리나 파일인지 신경쓰지 않고 동일 하게 처리하게 된다. 클라이언트 코드에서는 구조를 신경쓰지 않고 단지 컴포넌트를 사용해서 단일 객체와 복합 객체를 같은 방식으로 다룰 수 있게 된다. 

> 장점
{: .prompt-tip } 

- 컴포짓 패턴은 트리 구조를 단순화하고 직관적으로 표현할 수 있게 해준다. 파일 시스템, UI 구성요소나 조직 구조등 다양한 문제에 사용이 가능하다.
- 클라이언트는 Leaf냐 Composite이냐 객체를 구분할 필요없이 동일한 인터페이스로 모든 객체를 처리할 수 있기 때문에 코드 단순화가 이루어 진다.


> 단점
{: .prompt-warning}

- 간단한 구조에선 오히려 복잡한 구조가 될 수 있다. 특히 관리 해야할 객체가 많지 않을 경우 필요없이 복잡해지게 된다.
- 모든 객체가 동일한 인터페이스로 구현되기 때문에 때로는 타입 검사를 해야할 경우가 생긴다.
