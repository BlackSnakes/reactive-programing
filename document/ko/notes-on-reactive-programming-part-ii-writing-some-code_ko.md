# Notes on Reactive Programming Part II: Writing Some Code

이 기사에서는 Reactive Programming에 대한 시리즈를 계속 진행하며 실제 코드샘플을 통해 몇 가지 개념을 설명하는 데 집중합니다. 
최종적으로 Reactive가 무엇이 다른지와 Reactive를 Funtional하게 만드는법을 이해하는 것입니다. 
여기의 예제는 매우 추상적이지만 API와 프로그래밍 스타일에 대해 생각하고 다른 방식에 대한 느낌을 갖기 시작할 수있는 방법을 제공합니다. 
Reactive의 요소를보고 데이터 흐름을 제어하고 필요한 경우 백그라운드 스레드에서 처리하는 방법을 배웁니다.

## 프로젝트 설정

Reactor 라이브러리를 사용하여 필요한 사항을 설명합니다. 
코드는 다른 도구로 쉽게 작성할 수 있습니다. 
코드를 가지고 놀고 아무 것도 복사하지 않고 코드를보고 싶다면 Github에서 테스트 한 샘플이 있습니다.

시작하려면 https://start.spring.io에서 빈 프로젝트를 잡고 Reactor Core 종속성을 추가하십시오.  
Maven을 이용한 설정: 
```
<dependency>
	<groupId>io.projectreactor</groupId>
	<artifactId>reactor-core</artifactId>
	<version>3.0.0.RC2</version>
</dependency>
```
유사하게 Gradle을 사용하여 설정할 수 있습니다.:
```
compile 'io.projectreactor:reactor-core:3.0.0.RC2'
```    
이제 코드를 작성해 보겠습니다.

## 무엇이 Functional하게 만드는가?

Reactive의 기본 구성 요소는 일련의 이벤트(Event)와 게시자(Publisher) 및 해당 이벤트 구독자(Subscriber)라는 두 가지 주체입니다. 
그것이 시퀀스이기 때문에 시퀀스를 "스트림"이라고 부를 수도 있습니다. 
필요하다면 "stream"이라는 단어를 작은 "s"와 함께 사용 하겠지만 Java 8에는 다른 java.util.Stream이 있으므로 혼동하지 않도록하십시오. 
어쨌든 게시자와 구독자에 대한 설명에 집중하려고합니다. (그게 리액티브 스트림이하는 것입니다).

Reactor는 샘플에서 사용할 라이브러리이므로 여기서는 Reactor표기법에따라 게시자를 Flux(Reactive Streams의 인터페이스 Publisher를 구현 함)라고 부릅니다. 
RxJava 라이브러리는 매우 유사하고 많은 병렬 기능을 가지고 있으므로 이 경우 Observable에 대해 대신 설명 하겠지만 코드는 매우 유사합니다. 
(Reactor 2.0은 Stream이라고 불렀고, Java 8 Streams에 관해서도 혼란스러워하기 때문에 Reactor 3.0에서만 새로운 코드를 사용합니다.)

## Generators

Flux는 특정 POJO 유형의 이벤트 시퀀스를 발행하는 게시자이므로 일반적인 것입니다. 
즉, Flux<T>는 T의 게시자입니다. Flux는 다양한 소스에서 자체의 인스턴스를 만들 수 있는 정적인 편리한 메소드를 가지고 있습니다. 
예를 들어, 배열로 Flux를 만들려면:
```java
Flux<String> flux = Flux.just("red", "white", "blue");
```
우리는 방금 Flux를 만들었고, 이제 우리는 그것을 가지고 뭔가를 할 수 있습니다. 
실제로는 두 가지 작업 만 수행 할 수 있습니다 : 변환 (변환 또는 다른 시퀀스와 결합), 구독 (게시자).

## Single Valued Sequences

종종 하나 또는 0 개의 요소 만있는 시퀀스를 발견하게됩니다. 
예를 들어 ID로 엔티티를 찾는 repository 메소드가 있습니다. 
Reactor에는 단일 값 또는 빈 Flux를 나타내는 Mono 유형이 있습니다. 
Mono는 Flux와 매우 유사한 API를 가지고 있지만 모든 연산자가 단일 값 시퀀스에 대해 이해할 수있는 것은 아니기 때문에 더욱 집중적입니다. 
또한 RxJava에는 Single이라고하는 볼트 (버전 1.x)가 있고 빈 시퀀스에는 Completable도 있습니다. 
Reactor의 빈 시퀀스는 Mono <Void>입니다.

