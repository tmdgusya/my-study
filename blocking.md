# 개요

이 문서에서는 Blocking Structure / NonBlocking Structure 에 대해 내가 알고 있는 지식들을 연결하여 설명할 예정이다. 따라서 내용이 좀 길어질 수 있으며, 누군가에겐 루즈한 글일 수도 있다. 

## Code Flow

현재 우리가 대중적으로 사용하는 컴퓨터에서 우리가 작성하는 **Code 를 실행시키는 과정은 기본적으로 위에서 아래로 흐르려고 하는 성질**이 있다. 나는 이를 "Top-Down" 이라고 표현한다. 
가끔 condition 이나 go, switch 같은 회로를 만나면 위로 역전되는 현상이 있기도 하지만, 기본적으로는 위에서 아래로 흐르려는 방향이 있다. 즉, 그래서 대부분 우리는 코드를 동기적으로 짜는데 더 익숙할 수 밖에 없다. 
기본적으로 이 문장 실행후 아래 문장이 실행될꺼야. 라고 예상하기 때문이다. 더 자세하게 공부하고 싶다면 Interpreter 를 공부해보면 좋다.

## Sync?

**동기**란 무엇일까? 예전에 군대에서 각 인원들의 시간을 동일한 시간으로 맞추는 작업을 **시간 동기화** 라고 했다. 우리는 일상생활에서 어떠한 흐름속에서 Context 를 맞추는 일을 보통 동기라고 표현하는 것 같다. 
그렇다면 Function 에서 흐름이란 무엇일까? **Function 에서 흐름은 언어마다 다를 수 있지만 대부분 위에서 설명한 것처럼 Top-Down** 으로 흐른다. 따라서 우리가 Top-Down 의 흐름속에서 동기화를 한다는 건 어떤 뜻일까?
**위에서 실행된 어떠한 Context 에 대한 내용을 아래 코드에서 인지하고 있거나 처리할 수 있음**을 난 Function 에서 **Sync** 라고 생각한다. Multi-User OS 관점에서의 Sync 란 Multi-User 가 사용하는 흐름속에서 서로 공유하는 Data 를 어떻게 동기화 할까? 라고 생각할 수도 있다. 
즉, 어떻게 Scope 를 잡느냐에 따라 Structured Synchronization 를 구현해내는 방법이 다를 수 있다는 것 이다. 여하튼, 이 글을 클릭하는 대부분은 **Function Scope 관점**이기 때문에, 우리의 Sync 는 하나의 Function 에서 이어지는 Context 를 맞추는 일이 Sync 일 것이다. 
그렇다면 Blocking 은 무엇일까?

## Blocking?

이 개념을 이해하기 위해서는 Thread 에 대한 기본 배경이 어느정도는 필요하기에 간단히 설명하려고 한다. Thread 란 쉽게 말해 운영체제가 사용하는 작업자이다. 
우리가 `A.js` 라는 파일을 실행시킨다면, `A.js` 를 스레드가 컴파일 해주고, `A.js` 의 프로그램 진입점을 찾아서 Thread 는 `A.js` 를 위에서 아래로 읽어 내려가면서 실행시킬 것이다. 따라서 우리가 작성한 코드를 읽고, 실행시키는 녀석은 결국 Thread 
라는 것을 알게 될 것이다. 그래서 Thread 가 Blocking 된다는 의미는 일하고 있던 Thread 가 잠시 일을 멈춘 상태를 뜻한다. 우리는 컴퓨터 자원을 효율적으로 사용해야 하는데, Thread 가 Blocking 되는 것을 바라지 않을 것이다. 몇몇의 Thread 가 놀기 시작하면 CPU 를 효율적으로 사용하는 것은 
거의 불가능 할 것 이기 때문이다. 하지만 그럼에도 우리의 코드는 **Sync-Blocking Structure** 가 많다. 그 이유는 무엇일까? 위에서 설명했듯, 우리에게 익숙하고 훨씬 더 읽기 쉬운 방법이기 때문이다. 예를 들어 아래의 코드를 보자. **(대부분의 code는 특정 언어를 나타내기 보다는 sudo code 이다.)**

```js
function A() {
    const result = block B() // FuncB 를 실행시키는 건 다른 스레드에게 위임해야 함.

    console.log(result)
}
```

우리는 Thread 를 이해해 가며 위의 코드를 분석해 볼 것이다. 하나의 Main Thread 가 A 로 진입한다. 그래서 첫번째 줄을 읽는데, `B()` 를 다른 스레드에게 위임하여 실행시키고 그 결과가 나올때까지 blocking 하라는 명령을 발견한다. 
그래서 **Main Thread 는 Function Thread 에게 B() 를 실행시켜줘 라고 위임하고, Function Thread 가 B() 를 끝낼때 까지 대기**한다. 즉, 여기서 **Main Thread 는 Blocking 상태**이다. 즉, 어떻게 보면 Function Thread 가 `B()` 를 해석하고, 실행시키기 위해 코드 해석의 주도권을 가져갔다고 해석할 수도 있다. 
그래서 대부분의 설명에서 "A 함수가 B 함수를 호출 할 때, B 함수가 자신의 작업이 종료되기 전까지 A 함수에게 제어권을 돌려주지 않는 것" 라고 설명하는 것이라고 이해하면 편할 것이다. 일단 위의 설명을 읽었다면 어느정도 Blocking Model 에 관한 이해나 자신만의 판단이 자리잡았을 것이라고 생각한다.

