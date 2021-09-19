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
### 코루틴 제어를 위한 주요 키워드
- **launch , async**
- **Job , Deferred**
- **runBlocking**
코루틴 블록 내에서 어떤 작업을 “어떻게 처리”하고 “어떠한 결과로 반환” 할것인가 하는 제어 에 관한 이야기를 다루려고 한다.
즉, 코루틴 블록을 조합하여 동기 그리고 비동기 로 사용하는 방법이다.

##### launch() — Job
launch() 함수로 시작된 코루틴 블록은 Job 객체를 반환한다.

```Kolin
val job : Job = launch {
    ...
}
```
반환받은 Job 객체로 코루틴 블록을 취소하거나, 다음 작업의 수행전 코루틴 블록이 완료 되기를 기다릴수 있다.

```Kolin
        val job = launch {
            var i = 0
            while (i < 10) {
                delay(500)
                i++
            }
        }

        job.join() // 완료 대기
        job.cancel() // 취소
```
여러개의 launch 코루틴 블록을 실행할 경우 각각의 Job 객체에 대해서 join() 함수로 코루틴 블록이 완료 될때까지 다음 코드 수행을 대기할수 있다.
```Kolin
       val job1 : Job = launch {
            var i = 0
            while (i < 10) {
                delay(500)
                i++
            }
        }

        val job2 = launch {
            var i = 0
            while (i < 10) {
                delay(1000)
                i++
            }
        }

        job1.join()
        job2.join()
```
모든 Job 객체에 대해서 일일히 join() 함수를 호출하지 않고 joinAll() 함수를 이용하여 모든 launch 코루틴 블록이 완료 되기를 기다릴수도 있다.
```Kolin
joinAll(job1, job2)
```
또는, 다음의 예시와 같이 첫번째 launch 코루틴 블록에서 반환받은 Job 객체를 두번째 launch() 함수의 인자로 사용하면, 동일한 Job 객체로 두개의 코루틴 블록을 모두 제어 할수 있다.
```Kolin
      val job1 = launch {
            var i = 0
            while (i < 10) {
                delay(500)
                i++
            }
        }

        // 위 블록 과 같은 job1 객체를 사용
        launch(job1) {
            var i = 0
            while (i < 10) {
                delay(1000)
                i++
            }
        }

        // 같은 job 객체를 사용하게 되면
        // joinAll(job1, job2) 와 같다
        job1.join()
```
launch() 함수로 정의된 코루틴 블록은 즉시 수행되며, 반환 받은 Job 객체는 해당 블록을 제어는 할수 있지만 코루틴 블록의 결과를 반환하지는 않는다.
코루틴 블록의 결과 값을 반환받고 싶다면 async() 코루틴 블록을 생성한다.
##### async() — Deferred
async() 함수로 시작된 코루틴 블록은 Deferred 객체를 반환한다.
```Kolin
val deferred : Deferred<T> = async {
    ...
    T // 결과값
}
```
이렇게 시작된 코루틴 블록은 Deferred 객체를 이용해 제어가 가능하며 동시에 코루틴 블록에서 계산된 결과값을 반환 받을수 있다.
```Kolin
        val deferred : Deferred<String> = async {
            var i = 0
            while (i < 10) {
                delay(500)
                i++
            }

            "result"
        }
        
        val msg = deferred.await()
        println(msg) // result 출력
```
여러개의 async 코루틴 블록을 실행할 경우 각각의 Deferred 객체에 대해서 await() 함수로 코루틴 블록이 완료 될때까지 다음 코드 수행을 대기할수 있다. await() 함수는 코루틴 블록이 완료되면 결과를 반환한다.
```Kolin
        val deferred1 = async {
            var i = 0
            while (i < 10) {
                delay(500)
                i++
            }

            "result1"
        }

        val deferred2 = async {
            var i = 0
            while (i < 10) {
                delay(1000)
                i++
            }

            "result2"
        }

        val result1 = deferred1.await()
        val result2 = deferred2.await()
        
        println("$result1 , $result2") // result1 , result 2 출력
```
각각의 Deferred 객체에 대해서 await() 함수를 호출하지 않고 awaitAll() 함수를 이용하여 모든 async 코루틴 블록이 완료 되기를 기다릴수도 있다.
```Kolin
awaitAll(deferred1, deferred2)
```
##### 지연실행
launch 코루틴 블록 과 async 코루틴 블록은 모두 처리 시점을 뒤로 미룰수 있다.
각 코루틴 블록 함수의 start 인자에 CoroutineStart.LAZY 를 사용하면 해당 코루틴 블록은 지연 되어 실행된다.
```Kolin
val job = launch (start = CoroutineStart.LAZY) {
    ...
}
또는
val deferred = async (start = CoroutineStart.LAZY) {
    ...
}
```
launch 코루틴 블록을 지연 실행 시킬 경우 Job 클래스 의 start() 함수 를 호출하거나 join() 함수를 호출하는 시점에 launch 코드 블록이 수행된다.
```Kolin
job.start()
또는
job.join()
```
##### runBlocking()
runBlocking() 함수는 코드 블록이 작업을 완료 하기를 기다린다.
```Kolin
runBlocking {
    ...
}
```
### 코루틴 에서의 작업 취소
코루틴의 완벽한 제어를 위해서는 작업을 기다리고, 완료된 작업의 결과를 반환 받아서 처리하는것 뿐만아니라 작업의 취소 까지도 처리할수 있어야 한다.
제어에 사용 되는 Job 클래스 와 Deferred 클래스 에는 코루틴 블록의 작업을 취소하기 위한 cancel() 함수가 존재한다.

다음은 500 밀리초 간격으로 특정 문자열을 1000회 출력하는 코루틴 블록이다. 이 블록을 시작한 이후 1300 밀리초가 지나면 코루틴을 취소한다.
```Kolin
fun cancellingCoroutineExecution() = runBlocking {
    val job = launch {
        repeat(1000) { i ->
            println("job: I'm sleeping $i ...")
            delay(500L)
        }
    }
    
    delay(1300L)
    println("main: I'm tired of waiting!")
    job.cancel()
    job.join()
    println("main: Now I can quit.")
}
```
```Kolin
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
main: Now I can quit.
```
