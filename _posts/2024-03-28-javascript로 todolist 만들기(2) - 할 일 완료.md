---
title: "javascript로 todolist 만들기(2) - 할 일 완료"
excerpt: "javascript를 사용해 todolist를 만들어보자!"
date: 2024-03-28 20:07:00 +09:00
writer: 이주영
categories: javascript
tags: [javascript, html, css]
toc: true
toc_sticky: true
---
이전 포스팅의 2번째 시간으로 이전에 만들었던 todolist와 다른 점은 일단 UI가 아래와 같이 수정되었다.

![image](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/e649b661-85ca-4144-8f1e-3947d3e3831d)

![image](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/00f0ff5b-b97b-49a8-8618-091313f34379)

todolist 원래 컨셉은 낙서장 같은 느낌이였는데 보다보니 지저분해 보여서 바꿨다.

그리고 보면 알겠지만 '진행중인 할 일'과 '완료된 할 일'이라는 새로운 기능이 생겼다.

아래 소스코드를 보면서 같이 파헤쳐보자!

## 1. 리스트 요소 생성

내가 말하는 `리스트 요소`는 '진행중인 할 일'과 '완료된 할 일'을 의미한다.

아래와 같이 추가되었다.

```html
<ol id="ol-todolist" class="form-control">
    <span id="span-text">진행중인 할 일</span>
</ol>

<ul id="ul-todolist-success" class="form-control">
    <span id="span-text">완료된 할 일</span>
</ul>
```

진행중인 할 일은 순서가 있어야 하므로 ol 태그를 통해 li 요소를 넣도록 만들었고, 완료된 할 일은 순서가 없어도 상관없으므로 ul 태그를 통해 li 요소를 넣도록 했다.

그리고 완료된 할 일의 li 요소들은 다른 투두리스트 앱들과 비슷하게 글씨 색상도 연하게 해주고 완료되었다는 의미로 글씨 중간에 줄도 그어주기 위해 다른 id로 지정했다.

## 2. js 작성

내가 만들어놓고도 내가 설명을 다 할 수 있을까 싶긴한데 일단 전체 소스코드는 아래와 같다.

```js
const todolist = document.getElementById('ol-todolist');

// >>>>> 재사용 위해 메소드로 추출
function addTodolist(text) {
    // 노드 제어를 통한 동적 요소 생성
    const newLi = document.createElement('li'); // todolist ul에 들어갈 새로운 요소 추가
    newLi.id = 'li-tasking';
    const newText = document.createTextNode(text); // newLi에 들어갈 새로운 텍스트 노드 추가

    const delBtn = document.createElement('button'); // newLi에 들어갈 새로운 요소 추가
    delBtn.classList.add('btn', 'btn-danger'); // 버튼에 클래스 추가(bootswatch 클래스)
    delBtn.id = 'btn-del'; // 해당 요소에 id 추가
    delBtn.type = 'button'; // 해당 요소에 type 추가
    delBtn.innerText = '삭제'; // text 추가

    // appendChild는 하나의 자식만 가질 수 있지만 append는 여러 개의 자식을 가질 수 있다.
    newLi.append(newText, delBtn); // li 태그와 button 태그를 같이 추가
    todolist.appendChild(newLi);
}

document.querySelector('#btn-save').addEventListener('click', function() {
    addTodolist(text.value); // * id가 text인 input에 value를 받아 옴
});

document.addEventListener('click', function(e) {
    // 삭제 버튼 클릭 시 해당 함수 실행
    if(e.target.id == 'btn-del') {
        const target = e.target.parentNode;
        const targetValue = target.innerText.replace('삭제', '');
        const parentNode = target.parentNode;

        if(confirm(`${targetValue}을(를) 삭제하시겠습니까?`)) {
            alert(`${targetValue}이(가) 삭제되었습니다.`);
            parentNode.removeChild(target);
        } else {
            alert('삭제가 취소되었습니다.');
        }
    }
});

document.addEventListener('dblclick', function(e) {
    // '진행중인 할 일' 내에 li 요소 더블 클릭 시 이벤트 발생
    if(e.target.id == 'li-tasking') {
        // 1. '완료된 할 일'에 li 추가하기 위한 세팅 작업
        const newLi = document.createElement('li');
        newLi.id = 'li-sus';
        const target = e.target;
        const targetValue = target.innerText.replace('삭제', '');
        const newText = document.createTextNode(targetValue);
        newLi.append(newText);
        
        const successTodolist = document.getElementById('ul-todolist-success');
        successTodolist.append(newLi);
        
        // 2. '진행중인 할 일'에서 클릭한 요소 삭제
        // target - li
        // parentNode - ol
        const parentNode = target.parentNode;
        parentNode.removeChild(target); // 부모 노드 호출 후 이벤트 발생한 자식 노드 삭제
    }

    // '완료된 할 일' 내에 li 요소 더블 클릭 시 이벤트 발생
    if(e.target.id == 'li-sus') {
        addTodolist(e.target.innerText); // * 해당 이벤트가 발생한 타겟의 text를 받아 옴

        const successTodolist = document.getElementById('ul-todolist-success');
        const target = e.target;
        successTodolist.removeChild(target);
    }
});
```