## NonBlocking?

그렇다면 NonBlocking 은 무엇일까? 쉽게 말해 Thread 를 Blocking 하지 않는다. 라는 뜻이다. 설명을 위해 아까 코드를 다시 가져오겠다.

```js
function A() {
    const result = block B() // FuncB 를 실행시키는 건 다른 스레드에게 위임해야 함.

    console.log(result)
}
```

이 코드를 nonBlock 으로 실행시키기 위해서는 어떻게 해야할까? 방법은 간단하다 하나의 Rule 을 정하며 된다.

> Async Function 은 자신이 호출한 곳에 함수가 끝나지 않아도 특정 값을 Return 하여 호출한 쪽의 Thread 가 Blocking 되지 않도록 한다.

그래서 우리는 B Function 을 아래와 같이 구상해 볼 수 있을 것이다.

```js
nonBlock function B() {
    fake return "Async Execute"

    // do something

    return Proxy("realValue")
}
```

즉, B() 함수는 끝나지 않았지만 호출되는 동시에 A 에게 **"Async Execute"** 를 넘겨 줌으로써 마치 함수가 끝난 것 처럼 보이게 한다. 따라서 Main Thread 는 B 함수가 끝났구나, 
라고 생각하며 다음 코드들을 실행하게 만드는 것이다.

```js
function A() {
    const result = nonBlock B() // FuncB 를 실행시키는 건 다른 스레드에게 위임해야 함.

    console.log(result.realize()) // 실체화 하는 과정에서 B() 함수가 끝나지 않았다면 Blocking 이 발생할 수 있음.
}
```

그래서 우리가 만약 위의 함수처럼 `B()` 함수에서 Return 한 값을 써야만 한다면 Proxy 객체로 감싸서, 사용하는 시점에 실체화 되게 만들 수도 있다. 이 정도면 NonBlocking 모델에 대한 이해가 섰을 것이라고 생각한다.

## Async?

Async? 비동기란 무엇일까? **Aysnc 는 주로 Event Model 을 이용하여 많이 구현**된다. 아래의 코드를 한번 보자.

```js
/**
 *  EventBus 는 자동으로 Event 를 Publishing 하고 Consume 할 수 있도록 도와주는 프레임워크이다.
 */
const eventQueue : EventBus[] = new EventBus();

function A() {
    eventQueue.addEvent(BuyEvent())
    eventQueue.addEvent(CreateOrderEvent())
    eventQueue.addEvent(SendKakaoTalk())
}

consume function processBuyEvent() {
    if (eventQueue.isExist(BuyEvent::class)) {
        // do something
        return "consume Finish"
    }
}

consume function processCreateOrderEvent() {
    if (eventQueue.isExist(CreateOrderEvent::class)) {
        // do something
        return "consume Finish"
    }
}
```

위에서 보듯이, **A 는 저 두개의 이벤트가 순차적으로 실행되든 말든 신경쓰지 않는다.** 즉, 이때까지는 위에서 아래까지 순서대로 실행되기를 보장하기를 원했다면, 이제는 어떻게 순서대로 실행되든 말든, 일단 이 Event 들을 실행시켜줘 라고 하는 것이다. 즉, 동기 / 비동기의 차이는 순서대로 실행을 보장하는가? 에 의미가 크다. 즉, **동기 / 비동기의 개념은 "시간" 에 묶여있다면, Blocking / NonBlocking 의 개념은 "Thread" 에 밀접한 개념**이다. 

## Async-Await

이 글을 읽으면서 **Async-Await 모델**이 있는 언어를 사용하고 있는 사람이라면, 왜 **Event-Loop 가 존재하고 Callback 이 존재하는지 이해가 갈 것**이라고 생각한다. 다시 위의 코드를 들고 와서 설명을 하자면, 사실 아래 코드는 **Async-Await 모델** 기준에서는 아래와 같이 `callback function` 으로 해석될 것이다.

```js
/**
 *  EventBus 는 자동으로 Event 를 Publishing 하고 Consume 할 수 있도록 도와주는 프레임워크이다.
 */
const eventQueue : EventBus[] = new EventBus();

function A() {
    eventQueue.addEvent(BuyEvent()).callBack(processBuyEvent)
    eventQueue.addEvent(CreateOrderEvent()).callBack(processCreateOrderEvent)
    eventQueue.addEvent(SendKakaoTalk())
}

callback function processBuyEvent() {
    // do something
}

callback function processCreateOrderEvent() {
    // do something
}
```

## 끝마치며

다소 틀린내용이 포함되어 있을 수 있습니다. 틀린 내용을 발견한다면 댓글이나 메일로 알려주시면 감사합니다.