## Operators

Flux에는 많은 메소드가 있으며 거의 모든 메소드가 연산자입니다. 
Javadocs에서 더 좋은 정보를 찾을 수 있기 때문에 여기서 모든 것을 보지는 않을 것입니다. 
우리는 연산자(Operator)가 무엇인지, 그리고 그것으로 무엇을 할 수 있는지에 대한 영감을 얻을 필요가 있습니다.

예를 들어 Flux 내부의 내부 이벤트를 표준 출력에 기록하려면 .log () 메소드를 호출 할 수 있습니다. 또한 map()을 사용하여 변형 할 수 있습니다.:
```java
Flux<String> flux = Flux.just("red", "white", "blue");

Flux<String> upper = flux
  .log()
  .map(String::toUpperCase);
```  
이 코드에서는 입력에서 문자열을 대문자로 변환하여 문자열을 변환했습니다. 사소하지만.

이 작은 샘플에 대해 흥미로운 점은 다시보면, 만약 사용되지 않았다면 데이터가 아직 처리되지 않았다는 것입니다. 
말 그대로 아무것도 기록되지 않았기 때문에 아무 것도 기록되지 않았습니다 (한번 해보면 압니다). 
Flux에서 호출하는 연산자는 나중에 실행할 계획을 세우는 데 그 몫을합니다. 
그것은 완전히 선언적이며 사람들이 "기능적"이라고 부르는 이유입니다. 
연산자에 구현 된 로직은 데이터가 흐르기 시작할 때만 실행되며 누군가가 Flux (또는 Publisher와 동등한)에 가입 할 때까지는 발생하지 않습니다.

일련의 데이터를 처리하는 것과 동일한 선언적, 기능적 접근 방식이 모든 Reactive 라이브러리와 Java 8 Streams에 있습니다. 
Flux와 동일한 내용의 Stream을 사용하여 이와 유사한 코드를 고려하십시오.:
```java
Stream<String> stream = Streams.of("red", "white", "blue");
Stream<String> upper = stream.map(value -> {
    System.out.println(value);
    return value.toUpperCase();
});
```
Flux에 대한 관찰(observation)은 여기에 적용됩니다. 
데이터가 처리되지 않으면 실행 계획에 불과합니다. 
그러나 Flux와 Stream 간에는 몇 가지 중요한 차이점이 있습니다. 
이로 인해 Stream이 Reactive 사용 사례의 부적절한 API가됩니다. 
Flux에는 훨씬 더 많은 연산자가 있으며 그 중 대다수는 편의성을 제공하지만 실제 차이점은 데이터를 소비하려는 경우에 발생하므로 다음에 살펴볼 필요가 있습니다.

