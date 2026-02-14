---
title:  "[Design Pattern] Singleton Pattern
# excerpt: "번들링이란"
toc: true
toc_label: "index"
toc_sticky: true
date: 2026-02-14
comments: true
classes: wide
header:
    teaser: ../../assets/images/cs/260214-thumbs.png
---


### 싱글톤 패턴

---

하나의 클래스로부터 단 한개의 인스턴스만 생기도록 보장하는 패턴이다. **인스턴스의 생성을 클래스 차원에서 제어**하여, 시스템 전체에서 **동일한 상태를 공유**하고 **불필요한 메모리 낭비를 방지**하기 위해 사용한다. 

```jsx
class User {
    static #instance = null; //# = 클래스 내부의 메서드들만 접근
    
    constructor(name,age) {
    
	    if(User.#instance) {
            console.log(`이미  ${User.#instance.name}인스턴스 있으므로 constructor 호출 불가`)
            return User.#instance
        }
        
        this.name = name
        this.age = age
        console.log(`${this.name}인스턴스 생성`)
    }
    
     static getInstance(name,age) {
        if(User.#instance) {
            console.log(`이미 ${this.name} 존재함`)
            return User.#instance
        }
        
        User.#instance = new User(name,age);
        return User.#instance;
    }
}
```

JS로 코드를 작성하면 위와 같다.

```jsx
const a = User.getInstance("a",26); //a인스턴스 생성
const b = User.getInstance(); //이미 User 존재함
console.log(a === b) //true

```

하지만 아래의 코드를 실행하면 2개의 인스턴스가 생성됨을 알 수 있다. 

```jsx
const a = User.getInstance("a",26); //a인스턴스 생성
const b = new User("b",25) //b인스턴스 생성
```

JS에는 `constructor`를 클래스 내부에서만 사용할 수 있도록 private같은 키워드가 없기 때문에 중복 생성 문제가 발생할 수 있다. new 키워드는 '객체 생성'을 강제로 실행하기 때문이다. 따라서 `constructor` 내부에 if문을 추가해 이미 인스턴스가 있는지 한 번 더 확인한다. 

```jsx
class User {
	...
	    if(User.#instance) {
            console.log(`이미  ${User.#instance.name}인스턴스 있으므로 constructor 호출 불가`)
            return User.#instance
        }
        ...
}

const a = User.getInstance("a",26); //a인스턴스 생성
const b = new User("b",25) //이미  a인스턴스 있으므로 constructor 호출 불가
console.log(a === b) //true

```

TS에서는 private기능을 제공해주기 때문에 키워드를 사용해서 접근 범위를 조절할 수 있다. 

```jsx
class User {
	
	private static instance : User   = null;
	
	private constructor () {}
	
	public static getInstance() : User {}
	
}
```

`private static`에서 `static`은 객체로 내리지 않고 클래스에서만 존재하는 이라는 뜻이고, `private`는 클래스 내부에서만 접근가능하다는 의미다. 즉 외부인이 직접 `instance`를 만질 수 없게 만들고, 단 한개의 인스턴스만 만들도록 설계된 `public static getInstance()` 라는 메소드를 통해서만 전달 받을 수 있다. 

그럼 `getInstance()`는 왜 static일까? `getInstance()`는 “내가 유일한가?"를 감시하는 역할을 하기 때문에, 각 객체가 본인이 유일한 지 감시하게 된다면 설계상 책임 분배를 잘못하게 된다. 

### 싱글톤 패턴의 단점

---

1. 모듈 간 강결합 되어 있기 TDD할 때 걸림돌이 된다. 
    
    TDD의 핵심 원칙 중 하나는 “각 테스트는 독립적이고, 실행 순서와 상관없이 실행 결과가 같아야 한다”는 것이다. 하지만 싱글톤 패턴에서는 각 모듈이 모두 1개의 인스턴스에 접근해 있기 때문에 각 테스트가 독립적일 수가 없다.  
    
    즉, 인스턴스들이 서로의 이름을 직접 부르며 강결합되어 있어서, 특정 로직만 분리해서 테스트하는 단위 테스트(Unit Test)가 불가능해진다.
    
    ```jsx
    class OrderServices {
        process() {
        const payment  = Payment.getInstance();
        payment.pay();
        }
    }
    
    class Payment {
        process () {
            const Network = Network.getInstance();
            Network.connect();
        }
    }
    
    class Network {
        process () {
            const DB = DB.getInstance();
            DB.execute()
        }
    }
    
    ```
    
    이러한 결제 서비스가 있다고 가정해보자. 결제 로직인 `OrderServices` 만 테스트를 하고 싶지만,  해당 코드에서 가짜 모듈을 주입해 테스트할 방법이 없게 된다. 가짜 모듈을 넘겨줄 수 있는 곳이 존재하지 않기 때문이다. 이를 의존성 주입으로 해결할 수 있다. 
    
    ```jsx
    class OrderServices {
        constructor(paymentMoudule) {
            thist.paymentMoudule = paymentMoudule
        }
        
        process() {
        this.paymentMoudule.pay()
        }
    }
    ...
    ```
    
    결제 모듈과의 의존성을 끊어서 가짜 모듈을 넘겨줄 수 있는 경로를 만들어준다.  이후 테스트 시에는 `new OrderService(new 가짜결제모듈())`로 테스트가 가능하다. 
    
    코드를 유연하게 만들 수 있다는 장점은 있지만, 의존성 주입이 복잡해질 수록 보일러플레이트가 증가한다. 이를 위해서 DI 컨테이너로 이 과정을 자동화 할 수 있다. 
    
    ```jsx
    @Injectable() class Logger {}
    @Injectable() class Network { constructor(logger: Logger) {} }
    @Injectable() class Payment { constructor(network: Network) {} }
    @Injectable() class OrderService { constructor(payment: Payment) {} }
    ```
    
    데코레이터(`@`)나 설정 파일을 사용해서 하나의 설계도을 만들고 실제 사용할 때는 :
    
    ```jsx
    const orderService = container.get(OrderService);
    ```
    
    코드 한 줄로 사용이 가능하다. 
    
    하지만 실제 인스턴스들이 연결되는 구체적인 흐름을 놓치기 쉽기 때문에, 디버깅이 어려울 수 있다는 점을 유의해야 한다.