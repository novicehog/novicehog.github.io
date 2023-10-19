---
layout: single
title:  "react의 state사용하기"
categories: 
    - react
tag: [react]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2023-10-19
last_modified_at: 2023-10-19

---

## react에서의 자료 저장하기
react에서 값을 저장하는 방법은 두 가지가 있습니다.

하나는 let, var와 같은 변수를 선언하여 사용하는 것이고,<br>
나머지 하나는 state를 사용하는 것입니다.

### 일반적인 변수 선언하여 저장하기
아래와 같이 변수를 선언하여 그 값을 사용하는 방법이 있습니다.

```jsx
function App() {
  var myVar = "내가 저장한 값"; // 변수 선언
  return (
    <div className="App">
      <h1>{myVar /* 변수 사용 */}</h1>         
    </div>
  );
}
```


![image](https://github.com/novicehog/comments/assets/131991619/3f81be79-f7b6-4db9-967f-c206b04d5211)

<br>


### state를 통해 저장하기
react는 state라는 변수와 비슷한 역할을 하는 기능을 제공합니다.

react에서 기본적인 state 다음과 같습니다.

```jsx
let [값을_저장할_이름, 저장된_값을_바꾸는_기능_이름] = useState("저장할값");
```

<br>

이것이 동작하는 원리는 먼저 useState는 Array로 `[값을_저장할_곳, 값을_바꾸는_기능]`를 반환합니다.

```jsx
let [값을_저장할_이름, 저장된_값을_바꾸는_기능_이름] = [값을_저장할_곳, 값을_바꾸는_기능];
```

<br>

이때 자바스크립트의 문법 중 하나인 구조분해가 사용되어 저장이 됩니다.<br>
구조분해의 예를 보면 다음과 같습니다.

```jsx
var[a, b] = [10, 20];
//a는 10이 저장됨
//b는 20이 저장됨
```

<br>

그렇기 때문에 `값을_저장할_이름`에는 `값을_저장할_곳`가 저장되고<br>
`저장된_값을_바꾸는_기능_이름` 에는 `저장된_값을_바꾸는_기능`이 저장됩니다.

여기서 우리는 `값을_저장할_이름`을 변수처럼 사용할 수 있습니다.

다음은 위의 일반적인 변수를 사용한 코드를 state로 대체한 코드입니다.

```jsx
import {useState} from 'react';

function App() {
  //var myVar = "내가 저장한 값";
  let [myVar, SetMyBar] = useState("내가 저장한 값");
  return (
    <div className="App">
      <h1>{myVar  /* state 사용 */}</h1>
    </div>
  );
}

export default App;
```

똑같이 잘 되는 모습을 볼 수 있습니다.

![image](https://github.com/novicehog/comments/assets/131991619/e6d5ac8a-d2b6-4408-9b48-bced76be6674)


## state를 쓰는 이유
그럼 state를 사용하는 이유가 무엇이냐 하면 일반적인 변수와 다른 차이점이 한 가지 있습니다.

그것은 바로 값이 바뀔 때 나타나는데<br>
일반적인 변수는 값이 바뀌어도 `바뀐 값을 페이지에 적용`하는 것은 `따로 작업이 필요`합니다.

하지만 state의 경우 값이 수정되면 `자동으로 페이지가 다시 렌더링`됩니다.

즉, 우리는 `정적으로 다룰 값`이라면 `일반적인 변수`를 사용하면 되고,<br>
값이 `동적으로 바뀌는 변수`라면 `state를 사용`하면 됩니다.



## state 값 바꾸기
state의 값은 전용 기능을 통해 바꿔야합니다.

다음은 버튼을 누르면 값이 바뀌는 코드입니다.
```jsx
function App() {
  //var myVar = "내가 저장한 값";
  var [myVar, SetMyBar] = useState("내가 저장한 값");

  return (
    <div className="App">
      <button onClick={() => {SetMyBar(20)}}>클릭</button>
      <h1>{myVar  /* state 사용 */}</h1>
    </div>
  );
}
```

![image](https://github.com/novicehog/comments/assets/131991619/96597a25-9129-4bec-871f-5bfa093aad0b)

![image](https://github.com/novicehog/comments/assets/131991619/cc0c7b3f-74bb-429f-8b84-0e4b3265647e)


### 여러 값 함께 저장하기
state의 경우 다음과 같이 여러 값을 한 state에 저장할 수 있습니다.

```jsx
var [a, b] = useState(['첫번째값', '두번째값']);
```

이 경우 `a[0] 으로 "첫번째값"`값을 `a[1]로 "B두번째값"`의 값을 사용할 수 있습니다.

```jsx
function App() {
  //var myVar = "내가 저장한 값";
  var [a, setA] = useState(['첫번째값', '두번째값']);

  return (
    <div className="App">
      <h1>{a[0]}</h1>
      <h2>{a[1]}</h2>
    </div>
  );
}
```

![image](https://github.com/novicehog/comments/assets/131991619/4213df39-3e2b-43d0-be3e-a8f35c2b3b9a)

<br>

값의 사용은 그렇다하고 중요한건 값을 바꿀때인데 이렇게 한 state에 array로 여러 값이 저장된 경우에
`바꿀 값만이 아닌 다시 모든 값을 대입`해줘야 합니다.

다음은 클릭을 누르면 값이 바뀌는 코드입니다.
```jsx
function App() {
  //var myVar = "내가 저장한 값";
  var [a, setA] = useState(['첫번째값', '두번째값']);
  function Change(){
    setA(['바뀐 첫번째 값', '두번째값']);
  }

  return (
    <div className="App">
      <button onClick={Change}>버튼</button>
      <h1>{a[0]}</h1>
      <h2>{a[1]}</h2>
    </div>
  );
}
```

![image](https://github.com/novicehog/comments/assets/131991619/c72571ab-70b7-415f-8511-8a1e96f9f53b)

![image](https://github.com/novicehog/comments/assets/131991619/6ee7cd3f-367e-4c07-8fb5-a00f0d6e00ba)

잘 바뀝니다.


### state값을 바꿀 때 유의해야할 점
state가 array의 값을 저장할 때는 값을 바꿀 때 독립적인 카피본을 만들어서 수정하는 것이 좋다.

바로 위의 예제를 다음과 같이 바꿔서 적용하면 된다.
```jsx
function App() {
  //var myVar = "내가 저장한 값";
  var [a, setA] = useState(['첫번째값', '두번째값']);
  function Change(){
    let copy = [...a]; // 값 복사
    copy[0] ="바뀐 첫번째 값";
    setA(copy); 
  }

  return (
    <div className="App">
      <button onClick={Change}>버튼</button>
      <h1>{a[0]}</h1>
      <h2>{a[1]}</h2>
    </div>
  );
}
```