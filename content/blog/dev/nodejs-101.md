---
title: 내가 node.js를 처음 배웠을 때 알았으면 좋았을 것들
date: 2021-07-16 23:07:29
category: dev
thumbnail: { thumbnailSrc }
draft: false
---

## 노드 프로젝트의 시작, package.json.

- 노드 프로젝트는 package.json이라는 파일과 함께 시작한다. package.json은 '패키지'라는 이름처럼, 노드 프로젝트가 가진 많은 정보들이 한 데 집약된 곳이다. 
- package.json 안에는 프로젝트의 이름, 만든 사람, 라이센스 등에 대한 일반적인 정보부터 **dependencies**, **scripts**등의 노드가 제공하는 특수한 정보가 적혀있다.
- `npm install express` 커맨드를 통해 express 패키지를 설치해보자. package.json의 dependencies에 새로운 항목이 추가되고, node_modules 디렉토리가 새로 생성된 것을 확인할 수 있다.
- dependencies는 해당 노드 프로젝트가 구동되기 위한 기반 모듈의 리스트와 각 버전이 명시되어 있다. 때문에 node_modules 없이도 package.json 파일만 있다면 언제든 `npm install` 명령어를 통해 node_modules를 다시 만들어낼 수 있다. 


## 이걸 챙겨가렴, Starter pack.

### [express](https://expressjs.com/)
- Node.js로 서버를 구축하는 데의 사실상의 표준(*De facto standard*). 라우팅에 필요한 각종 모듈을 제공한다. 
- Express가 제공하는 middleware, globalRouter 기능을 활용해 scalable한 서비스를 구축 가능하다. 
- [Koa](https://koajs.com/)라는 대체제가 있음.

### [morgan](https://github.com/expressjs/morgan)
- HTTP 리퀘스트에 대한 로거(logger) 기능을 제공하는 미들웨어.
- express와 함께 사용할 경우 시너지가 난다.
- 라우팅 단의 middleward에 morgan을 심어두면 알록달록 아름다운 로그를 간편하게 남길 수 있음.

```javascript
import express from "express";
import morgan from "morgan";

const app = express();       
const logger = morgan("dev") // 파라미터로 로그 스타일 선택 가능

app.use(logger);             // 미들웨어로 logger 전달
app.use('/', (req, res) => {
    ...
});
```

### [babel](https://babeljs.io/)
- 우리가 사용하고 싶은 ES6 이상의 최신 자바스크립트는 애석하게도 모든 웹에서 통용되지 않는다. 낮은 버전의 브라우저를 사용하는 사람들이 있기 때문. 
- Backward compatibility를 위해 이전 버전의 Javascript 문법을 사용해야 하는데, 이는 언어의 발전이 가져다주는 수혜를 모두 버리게 되는 것. 
- babel은 JS compiler로 최신 문법을 compatible한 문법으로 변환해준다! 

### [nodemon](https://nodemon.io/)
- 노드몬은 소스코드의 변화를 감지해 자동으로 서버를 재시작한다. 
- 코드 수정 -> 코드 중단 -> 코드 재컴파일의 반복 노동으로부터 당신을 해방시켜준다!
- 프로젝트에 사용하기 전 [preset](https://babeljs.io/docs/en/babel-preset-env) 설정을 마칠 것.

## 배운 것을 합쳐보자

스타터팩을 이어 붙이면 다음과 같다. 우리는 `Node.js`를 이용해 서버를 만들 것이다. 서버는 클라이언트가 보낸 요청(request)을 적절히 처리해 응답(respond) 만들어 보낸다. 이 과정을 라우팅이라고 한다. `express`는 노드의 라우팅을 돕는다. `morgan`은 라우팅 과정을 돕는다. 

서버 코드를 작성할 때, 우리는 최신 문법이 적용된 자바스크립트 코드로 작성할 것이다. 이 코드는 `babel`에 의해 backward compatible한 문법으로 변환된다. `Nodemon`은 코드의 변화를 감지해 자동으로 서버를 재시작할 것이다.

Node.js는 `node my_code.js`와 같은 스크립트를 통해 자바스크립트 코드를 실행할 수 있게 한다. babel과 nodemon의 기능을 이용하기 위해선, 코드 실행 시에 몇 가지 옵션을 추가해야 한다. 

```shell
$ nodemon --exec babel-node my_code.js
```

서버를 시작하려 할 때마다 위와같은 코드를 입력해야 한다면 매우 번거로울 것이다. 이때, 앞서 언급된 package.json의 scripts 기능을 활용할 수 있다.

```json
// package.json
"scripts": {
    "start": "nodemon --exec babel-node my_code.js"
}
```

위와 같이 scripts 안에 KEY-VALUE 쌍을 등록하면 `npm run KEY` 명령어를 통해 `VALUE` 스크립트를 실행할 수 있게 된다. 즉, 복잡한 스크립트의 별명(alias)을 등록해놓고 사용할 수 있는 것.


---

Node.js 서버를 구축할 때 알아두면 좋은 간단한 지식과 몇 가지 모듈 플러그인을 추천해보았다. 이 글을 읽는 누군가는 부디 헤매지 않고, 멋진 프로젝트를 진행해가길 바란다.