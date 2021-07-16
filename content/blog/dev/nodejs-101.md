---
title: 내가 node.js를 처음 배웠을 때 알았으면 좋았을 것들
date: 2021-07-16 23:07:29
category: dev
thumbnail: { thumbnailSrc }
draft: false
---

## package.json

- 노드 프로젝트는 package.json이라는 파일과 함께 시작한다. package.json은 '패키지'라는 이름처럼, 노드 프로젝트가 가진 많은 정보들이 한 데 집약된 곳이다. 
- package.json 안에는 프로젝트의 이름, 만든 사람, 라이센스 등에 대한 일반적인 정보부터 **dependencies**, **scripts**등의 노드가 제공하는 특수한 정보가 적혀있다.
- `npm install express` 커맨드를 통해 express 패키지를 설치해보자. package.json의 dependencies에 새로운 항목이 추가되고, node_modules 디렉토리가 새로 생성된 것을 확인할 수 있다.
- dependencies는 해당 노드 프로젝트가 구동되기 위한 기반 모듈의 리스트와 각 버전이 명시되어 있다. 때문에 node_modules 없이도 package.json 파일만 있다면 언제든 `npm install` 명령어를 통해 node_modules를 다시 만들어낼 수 있다. 

## 초기 셋업을 돕는 plugin

### babel
- 우리가 사용하고 싶은 ES6 스타일의 최신 자바스크립트는 애석하게도 모든 웹에서 통용되지 않는다. 아직 낮은 버전의 브라우저를 사용하는 사람들이 있기 때문이다. 때문에 backward compatibility를 위해 이전 버전의 Javascript 문법을 사용해 코드를 작성해야 하는데, 이는 최신 업데이트의 수혜를 모두 버리게 되는 것이다. babel은 JS compiler로 최신 문법을 compatible 문법으로 변환해준다. 