>Tip
>[Reactive Types](https://spring.io/blog/2016/04/19/understanding-reactive-types)에는 Sebastien Deleuze가 작성한 유용한 블로그가 있습니다. 
>여기에서 다양한 스트리밍 API와 반응 API의 차이점은 정의하는 유형과 사용 방법을 살펴 보는 것입니다. 
>Flux와 Stream의 차이점이 더 자세히 설명되어 있습니다.

## Subscribers

데이터 흐름을 만들려면 subscribe () 메소드 중 하나를 사용하여 Flux에 가입해야합니다. 이러한 메서드 만 데이터 흐름을 만듭니다. 시퀀스에 선언 된 연산자 체인을 통해 다시 도달하고 게시자에게 데이터 작성을 요청합니다. 우리가 함께 작업 한 샘플 샘플에서는 기본 문자열 컬렉션이 반복된다는 것을 의미합니다. 좀 더 복잡한 경우에는 파일 시스템에서 파일을 읽거나 데이터베이스 또는 HTTP 서비스 호출에서 가져 오기를 트리거 할 수 있습니다.

실행중인 subscribe () 호출이 있습니다.:
```java
Flux.just("red", "white", "blue")
  .log()
  .map(String::toUpperCase)
.subscribe();
```
출력은 다음과 같습니다.:
```
09:17:59.665 [main] INFO reactor.core.publisher.FluxLog -  onSubscribe(reactor.core.publisher.FluxIterable$IterableSubscription@3ffc5af1)
09:17:59.666 [main] INFO reactor.core.publisher.FluxLog -  request(unbounded)
09:17:59.666 [main] INFO reactor.core.publisher.FluxLog -  onNext(red)
09:17:59.667 [main] INFO reactor.core.publisher.FluxLog -  onNext(white)
09:17:59.667 [main] INFO reactor.core.publisher.FluxLog -  onNext(blue)
09:17:59.667 [main] INFO reactor.core.publisher.FluxLog -  onComplete()
```

따라서 인수없이 subscribe ()의 효과가 게시자에게 모든 데이터를 보내도록 요청하는 것입니다. 하나의 요청 () 만 기록되고 "제한되지 않음"입니다. 또한 게시 된 항목 (onNext ()), 시퀀스 종료 (onComplete ()) 및 원래 구독 (onSubscribe ())에 대한 콜백을 볼 수 있습니다. 필요하다면 Flux에서 doOn * () 메소드를 사용하여 직접 이벤트를 수신 할 수 있습니다.이 메소드는 가입자가 아닌 운영자이기 때문에 자체적으로 데이터가 흐르지 않습니다.

subscribe () 메서드가 오버로드되고 다른 변형을 사용하면 발생하는 상황을 제어하는 다양한 옵션이 제공됩니다. 하나의 중요하고 편리한 형식은 콜백을 인수로 갖는 subscribe ()입니다. 첫 번째 인수는 각 항목에 대해 콜백을 제공하는 Consumer이며, Consumer에 오류가 있으면 추가하고 시퀀스 완료시 실행할 바닐라 Runnable을 선택적으로 추가 할 수도 있습니다. 예를 들어 항목 당 콜백만으로:
```java
Flux.just("red", "white", "blue")
    .log()
    .map(String::toUpperCase)
.subscribe(System.out::println);
```
출력은 다음과 같습니다.:
```
09:56:12.680 [main] INFO reactor.core.publisher.FluxLog -  onSubscribe(reactor.core.publisher.FluxArray$ArraySubscription@59f99ea)
09:56:12.682 [main] INFO reactor.core.publisher.FluxLog -  request(unbounded)
09:56:12.682 [main] INFO reactor.core.publisher.FluxLog -  onNext(red)
RED
09:56:12.682 [main] INFO reactor.core.publisher.FluxLog -  onNext(white)
WHITE
09:56:12.682 [main] INFO reactor.core.publisher.FluxLog -  onNext(blue)
BLUE
09:56:12.682 [main] INFO reactor.core.publisher.FluxLog -  onComplete()
```
우리는 데이터의 흐름을 제어하고 다양한 방식으로 "제한적"으로 만들 수 있습니다. 제어를위한 원시 API는 구독자로부터받는 구독입니다. 위의 subscribe ()에 대한 짧은 호출의 동일한 긴 형식은 다음과 같습니다.:
```java
.subscribe(new Subscriber<String>() {

    @Override
    public void onSubscribe(Subscription s) {
        s.request(Long.MAX_VALUE);
    }
    @Override
    public void onNext(String t) {
        System.out.println(t);
    }
    @Override
    public void onError(Throwable t) {
    }
    @Override
    public void onComplete() {
    }

});
```
흐름을 제어하기 위해. 한 번에 최대 2 개의 항목을 소비하려면 Subscription을보다 지능적으로 사용할 수 있습니다:
```java
.subscribe(new Subscriber<String>() {

    private long count = 0;
    private Subscription subscription;

    @Override
    public void onSubscribe(Subscription subscription) {
        this.subscription = subscription;
        subscription.request(2);
    }

    @Override
    public void onNext(String t) {
        count++;
        if (count>=2) {
            count = 0;
            subscription.request(2);
        }
     }
...
```
이 구독자는 한 번에 2 번 항목을 일괄 처리합니다. 일반적인 사용 사례이므로 구현을 편의 클래스로 추출하면 코드를보다 쉽게 ​​읽을 수 있습니다. 결과는 다음과 같습니다:
```
09:47:13.562 [main] INFO reactor.core.publisher.FluxLog -  onSubscribe(reactor.core.publisher.FluxArray$ArraySubscription@61832929)
09:47:13.564 [main] INFO reactor.core.publisher.FluxLog -  request(2)
09:47:13.564 [main] INFO reactor.core.publisher.FluxLog -  onNext(red)
09:47:13.565 [main] INFO reactor.core.publisher.FluxLog -  onNext(white)
09:47:13.565 [main] INFO reactor.core.publisher.FluxLog -  request(2)
09:47:13.565 [main] INFO reactor.core.publisher.FluxLog -  onNext(blue)
09:47:13.565 [main] INFO reactor.core.publisher.FluxLog -  onComplete()
```
실제로 일괄 처리 구독자는 Flux에서 이미 사용할 수있는 편리한 방법이있는 일반적인 사용 사례입니다. 위의 일괄 처리 예제는 다음과 같이 구현할 수 있습니다.:
```java
Flux.just("red", "white", "blue")
  .log()
  .map(String::toUpperCase)
.subscribe(null, 2);
```
(요청 제한이있는 subscribe () 호출 참고). 여기 출력:
```
10:25:43.739 [main] INFO reactor.core.publisher.FluxLog -  onSubscribe(reactor.core.publisher.FluxArray$ArraySubscription@4667ae56)
10:25:43.740 [main] INFO reactor.core.publisher.FluxLog -  request(2)
10:25:43.740 [main] INFO reactor.core.publisher.FluxLog -  onNext(red)
10:25:43.741 [main] INFO reactor.core.publisher.FluxLog -  onNext(white)
10:25:43.741 [main] INFO reactor.core.publisher.FluxLog -  request(2)
10:25:43.741 [main] INFO reactor.core.publisher.FluxLog -  onNext(blue)
10:25:43.741 [main] INFO reactor.core.publisher.FluxLog -  onComplete()
```
>Tip
>Spring Reactive Web과 같은 시퀀스를 처리 할 라이브러리가 구독을 처리 할 수 ​​있습니다. 이러한 문제는 스택을 사용하지 않아도되므로 비즈니스 로직이 아닌 코드를 복잡하게 만들지 않아 읽기 쉽고 테스트 및 유지 관리가 쉬워 지므로 이점을 누릴 수 있습니다. 원칙적으로 시퀀스 구독을 피하거나 처리 계층에 코드를 푸시하고 비즈니스 로직 외부로 푸시하는 것은 좋은 방법입니다.


## Threads, Schedulers and Background Processing

위의 모든 로그의 흥미로운 특징은 구독자 ()의 호출자 인 스레드가 "주"스레드에 있다는 것입니다. 이것은 중요한 점을 강조합니다. Reactor는 스레드에 대해 매우 검소합니다. 이는 최상의 성능을 발휘할 수있는 가장 큰 기회를 제공하기 때문입니다. 지난 5 년 동안 스레드와 스레드 풀 및 비동기 실행 문제를 해결하고 서비스에서 더 많은 주스를 뽑으려고하면 놀라운 결과 일 수 있습니다. 그러나 사실입니다. 스레드를 전환하는 것이 필수적이지 않으면 JVM이 스레드를 매우 효율적으로 처리하도록 최적화되어 있어도 단일 스레드에서 계산하는 것이 항상 더 빠릅니다. Reactor는 모든 비동기 처리를 제어 할 수있는 키를 사용자에게 제공했으며 사용자가 수행중인 작업을 알고 있다고 가정합니다.

Flux는 스레드 경계를 제어하는 몇 가지 구성 메소드를 제공합니다. 예를 들어, Flux.subscribeOn ()을 사용하여 배경 스레드에서 처리 할 구독을 구성 할 수 있습니다.:
```java
Flux.just("red", "white", "blue")
  .log()
  .map(String::toUpperCase)
  .subscribeOn(Schedulers.parallel())
.subscribe(null, 2);
```
결과는 출력에서 볼 수있다.:
```
13:43:41.279 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  onSubscribe(reactor.core.publisher.FluxArray$ArraySubscription@58663fc3)
13:43:41.280 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  request(2)
13:43:41.281 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  onNext(red)
13:43:41.281 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  onNext(white)
13:43:41.281 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  request(2)
13:43:41.281 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  onNext(blue)
13:43:41.281 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  onComplete()
```
>Tip
>이 코드를 직접 작성하거나 복사하여 붙여 넣으면 JVM이 종료되기 전에 처리가 중지 될 때까지 기다려야합니다.
구독 및 모든 처리는 단일 배경 스레드 "parallel-1-1"에서 발생합니다. 이는 우리가 주요 Flux의 구독자를 배경으로 요청했기 때문입니다. 항목 처리가 CPU 집중적 인 경우 (사실상 백그라운드 스레드에 있다는 것은 의미가 없지만 컨텍스트 스위치에 대해 비용을 지불하지만 결과가 더 빠르기 때문에) 괜찮습니다. I / O를 집중적으로 사용하고 차단하는 항목 처리를 수행 할 수도 있습니다. 이 경우 호출자를 차단하지 않고 최대한 빨리 처리하려고 할 수 있습니다. 스레드 풀은 여전히 ​​당신의 친구이며, Schedulers.parallel ()에서 얻을 수 있습니다. 개별 항목의 처리를 개별 스레드로 전환하려면 (풀의 한도까지) 개별 게시자로 분리해야하며 각 게시자는 결과 스레드를 백그라운드 스레드에서 요청해야합니다. 이 작업을 수행하는 한 가지 방법은 flatMap ()이라는 연산자로 항목을 잠재적으로 다른 유형의 게시자에 매핑 한 다음 새로운 유형의 시퀀스로 다시 매핑하는 것입니다:
```java
Flux.just("red", "white", "blue")
  .log()
  .flatMap(value ->
     Mono.just(value.toUpperCase())
       .subscribeOn(Schedulers.parallel()),
     2)
.subscribe(value -> {
  log.info("Consumed: " + value);
})
```
여기서 flatMap ()을 사용하여 항목을 "하위"게시자로 푸시 다운합니다. 여기에서 전체 시퀀스 대신 항목 당 구독을 제어 할 수 있습니다. Reactor는 가능한 한 오랫동안 단일 스레드에 매달릴 기본 동작을 내장하고 있으므로 특정 스레드 나 항목 그룹을 백그라운드 스레드에서 처리하도록하려면 Reactor가 명시 적이어야합니다. 사실 이것은 병렬 처리를 강제하는 몇 가지 알려진 트릭 중 하나입니다 (자세한 내용은 Reactive Gems 문제 참조).

결과는 다음과 같습니다.:
```
15:24:36.596 [main] INFO reactor.core.publisher.FluxLog -  onSubscribe(reactor.core.publisher.FluxIterable$IterableSubscription@6f1fba17)
15:24:36.610 [main] INFO reactor.core.publisher.FluxLog -  request(2)
15:24:36.610 [main] INFO reactor.core.publisher.FluxLog -  onNext(red)
15:24:36.613 [main] INFO reactor.core.publisher.FluxLog -  onNext(white)
15:24:36.613 [parallel-1-1] INFO com.example.FluxFeaturesTests - Consumed: RED
15:24:36.613 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  request(1)
15:24:36.613 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  onNext(blue)
15:24:36.613 [parallel-1-1] INFO reactor.core.publisher.FluxLog -  onComplete()
15:24:36.614 [parallel-3-1] INFO com.example.FluxFeaturesTests - Consumed: BLUE
15:24:36.617 [parallel-2-1] INFO com.example.FluxFeaturesTests - Consumed: WHITE
```
이제 항목을 소비하는 스레드가 여러 개 있고 flatMap ()의 동시성 힌트는 사용 가능한 한 2 개의 항목이 주어진 시간에 처리되도록합니다. 시스템은 파이프 라인에 2 개의 항목을 유지하려고 시도하고 있으며 일반적으로 동시에 처리를 완료하지 않기 때문에 요청 (1)이 많이 보입니다. Reactor는 사실 매우 똑똑하기 때문에 구독자 대기 시간을 없애기 위해 업스트림 게시자의 항목을 미리 가져옵니다 (숫자가 낮기 때문에 여기서는 표시되지 않습니다 - 우리는 3 개의 항목 만 처리 중입니다) .

>Tip
>세 가지 항목 ( "빨간색", "흰색", "파란색")이 너무 적어서 하나 이상의 배경 스레드를 확실하게 볼 수 없으므로 더 많은 데이터를 생성하는 것이 좋습니다. 예를 들어 난수 생성기를 사용하면됩니다.
Flux에는 publishOn () 메소드도 있지만 구독자 자체 대신에 리스너 (onNext () 또는 소비자 콜백)에 대해 동일합니다.:
```java
Flux.just("red", "white", "blue")
  .log()
  .map(String::toUpperCase)
  .subscribeOn(Schedulers.newParallel("sub"))
  .publishOn(Schedulers.newParallel("pub"), 2)
.subscribe(value -> {
    log.info("Consumed: " + value);
});
```
결과는 다음과 같습니다.:
```
15:12:09.750 [sub-1-1] INFO reactor.core.publisher.FluxLog -  onSubscribe(reactor.core.publisher.FluxIterable$IterableSubscription@172ed57)
15:12:09.758 [sub-1-1] INFO reactor.core.publisher.FluxLog -  request(2)
15:12:09.759 [sub-1-1] INFO reactor.core.publisher.FluxLog -  onNext(red)
15:12:09.759 [sub-1-1] INFO reactor.core.publisher.FluxLog -  onNext(white)
15:12:09.770 [pub-1-1] INFO com.example.FluxFeaturesTests - Consumed: RED
15:12:09.771 [pub-1-1] INFO com.example.FluxFeaturesTests - Consumed: WHITE
15:12:09.777 [sub-1-1] INFO reactor.core.publisher.FluxLog -  request(2)
15:12:09.777 [sub-1-1] INFO reactor.core.publisher.FluxLog -  onNext(blue)
15:12:09.777 [sub-1-1] INFO reactor.core.publisher.FluxLog -  onComplete()
15:12:09.783 [pub-1-1] INFO com.example.FluxFeaturesTests - Consumed: BLUE
```

소비자 콜백 ( "Consumed : ..."로깅)은 게시자 스레드 pub-1-1에 있습니다. subscribeOn () 호출을 꺼내면 pub-1-1 스레드에서 처리 된 두 번째 데이터 청크도 모두 표시됩니다. 다시 말해, Reactor가 쓰레드에 대해 검소한 것입니다. 쓰레드를 전환하라는 명시적인 요청이 없으면 다음 호출을 위해 동일한 스레드에 머무르게됩니다.

>Note
>이 샘플의 코드를 subscribe (null, 2)에서 publishOn ()에 프리 페치 = 2를 추가하는 것으로 변경했습니다. 이 경우 subscribe ()의 가져 오기 크기 힌트가 무시되었습니다.

## Extractors: The Subscribers from the Dark Side

Mono.block () 또는 Mono.toFuture () 또는 Flux.toStream ()을 호출하는 시퀀스를 구독하는 또 다른 방법이 있습니다 (이들은 "추출기"메서드입니다 - 그들은 당신을 Reactive 타입에서 덜 유연한, 추상화 차단). Flux에는 Flux에서 Mono로 변환하는 변환기 collectList () 및 collectMap ()도 있습니다. 그들은 실제로 시퀀스에 가입하지 않지만 개별 항목 수준에서 종결보다 더 많은 컨트롤을 버립니다.

>Warning
>엄지 손가락의 좋은 규칙은 "절대로 추출기를 부르지 말라"입니다. 몇 가지 예외가 있습니다 (그렇지 않으면 메소드가 존재하지 않을 것입니다). 주목할만한 >예외 중 하나는 테스트 결과입니다. 결과를 축적 할 수 있도록 차단하는 것이 유용하기 때문입니다.

이 메소드들은 Reactive에서 Blocking으로 이어지는 탈출 해치 (escape hatch)로 존재합니다. Spring MVC와 같은 레거시 API에 적응해야하는 경우. Mono.block ()을 호출하면 리 액티브 스트림의 모든 이점을 버리게됩니다. 이것은 리 액티브 스트림과 Java 8 스트림의 주요 차이점입니다. 원시 Java 스트림은 Mono.block ()에 해당하는 "모두 또는 아무것도 없음"구독 모델 만 갖습니다. 물론 subscribe ()도 호출 스레드를 차단할 수 있으므로 변환기 메서드와 마찬가지로 위험하지만 더 많은 제어 권한이 있습니다. subscribeOn ()을 사용하여 차단하지 못하도록하고 다시 적용하여 항목을 똑똑 떨어 뜨릴 수 있습니다 계속할지 여부를 주기적으로 결정해야합니다.

## Conclusion

이 기사에서는 Reactive Streams 및 Reactor API의 기본 사항을 다루었습니다. 더 많이 알아야 할 곳이 많이 있지만 코딩에 대한 대안이 없으므로 GitHub에서 코드를 사용하십시오 (이 프로젝트의 "flux"테스트에서이 기사에 대해). 또는 Lite로 넘어가십시오. RX 손에 워크샵. 지금까지는 실제로 이것은 오버 헤드였습니다. 우리는 비 반응 도구를 사용하여 더 명백한 방법으로 할 수 없었던 많은 것을 배웠습니다. 이 시리즈의 다음 기사에서는 Reactive 모델의 차단, 디스패치 및 비동기 측면에 대해 좀 더 자세히 살펴보고 실제 이점을 얻을 수있는 기회를 보여줍니다.
