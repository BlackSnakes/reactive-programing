# Notes on Reactive Programming Part I: The Reactive Landscape

리액티브 프로그래밍은 흥미롭지만 필자와 같은 간단한 엔테프라이즈 자바 개발자나 다른 외부인이 이해하기 어려운만큼 매 순간 많은 노이즈가 있다.

이 기사 (시리즈의 첫번째 기사) 에서는 그 어지러운 노이즈들에 대한 이해를 밝히는 데 도움이 될 수 있다.
접근 방식은 가급적 구체적이고, 의미론적 Semantics 에 대해서는 언급이 없다.

당신이 만일 학문적 접근법이나, 많은 양의 하스켈 샘플 코드를 찾고 있다면, 인터넷에 더 많은 자료가 있고, 아마 여기에 있진 않을 것 같다.

리액티브 프로그래밍은 종종 비동기 프로그래밍, 고성능 프로그래밍과 결합되어 그 개념들과 구분하기 어렵지만 실제로 그것들과는 원칙적으로 완전히 다르다.

이건 필연적으로 혼란을 가져온다.

리액티브 프로그래밍은 또한 종종 함수형 리액티브 프로그래밍 혹은 FRP(서로 같은 의미) 와 결합되거나 그렇게 불려진다. 어떤 사람은 리액티브 프로그래밍이 전혀 새로운게 아니며 보통의 일상이라고 한다 (대부분 그들은 자바스크립트를 사용한다).