하나씩 살펴보자.

### 2-1. 재사용을 위한 메소드 추출

![image](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/fb2e5336-9d5e-4e1d-bbb3-109c6516be3e)

사용자가 '저녁 짱 맛있게 먹기'라는 할 일을 등록한다. → '진행중인 할 일'에 저장되어야 한다.

![image](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/988c14b8-82fe-4b67-ab4f-122b3b284c06)

사용자가 '완료된 할 일'에 있는 '저녁 짱 맛있게 먹기'를 더블 클릭한다. → 다시 '진행중인 할 일'에 저장되어야 한다.

같은 프로세스를 가진 addEventListener를 아래와 같이 하나의 함수로 따로 만들어 사용했다.

```js
// >>>>> 재사용 위해 메소드로 추출
function addTodolist(text) {
    // 노드 제어를 통한 동적 요소 생성
    const newLi = document.createElement('li'); // todolist ul에 들어갈 새로운 요소 추가
    newLi.id = 'li-tasking';
    const newText = document.createTextNode(text); // newLi에 들어갈 새로운 텍스트 노드 추가

    const delBtn = document.createElement('button'); // newLi에 들어갈 새로운 요소 추가
    delBtn.classList.add('btn', 'btn-danger'); // 버튼에 클래스 추가(bootswatch 클래스)
    delBtn.id = 'btn-del'; // 해당 요소에 id 추가
    delBtn.type = 'button'; // 해당 요소에 type 추가
    delBtn.innerText = '삭제'; // text 추가

    // appendChild는 하나의 자식만 가질 수 있지만 append는 여러 개의 자식을 가질 수 있다.
    newLi.append(newText, delBtn); // li 태그와 button 태그를 같이 추가
    todolist.appendChild(newLi);
}
```

그리고 아래와 같이 `1. '저장' 버튼을 눌렀을 때` 그리고 `2. '완료된 할 일'에서 특정 li 요소를 더블 클릭했을 때` addTodolist에 매개변수로 li의 text를 넘겨준다.

```js
document.querySelector('#btn-save').addEventListener('click', function() {
    addTodolist(text.value); // * id가 text인 input에 value를 받아 옴
});
```

```js
// '완료된 할 일' 내에 li 요소 더블 클릭 시 이벤트 발생
if(e.target.id == 'li-sus') {
    addTodolist(e.target.innerText); // * 해당 이벤트가 발생한 타겟의 text를 받아 옴

    const successTodolist = document.getElementById('ul-todolist-success');
    const target = e.target;
    successTodolist.removeChild(target);
}
```

### 2-2. '진행 중인 할 일'과 '완료된 할 일' 이동

사용자가 '진행 중인 할 일'의 특정 li 요소를 더블 클릭 할 경우에는 '완료된 할 일'로 가야하며, '완료된 할 일'의 특정 li 요소를 더블 클릭 할 경우에는 다시 '진행 중인 할 일'로 가야한다. 다른 list를 넘나들어야 하는 상황에서 프로세스를 어떻게 할까 하다가 아래와 같이 작성해봤다.

```js
document.addEventListener('dblclick', function(e) {
    // '진행중인 할 일' 내에 li 요소 더블 클릭 시 이벤트 발생
    if(e.target.id == 'li-tasking') {
        // 1. '완료된 할 일'에 li 추가하기 위한 세팅 작업
        const newLi = document.createElement('li');
        newLi.id = 'li-sus';
        const target = e.target;
        const targetValue = target.innerText.replace('삭제', '');
        const newText = document.createTextNode(targetValue);
        newLi.append(newText);
        
        const successTodolist = document.getElementById('ul-todolist-success');
        successTodolist.append(newLi);
        
        // 2. '진행중인 할 일'에서 클릭한 요소 삭제
        // target - li
        // parentNode - ol
        const parentNode = target.parentNode;
        parentNode.removeChild(target); // 부모 노드 호출 후 이벤트 발생한 자식 노드 삭제
    }

    // '완료된 할 일' 내에 li 요소 더블 클릭 시 이벤트 발생
    if(e.target.id == 'li-sus') {
        addTodolist(e.target.innerText); // * 해당 이벤트가 발생한 타겟의 text를 받아 옴

        const successTodolist = document.getElementById('ul-todolist-success');
        const target = e.target;
        successTodolist.removeChild(target);
    }
});
```

