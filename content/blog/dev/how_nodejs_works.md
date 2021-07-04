---
title: Node.js 실행구조 분석
date: 2021-06-29 23:06:55 +0900
category: dev
thumbnail: { nodejs.svg }
draft: false
---

너무도 많은 사람들이 사용하지만, 대부분의 사람들이 내부적으로 어떻게 작동하는지 알지 못한채로 사용하는 Node.js. 오늘은 node.js 인터페이스에서 딱 한 단계 더 깊이 들어가 node.js를 분석하고자합니다. 

![img](../img/nodejs.svg)

> 편의를 위해 이하 글에서는  node.js를 '노드', 자바스크립트를 JS로 줄여부를게요 :) 

## Node.js에 대한 간략한 설명
노드는 2009년 JSConf EU에서 라이언 달이라는 사람에 의해 처음 공개되었습니다. 당시에 가장 널리 쓰이던 웹 서버는 Apache HTTP 서버였는데, 이 프로그램은 **concurrent connection을 핸들링하는 데에 한계**가 있었습니다. 내부적으로 synchronous하게 동작하도록 짜여있어 하나의 큰 long-running function를 실행하다가 전체 프로세스가 멈춰버리는 경우가 많았다고 하네요. 

[위키피디아](https://en.wikipedia.org/wiki/Node.js)에 의하면 노드는 'V8 엔진을 이용해 브라우저 바깥에서 자바스크립트 코드를 실행하는 오픈소스, 크로스플랫폼 자바스크립트 런타임 환경.'라고 되어있습니다. V8 엔진은 자바스크립트 코드를 실행하는 엔진의 한 종류입니다. 이외에도 Chakra, SpiderMonkey 등의 엔진이 있습니다.

이 두 가지 사실을 조합해 정리하자면, 노드는 **당시의 웹 서버가 가진 문제점을 해결하기 위해** 고안된 **asynchronous하게 동작하는 JS 런타임 환경**입니다. 

## 노드에 대한 심적 표상 만들기
먼저 큰 틀에서 '노드가 무엇을 하는지, 어떻게 작동시킬 수 있는지'에 대한 **심적 표상**을 그리는 것이 중요합니다. 유저 인터페이스의 관점에서 노드를 살펴보겠습니다. 사실 노드는 설명하기 민망할 정도로 간단하게 사용할 수 있습니다.

JS로 작성한 코드를 node로 실행하면 됩니다 (응?) 앞서 말했듯이 노드는 브라우저가 아닌 환경에서 JS 코드를 실행할 수 있는 실행환경이기 때문입니다. node 커맨드와 함께 실행하고 싶은 JS 파일을 인자로 넘기면 (마법처럼) 코드가 실행됩니다.
`node hello.js` 

추가로 노드는 굉장히 큰 오픈소스 생태계를 가지고 있습니다. npm이라는 패키지 매니저를 통해 이 모듈들을 설치해 자신의 프로젝트에 사용할 수 있는데요, 때문에 필요한 것을 일일이 만들 필요없어 개발에 매우 속도를 붙일 수 있습니다. 노드가 인기있는 이유이기도 하지요.
`npm install mysql`

## 우리가 답해볼 질문
이번 포스팅은 노드의 가장 큰 두 가지 특징에 대한 **다음 두 가지 질문에 답변하는 것을 목표로** 진행하려 합니다.
1. 노드는 어떻게 브라우저 바깥에서 JS를 실행하는가?
2. 노드는 어떻게 (single thread이면서) I/O operation을 비동기로 처리하는가?

먼저, 세부사항에 대해 알아보기 전에 노드 내부의 큼직한 구조도를 살펴봅시다. 노드는 여러 가지의 큰 모듈들의 집합으로 구성되어 있는데요, 이중에서 주요한 역할을 하는 두 가지의 큰 기둥이 **V8과 libuv**입니다. 

![image](../img/nodejs_module_diagram.png)

- **V8**은 구글이 개발한 오픈소스 JS 엔진으로 JS를 native C++ 코드로 컴파일하는 역할을 맡습니다. V8은 자체적으로 70% C++와 30%의 JS로 구성되어 있습니다. 
- **libuv**는 비동기 I/O operation을 구현하는 엔진입니다. 100% C로 짜여져 있습니다.


우리가 앞서 물었던 문제는 결국 '`JS 코드`와 `V8`, `libuv` 3가지 컴포넌트가 서로 어떻게 상호작용하는지'에 그 해답이 있었습니다.

> 이 포스팅에서는 V8과 libuv가 어떻게 구현되어 있는지는 살펴보지 않고, 전체적인 실행 흐름에 집중하겠습니다.



## 1. 노드는 JS를 어떻게 실행하는가?
> **TLDR;** V8엔진이 JS 코드를 C++로 컴파일. 노드가 제공하는 모듈은 process.binding() 과정을 통해 C++ 코드와 연결되어 있음.


[Github repository of Node.js](https://github.com/nodejs/node)

노드의 소스코드에는 크게 2개의 메인 디렉토리가 있습니다.
- `lib` : JS 코드를 작성하는 데에 필요한 definition이 저장. 이외에도 node가 제공하는 라이브러리들이 있음(fs, crypto, path, http, etc.)
- `src` : C++로 구현. V8 엔진의 C++로 구현된 JS object(Array, Boolean)를 import해서 사용. lib 엔진을 이용해 OS에 access.

파일 시스템(fs)를 예시로 들어 동작을 설명하려 합니다. `fs.readFile`은 JS도 V8에도 속하지 않는 구현체로, 노드가 제공하는 함수입니다. 이 함수가 어떻게 실행되는지 살펴봄으로써 JS로 작성한 코드가 어떻게 V8과 연결되는지 살펴볼 수 있을 것입니다.

---
다음과 같은 코드를 작성했다고 가정해봅시다.
```javascript
const fs = require('fs')
...
// read file at some_path
fs.readFile(some_path, some_options, (err, data) => {
	// error handling
	// do something with data
});
```

호출한 fs 모듈(`fs.js`) 내부의 readFile은 다음과 같이 구현되어 있습니다. ([Link](https://github.com/nodejs/node/blob/1317252dfe8824fd9cfee125d2aaa94004db2f3b/lib/fs.js#L370-L399))
```javascript
function readFile(path, options, callback) {
  callback = maybeCallback(callback || options);
  options = getOptions(options, { flag: 'r' });
  const context = new ReadFileContext(callback, options.encoding);
  context.isUserFd = isFd(path); // File descriptor ownership

  if (options.signal) {
    context.signal = options.signal;
  }
  if (context.isUserFd) {
    process.nextTick(function tick(context) {
      ReflectApply(readFileAfterOpen, { context }, [null, path]);
    }, context);
    return;
  }

  if (checkAborted(options.signal, callback))
    return;

  const flagsNumber = stringToFlags(options.flag, 'options.flag');
  path = getValidatedPath(path);

  const req = new FSReqCallback();
  req.context = context;
  req.oncomplete = readFileAfterOpen;
  // binding.open() 찾았다 연결점
  binding.open(pathModule.toNamespacedPath(path),
               flagsNumber,
               0o666,
               req);
}
```

우리가 초점을 맞추고 있는 것은 어떻게 이 함수가 C++ 구현체와 연결되는가 하는 부분입니다. readFile 함수는 ReadFileContext 모듈을 이용해 파일을 읽는 데에 필요한 `context`(사이즈, 버퍼, 인코딩, etc.)를 초기화합니다. 몇 가지 작업을 거친 후, 마지막으로 `binding.open()` 를 호출합니다. 

같은 fs.js 파일에 binding은 `internalBinding('fs')`로 선언되어 있는데요([Link](https://github.com/nodejs/node/blob/1317252dfe8824fd9cfee125d2aaa94004db2f3b/lib/fs.js#L70)), internalBinding 함수는 인자로 넘겨진 것과 이름이 같은 같은 namespace를 찾아 로드하는 함수입니다. 이 함수를 통해 `binding` 은 'fs' namespace 내부에 있는 `src/node_file.cc`와 연결됩니다([Link](https://github.com/nodejs/node/blob/1317252dfe8824fd9cfee125d2aaa94004db2f3b/src/node_file.cc#L52)). 즉 binding.open()을 호출함으로써 JS 코드에서 node_file.cc 내부에 구현된 `open()` 함수를 사용할 수 있게 된 것입니다. 

```cpp
void FSReqCallback::Reject(Local<Value> reject) {
  MakeCallback(env()->oncomplete_string(), 1, &reject);
}

void FSReqCallback::ResolveStat(const uv_stat_t* stat) {
  Resolve(FillGlobalStatsArray(env(), use_bigint(), stat));
}

void FSReqCallback::Resolve(Local<Value> value) {
  Local<Value> argv[2] {
    Null(env()->isolate()),
    value
  };
  MakeCallback(env()->oncomplete_string(),
               value->IsUndefined() ? 1 : arraysize(argv),
               argv);
}
```
`node_file.cc` 내부에는 C++로 구현된 file system 함수와 callback logic이 구현되어 있습니다. 위 JS 코드에서 우리는 `const req = new FSReqCallback()`로 생성한 `req` 안에, JS 코드에서 인자로 받은 callback을 context 안에 담아 `binding.open()` 함수에 인자로 넘겼습니다.  `req`는 이 context property를 사용해 원하는 fs 동작을 수행하고, 동작이 완료되면 callback 합니다([Link](https://github.com/nodejs/node/blob/1317252dfe8824fd9cfee125d2aaa94004db2f3b/src/node_file.cc#L568-L584)).

## 2. 노드는 어떻게 비동기적으로 작동하는가?
> TLDR; 이벤트 루프를 통해 I/O operation을 워커 스레드에 위임. 결과값을 나중에 따로 확인하는 방식으로 동작함.

### 자바스크립트 런타임
먼저 자바스크립트 런타임에 대해 알아야 합니다. 자바스크립트는 싱글 스레드로 작동합니다. 이 말의 뜻은 하나의 프로세스에서 실행되는 자바스크립트 엔진(V8)은 하나의 `콜 스택`만을 제공한다는 것과 같습니다.

스택은 다들 아시다시피 abstract data type의 한 종류로 LIFO로 동작합니다. JS는 현재 실행하는 함수의 주소값을 스택에 push하고, 완료되면 stack top에서 pop하는 식으로 동작합니다. 스택이 하나 밖에 없다는 말은, 한 번에 하나의 함수만 실행할 수 있다는 뜻으로 해석할 수 있습니다.

만약 동기적인 자바스크립트 언어로 I/O operation(long-running function)을 포함한 코드를 실행하면 어떻게 될까요? 행렬 연산이나 이미지 프로세싱 같은 intensive한 동작이 콜 스택에 들어간다면, 이 동작이 완료되기 전까지 이후의 모든 코드가 실행되지 않고 blocking 되어 있을 겁니다. 이는 매우 바람직하지 않은 현상이죠. 

### 이벤트 루프
이벤트 루프는, (대부분 멀티 스레드로 동작하는) 시스템 커널에 업무를 분담함으로써 노드가 blocking 되어 있지 않고도 I/O operation을 수행할 수 있게 해줍니다. 커널에 업무를 위임하고, 완료되었는지 주기적으로 체크하는 식으로 작동하는 것이지요.  

![image](../img/nodejs_event_loop.png)

이벤트 루프는 노드가 시작될 때 같이 초기화됩니다. 이벤트 루프는 (앞서 살펴본) 콜 스택과 이벤트 큐를 활용해 동작합니다. Synchronous하게 동작하는 코드는 콜 스택에 쌓이면서 차례로 실행됩니다. 그러던 중 blocking I/O 나 복잡한 연산이 필요한 코드를 만나면, 이벤트 루프는 커널의 스레드 풀에서 적절한 스레드를 골라 해당 코드를 실행하는 업무를 위임하고, 다음 코드 실행을 재개합니다. 이 동작은 libuv에 의해 이루어집니다.

업무를 위임받은 스레드는 해당 I/O 업무를 수행하고 그 결과를 이벤트 큐에 push합니다. 이벤트 루프는 (loop 이름에 걸맞게) 콜 스택과 이벤트 큐를 계속 확인하고 있다가, 콜 스택이 비면 이벤트 큐에서 결과값을 pop해, 콜 스택에 push합니다. 그리고 이 콜 스택을 다시 실행하는 식입니다.

## 마무리
이렇게 노드가 내부적으로 동작하는 방식을 한 단계 깊게 들어가 알아보았습니다. 물론 이것 역시 꽤나 많은 부분이 추상화되어 있는 high-level에서의 분석이지만, 노드의 동작을 이해할 수 있는 첫 걸음이 되었기를 바랍니다. 추후 포스팅에서 더 자세한 내용을 다뤄보겠습니다 :)

## Reference
- [The Node.js Event Loop, Timers, and process.nextTick() | Node.js](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
- [Node.js Under The Hood #1 - Getting to know our tools - DEV Community](https://dev.to/khaosdoctor/node-js-under-the-hood-1-getting-to-know-our-tools-1465)
- [Node.js Beyond the Basics :: jsComplete](https://jscomplete.com/learn/node-beyond-basics)
- [Node JS Architecture - Single Threaded Event Loop - JournalDev](https://www.journaldev.com/7462/node-js-architecture-single-threaded-event-loop#node-js-architecture-8211-single-threaded-event-loop)