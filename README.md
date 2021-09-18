# 코루틴에 대하여

### 코루틴이란

코틀린 의 코루틴은 비동기 프로그래밍을 처리할수 있는 좋은 방법이다.
코루틴을 사용하면 어떠한 작업을 foreground (전경) 에서 background (배경) 으로 또는 그 반대로 쉽게 전환하면서 처리할수 있다.

***
### 코루틴 핵심 키워드
- CoroutineScope (그리고 GlobalScope)
- CoroutineContext
- Dispatcher
- launch (그리고 async)

#### CoroutineScope
CoroutineScope 는 말 그대로 코루틴의 범위, 코루틴 블록을 묶음으로 제어할수 있는 단위이다.

GlobalScope 는 CoroutineScope 의 한 종류이다. 미리 정의된 방식으로 프로그램 전반에 걸쳐 백그라운드 에서 동작한다.
#### CoroutineContext
CoroutineContext 는 코루틴을 어떻게 처리 할것인지 에 대한 여러가지 정보의 집합이다.

CoroutineContext 의 주요 요소 로는 **Job** 과 **dispatcher** 가 있다.
#### Dispatcher
Dispatcher 는 CoroutineContext 의 주요 요소 이다.

CoroutineContext 을 상속받아 어떤 스레드를 이용해서 어떻게 동작할것인지를 미리 정의해 두었다.

다음은  io’19 세션 Understand *Kotlin Coroutines on Android* 에서 설명된 Dispatcher 의 용도 입니다.

- Dispatchers.Default : CPU 사용량이 많은 작업에 사용한다. 주 스레드에서 작업하기에는 너무 긴 작업 들에게 알맞다.
- Dispatchers.IO : 네트워크, 디스크 사용 할때 사용한다. 파일 읽고, 쓰고, 소켓을 읽고, 쓰고 작업을 멈추는것에 최적화되어 있다.
- Dispatchers.Main : 안드로이드의 경우 UI 스레드를 사용한다.
### 코루틴 사용방법
1. 사용할 Dispatcher 를 결정한다.
2. Dispatcher 를 이용해서 CoroutineScope 만든다.
3. CoroutineScope 의 launch 또는 async 에 수행할 코드 블록을 넘긴다.
launch 와 async 는 CoroutineScope 의 확장함수 이며, 넘겨 받은 코드 블록으로 코루틴을 만들고 실행해주는 코루틴 빌더 이다.
launch 는 Job 객체를, async 는 Deferred 객체를 반환 하며, 이 객체를 사용해서 수행 결과를 받거나, 작업이 끝나기를 대기하거나, 취소하는 등의 제어가 가능하다.

##### 기본적인 형태

```Kolin
   // 이 CoroutineScope 는 메인 스레드를 기본으로 동작합니다
    // Dispatchers.IO 나 Dispatchers.Default 등의 다른 Dispatcher 를 사용해도 됩니다
    val scope = CoroutineScope(Dispatchers.Main)

    scope.launch {
        // 포그라운드 작업
    }

    scope.launch(Dispatchers.Default) {
        // CoroutineContext 를 변경하여 백그라운드로 전환하여 작업을 처리합니다
    }

```