2개의 if문이 비슷한 로직으로 처리되는 것을 확인할 수 있다. 차이점이라면 '완료된 할 일'의 리스트에 추가할 때는 버튼이 필요없다는 것 정도일까..

설명은 주석으로 다 해놓은 것 같으니 패스하겠다.

- '진행 중인 할 일' 로직
    - li 생성
    - 이벤트 발생 타겟의 text에서 '삭제' 텍스트 없애기(버튼에 있는 값도 '텍스트 노드'이기 때문에 같이 포함된다)
    - 호출된 addTodolist의 매개변수 text를 텍스트 노드에 추가
    - li에 텍스트 노드 추가
    - '완료된 할 일' 요소 불러오기
    - '완료된 할 일' 리스트에 li 추가
    - '진행 중인 할 일'에 있는 해당 요소는 부모 노드에서 호출하여 삭제 처리

- '완료된 할 일' 로직
    - li 생성
    - addTodolist(text) 호출
        - li 생성
        - li id 지정
        - 호출된 addTodolist의 매개변수 text를 텍스트 노드에 추가
        - '삭제' 버튼을 위한 button 요소 생성
        - button 요소에 클래스 추가
        - button id 지정
        - button type 지정
        - button text 지정
        - li에 텍스트 노드 및 '삭제' 버튼 요소 추가
        - '진행 중인 할 일' 리스트에 li 추가
    - '완료된 할 일'에 있는 해당 요소는 부모 노드에서 호출하여 삭제 처리

## 3. 이슈 사항
### 3-1. addTodolist 호출 시 헷갈리는 매개변수 값

addTodolist를 호출할 때 li 요소에 있는 text를 보내줘야 했는데 '진행 중인 할 일'에서 매개변수 보낼 때랑 '완료된 할 일'에서 매개변수를 보낼 때 조금 헷갈렸다.

지금 보면 정말 별 거 아닌데 다음에 헷갈리는 일 없게 리마인드 겸 작성한다.

```js
document.querySelector('#btn-save').addEventListener('click', function() {
    addTodolist(text.value); // * id가 text인 input에 value를 받아 옴
});
```

일단 '저장' 버튼을 클릭 할 때는 html 문서 내에 있는 id가 text인 input 요소의 value 값을 매개변수로 보낸다.

```js
if(e.target.id == 'li-sus') {
    addTodolist(e.target.innerText); // * 해당 이벤트가 발생한 타겟의 text를 받아 옴

    const successTodolist = document.getElementById('ul-todolist-success');
    const target = e.target;
    successTodolist.removeChild(target);
}
```

그리고 '완료된 할 일'에서 특정 li를 더블 클릭할 경우에는 더블 클릭한 요소의 text를 매개변수로 보낸다.

이전에는 부모 노드까지 들락날락하면서 값을 가져왔던 터라 왜 안되지 하고 있었는데 당연히 li에서 이벤트가 발생했으니 해당 이벤트 타겟에서 innerText로 값을 보내주면 되는 일이였다!

### 3-2. 더블 클릭 시 보기싫은 블록 처리

![javascript로 todolist 만들기(3)](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/0bca2c55-1bc0-4713-9260-b55233198636)

더블 클릭할 때마다 블록 처리가 되는 현상이 보기 싫었다.

구글링으로 'html 드래그 금지'와 같은 검색어로 해결 방법을 찾아냈다.

```html
<body onselectstart='return false'>
```

body에다가 `onselectstart` 속성에 'return false'를 적용시켜 드래그를 금지하게 만들었다.

이후 깔끔하게 리스트 값이 넘어가는 것을 확인할 수 있었다!

## 4. 결과

결과물은 [해당 링크](https://hobbyscripterii.github.io/to-do-list.io/){:target="_blank"}를 통해 확인할 수 있다!

직접 들어가서 확인해보자!

## 5. 느낀 점

토이 프로젝트로 todolist를 진행해봤는데 확실히 직접 만들어봐야 머리가 트이는 걸 느낀다.

그리고 결과물이 눈에 바로바로 보여서 꽤나 재밌게 작업했던 프로젝트였던 것 같다.

그리고 KDT 과정에서 선생님께서 가르쳐주셨던 '중복 메소드는 죄악이다', 'jquery 보다는 javascript를 사용하자'와 같은 말씀을 해주셨는데 덕분에 java, spring이 아니더라도 조금씩 중복 메소드도 제거하고 jquery를 지양하는 코드도 짤 수 있었다.

즐거웠다!

todolist를 여기서 끝내는 건 아니고 일단락 짓는거긴 한데 필요할 때 마다 해당 프로젝트로 꾸준히 작업할 생각이다!

아니면 다른 javascript 프로젝트를 또 만들수도?