다른 사람들은 Microsoft 가 주는 선물이라고 생각한다 (얼마전 어떤 C# 의 확장이 릴리즈되었을때 그것에 대해 크게 환호한 사람들).

Enterprise Java 세계에서는 최근 화제가 되었는데, (에를 들어 Reactive Streams initiative 를 봐라) 새롭고 반짝이는 무언가와 그것들이 만들어내는 언제 어디서나 발생된 간단한 실수들이 많았다. (뭔소리여...)

## What Is It?

Reactive Programming 은 개선된 라우팅과 이벤트의 소비를 포함하는 마이크로 아키텍쳐 스타일이고, 전부 행동을 바꾸기 위해 결합된다.

이는 다소 추상적이며, 이것에 대한 다른 많은 정의도 온라인에 있다. 우리는 Reactive 가 의미하는 것이 무엇인지, 또는 다음에 설명할 것이 왜 중요한지에 대해 좀 더 구체적인 개념을 세우려고 한다.

Reactive Programming 의 기원은 아마 1970년 혹은 더 이르게 거슬러 올라갈 수 있기에 전혀 새로운 아이디어가 아니지만 그것들은 근대적 엔터프라이즈 안에 무언가 공감되는 것이 있다.

이 공명은 마이크로서비스의 발생과 같은 시기에 멀티코어 프로세서를 전제하여 발생했다. (우연이 아니다.) 그 이유 중 어떤건 아마 조금 더 확실해질 것이다.

여기 유용한 다른 출처에서 거져온 유용한 정의가 있다:

> Reactive Programming 뒤의 기본 아이디어는 시간이 지남에 따라 값을 표현하는 데이터 유형이라는 것이다. 이런 시간 경과 변화들을 포함한 Value 의 계산은 시간이 변하는 값을 각자 가진다. (???)

그리고...

>첫 이해를 하기에 쉬운 방법은 다음과 같이 너의 프로그램이 스프레드 시트이고 모든 변수들이 셀이라고 생각하는 것이다.
스프레드시트에서 어떤 셀이 변하면, 그 셀의 변화를 어떤 셀들은 참조한다. 이건 FRP 와 똑같다.<br/>
지금 일부의 셀들이 스스로 변한다고 생각해보라.(혹은 다른 세계에서 가져온 것들이다): GUI 에서 마우스의 포지션은 좋은 예가 될 수 있다.<br/>

(Stackoverflow 의 기술 질문에서 가져옴)

>참고자료 Mouse is Database.<br/>
http://huns.me/development/2051#attachment_2068


FRP 는 고성능, 동시성, 비동기 명령과 넌 블러킹 IO에 강한 친화력을 가지고 있다. 

하지만, FRP 가 그들과 어떠한 관련이 없다는 의심으로 시작하는 것이 도움이 될 것이다.

Reactive Model 을 사용할때, 각각의 관심사가 호출자에게 투명하게 처리하는건 당연한 것이다.

하지만 이러한 관심사를 효과적 혹은 효율적으로 처리하는 면에서는 전적으로 해당 구현에 달려 있다. (따라서 엄격한 판단이 필요하다).

또한 동기식으로 싱글 스레드를 사용한 올바르고 유용한 FRP 프레임워크를 구현하는게 가능하지만, 그것은 새로운 도구나 라이브리를 사용려고 하는 데에는 도움이 되지 않는다.

>참고 : 리액티브 개발 패러다임에 담긴 메시지<br/>
http://www.zdnet.co.kr/column/column_view.asp?artice_id=20161010104628

## Reactive Use Cases

The hardest question to get an answer to as a newbie seems to be "what is it good for?" Here are some examples from an enterprise setting that illustrate general patterns of use:

`External Service Calls` Many backend services these days are REST-ful (i.e. they operate over HTTP) so the underlying protocol is fundamentally blocking and synchronous. Not obvious territory for FRP maybe, but actually it’s quite fertile ground because the implementation of such services often involves calling other services, and then yet more services depending on the results from the first calls. With so much IO going on if you were to wait for one call to complete before sending the next request, your poor client would give up in frustration before you managed to assemble a reply. So external service calls, especially complex orchestrations of dependencies between calls, are a good thing to optimize. FRP offers the promise of "composability" of the logic driving those operations, so that it is easier to write for the developer of the calling service.

Highly Concurrent Message Consumers Message processing, in particular when it is highly concurrent, is a common enterprise use case. Reactive frameworks like to measure micro benchmarks, and brag about how many messages per second you can process in the JVM. The results are truly staggering (tens of millions of messages per second are easy to achieve), but possibly somewhat artificial - you wouldn’t be so impressed if they said they were benchmarking a simple "for" loop. However, we should not be too quick to write off such work, and it’s easy to see that when performance matters, all contributions should be gratefully accepted. Reactive patterns fit naturally with message processing (since an event translates nicely into a message), so if there is a way to process more messages faster we should pay attention.

Spreadsheets Perhaps not really an enterprise use case, but one that everyone in the enterprise can easily relate to, and it nicely captures the philosophy of, and difficulty of implementing FRP. If cell B depends on cell A, and cell C depends on both cells A and B, then how do you propagate changes in A, ensuring that C is updated before any change events are sent to B? If you have a truly active framework to build on, then the answer is "you don’t care, you just declare the dependencies," and that is really the power of a spreadsheet in a nutshell. It also highlights the difference between FRP and simple event-driven programming — it puts the "intelligent" in "intelligent routing".

Abstraction Over (A)synchronous Processing This is more of an abstract use case, so straying into the territory we should perhaps be avoiding. There is also some (a lot) of overlap between this and the more concrete use cases already mentioned, but hopefully it is still worth some discussion. The basic claim is a familiar (and justifiable) one, that as long as developers are willing to accept an extra layer of abstraction, they can forget about whether the code they are calling is synchronous or asynchronous. Since it costs precious brain cells to deal with asynchronous programming, there could be some useful ideas there. Reactive Programming is not the only approach to this issue, but some of the implementaters of FRP have thought hard enough about this problem that their tools are useful.

This Netflix blog has some really useful concrete examples of real-life use cases: Netflix Tech Blog: Functional Reactive in the Netflix API with RxJava

## Comparisons

If you haven’t been living in a cave since 1970 you will have come across some other concepts that are relevant to Reactive Programming and the kinds of problems people try and solve with it. Here are a few of them with my personal take on their relevance:

Ruby Event-Machine The Event Machine is an abstraction over concurrent programming (usually involving non-blocking IO). Rubyists struggled for a long time to turn a language that was designed for single-threaded scripting into something that you could use to write a server application that a) worked, b) performed well, and c) stayed alive under load. Ruby has had threads for quite some time, but they aren’t used much and have a bad reputation because they don’t always perform very well. The alternative, which is ubiquitous now that it has been promoted (in Ruby 1.9) to the core of the language, is Fibers(sic). The Fiber programming model is sort of a flavour of coroutines (see below), where a single native thread is used to process large numbers of concurrent requests (usually involving IO). The programming model itself is a bit abstract and hard to reason about, so most people use a wrapper, and the Event Machine is the most common. Event Machine doesn’t necessarily use Fibers (it abstracts those concerns), but it is easy to find examples of code using Event Machine with Fibers in Ruby web apps (e.g. see this article by Ilya Grigorik, or the fibered example from em-http-request). People do this a lot to get the benefit of scalability that comes from using Event Machine in an I/O intensive application, without the ugly programming model that you get with lots of nested callbacks.

Actor Model Similar to Object Oriented Programming, the Actor Model is a deep thread of Computer Science going back to the 1970s. Actors provide an abstraction over computation (as opposed to data and behaviour) that allows for concurrency as a natural consequence, so in practical terms they can form the basis of a concurrent system. Actors send each other messages, so they are reactive in some sense, and there is a lot of overlap between systems that style themselves as Actors or Reactive. Often the distinction is at the level of their implementation (e.g. Actors in Akka can be distributed across processes, and that is a distinguishing feature of that framework).

Deferred results (Futures) Java 1.5 introduced a rich new set of libraries including Doug Lea’s "java.util.concurrent", and part of that is the concept of a deferred result, encapsulated in a Future. It’s a good example of a simple abstraction over an asynchronous pattern, without forcing the implementation to be asynchronous, or use any particular model of asynchronous processing. As the Netflix Tech Blog: Functional Reactive in the Netflix API with RxJava shows nicely, Futures are great when all you need is concurrent processing of a set of similar tasks, but as soon as any of them want to depend on each other or execute conditionally you get into a form of "nested callback hell". Reactive Programming provides an antidote to that.

