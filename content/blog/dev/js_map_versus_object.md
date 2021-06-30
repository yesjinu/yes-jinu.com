---
title: Javascript, Map vs. Object
date: 2021-06-30 20:06:17
category: dev
thumbnail: { thumbnailSrc }
draft: false
---

![image](../img/thankyou.png)

## Intro.
회사에서 간단한 로직을 작성하는 중이었다. `map` 안에 `map`이 들어있는 nested map형태의 데이터를 순회하며 가장 하위에 있는 `number[]`  타입의 값을 가져와야하는 일이었다.

JS가 제공하는 `forEach`, `map` 등의 함수를 사용하는 중에, 문득 몇 가지 궁금증이 생겼다.  map은 `forEach` 함수를 써서 iterate할 수 있지만, object는 불가능하다. 그렇기 때문에 `object.keys()`나 `for...in`과 같은 우회적인 방법을 통해 iterate 동작을 수행하는 것이다. 

내 머릿속에 그려진 JS의 map과 object의 심적표상은 비슷하게 동작하고 있었다. 둘 다 key, value의 쌍이라는 점. key를 넣으면 value가 퐁하고 튀어나온다. **map과 object는 어떻게 다르길래, 굳이 map이라는 자료구조를 새로 만들어야 했을까?**

이런 경우 MDN이 제공하는 문서를 꼼꼼히 읽어보는 것이 정답이지만, 클라이언트 단에서 코드 완성을 기다리고 있었기 때문에, 일단은 빠르게 코드 완성/디플로이하고 퇴근 이후에 차분히 읽어보고 정리하기로 한다.

## Map vs. Object
[Map - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)

Object와 map은 비슷한 구석이 많다. 둘 다 key, value 쌍을 저장할 수 있고, key값으로 value를 받아올 수 있고, key값을 삭제할 수 있고, key값에 value가 저장되어 있는지 여부를 알 수 있다. 실제로 map이 구현되기 이전까지는 object가 map의 역할을 수행했다고 한다.

그런데 둘 사이에는 분명 차이가 있다. mdn이 제공하는 예시를 살펴보면서 둘 사이의 차이점을 알아보도록 하자.

### Accidental keys.
- Map에는 default key값이 없다. Map에는 오직 명시적으로 넣은 key값만이 존재한다.
- Object는 prototype을 가지고 있다. 크롬 개발자 도구를 통해 object 내부를 살펴보다가, 내가 선언하지 않은 prototype 값들이 들어있는 것을 본 적이 있을 것이다. [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)는 JS에서 제공하는 기본 데이터 타입으로 내부에 built-in method들이 구현되어 있다. 이 부분은 다음에 자세히 알아보기로 하자.

### Key types
- Map은 key로 어떤 value든 사용할 수 있다. 심지어 함수, object까지!
- Object의 key값은 String 혹은 symbol이어야 한다.

### Key order
- Map은 key값의 insertion order를 기억하고 있다. 따라서 map object를 iterate할 때 직관적으로 사용할 수 있다.
- Object의 key값은 순서를 유지하는 것처럼 보이지만, 항상 그렇지는 않았다고. JS의 역사에 따라 변화해온듯. object에 의해 property order가 지켜질 것이라 의존하지 말 것.

### Size
- Map은 item의 개수를 `size` property를 통해 쉽게 구할 수 있다.
- Object는 item의 개수를 일일이 세야한다.

### Iteration
- Map은 `iterable`하다! `for...of`, `forEach` 등 JS가 제공하는 iterate 이기를 사용할 수 있다.
- Object는 iteration protocol을 구현하지 않았기 때문에 iterable하지 않다. [Iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterable_protocol)에 의해 Iterable하기 위해서는 object 내부에 `@@iterator` property를 가지고 있어야 한다.

### Performance
- Map은 잦은 추가, 삭제가 있는 시나리오에서 더 나은 성능을 내도록 최적화되어 있다.
- Object에는 잦은 추가, 삭제 시나리오에 대한 최적화가 되어 있지 않다.

### Serialization and parsing
- Map은 serialization(object -> JSON), parsing(JSON -> object)을 지원하는 native support가 없다. 
- Object는 serialization, parsing을 지원하는 native support가 있다. `JSON.stringify()`, `JSON.parse()`가 바로 그것!

### 그 외 Map에 관해 몰랐던 사실
- Map은 insertion 순서를 기억한다.
- Primitive value는 물론 function, object도 key값으로 사용될 수 있다.
  - 아래와 같이 쓸 수 있는 것. 이걸 어떤 상황에 쓸 수 있을지? :thinking_face:


```javascript
const m = new Map();
function f () {
  // do something
}

m.set(f, some_value);
m.get(f); // some_value
```

	