Map-reduce and fork-join Abstractions over parallel processing are useful and there are many examples to choose from. Map-reduce and fork-join that have evolved recently in the Java world, driven by massively parallel distributed processing (MapReduce and Hadoop) and by the JDK itself in version 1.7 (Fork-Join). These are useful abstractions but (like deferred results) they are shallow compared to FRP, which can be used as an abstraction over simple parallel processing, but which reaches beyond that into composability and declarative communication.

Coroutines A "coroutine" is a generalization of a "subroutine" — it has an entry point, and exit point(s) like a subroutine, but when it exits it passes control to another coroutine (not necessarily to its caller), and whatever state it accumulated is kept and remembered for the next time it is called. Coroutines can be used as a building block for higher level features like Actors and Streams. One of the goals of Reactive Programming is to provide the same kind of abstraction over communicating parallel processing agents, so coroutines (if they are available) are a useful building block. There are various flavours of coroutines, some of which are more restrictive than the general case, but more flexible than vanilla subroutines. Fibers (see the discussion on Event Machine) are one flavour, and Generators (familiar in Scala and Python) are another.

## Reactive Programming in Java

Java is not a "reactive language" in the sense that it doesn’t support coroutines natively. There are other languages on the JVM (Scala and Clojure) that support reactive models more natively, but Java itself does not until version 9. Java, however, is a powerhouse of enterprise development, and there has been a lot of activity recently in providing Reactive layers on top of the JDK. We only take a very brief look at a few of them here.

Reactive Streams is a very low level contract, expressed as a handful of Java interfaces (plus a TCK), but also applicable to other languages. The interfaces express the basic building blocks of Publisher and Subscriber with explicit back pressure, forming a common language for interoperable libraries. Reactive Streams have been incorporated into the JDK as java.util.concurrent.Flow in version 9. The project is a collaboration between engineers from Kaazing, Netflix, Pivotal, Red Hat, Twitter, Typesafe and many others.

RxJava: Netflix were using reactive patterns internally for some time and then they released the tools they were using under an open source license as Netflix/RxJava (subsequently re-branded as "ReactiveX/RxJava"). Netflix does a lot of programming in Groovy on top of RxJava, but it is open to Java usage and quite well suited to Java 8 through the use of Lambdas. There is a bridge to Reactive Streams. RxJava is a "2nd Generation" library according to David Karnok’s Generations of Reactive classification.

Reactor is a Java framework from the Pivotal open source team (the one that created Spring). It builds directly on Reactive Streams, so there is no need for a bridge. The Reactor IO project provides wrappers around low-level network runtimes like Netty and Aeron. Reactor is a "4th Generation" library according to David Karnok’s Generations of Reactive classification.

Spring Framework 5.0 (first milestone June 2016) has reactive features built into it, including tools for building HTTP servers and clients. Existing users of Spring in the web tier will find a very familiar programming model using annotations to decorate controller methods to handle HTTP requests, for the most part handing off the dispatching of reactive requests and back pressure concerns to the framework. Spring builds on Reactor, but also exposes APIs that allow its features to be expressed using a choice of libraries (e.g. Reactor or RxJava). Users can choose from Tomcat, Jetty, Netty (via Reactor IO) and Undertow for the server side network stack.

Ratpack is a set of libraries for building high performance services over HTTP. It builds on Netty and implements Reactive Streams for interoperability (so you can use other Reactive Streams implementations higher up the stack, for instance). Spring is supported as a native component, and can be used to provide dependency injection using some simple utility classes. There is also some autoconfiguration so that Spring Boot users can embed Ratpack inside a Spring application, bringing up an HTTP endpoint and listening there instead of using one of the embedded servers supplied directly by Spring Boot.

Akka is a toolkit for building applications using the Actor pattern in Scala or Java, with interprocess communication using Akka Streams, and Reactive Streams contracts are built in. Akka is a "3rd Generation" library according to David Karnok’s Generations of Reactive classification.

## Why Now?

What is driving the rise of Reactive in Enterprise Java? Well, it’s not (all) just a technology fad — people jumping on the bandwagon with the shiny new toys. The driver is efficient resource utilization, or in other words, spending less money on servers and data centres. The promise of Reactive is that you can do more with less, specifically you can process higher loads with fewer threads. This is where the intersection of Reactive and non-blocking, asynchronous I/O comes to the foreground. For the right problem, the effects are dramatic. For the wrong problem, the effects might go into reverse (you actually make things worse). Also remember, even if you pick the right problem, there is no such thing as a free lunch, and Reactive doesn’t solve the problems for you, it just gives you a toolbox that you can use to implement solutions.

## Conclusion

In this article we have taken a very broad and high level look at the Reactive movement, setting it in context in the modern enterprise. There are a number of Reactive libraries or frameworks for the JVM, all under active development. To a large extent they provide similar features, but increasingly, thanks to Reactive Streams, they are interoperable. In the next article in the series we will get down to brass tacks and have a look at some actual code samples, to get a better picture of the specifics of what it means to be Reactive and why it matters. We will also devote some time to understanding why the "F" in FRP is important, and how the concepts of back pressure and non-blocking code have a profound impact on programming style. And most importantly, we will help you to make the important decision about when and how to go Reactive, and when to stay put on the older styles and stacks.
