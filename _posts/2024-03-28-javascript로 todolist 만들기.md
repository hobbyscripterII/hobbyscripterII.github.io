---
title: "javascript로 todolist 만들기"
excerpt: "javascript를 사용해 todolist를 만들어보자!"
date: 2024-03-28 13:59:00 +09:00
writer: 이주영
categories: javascript
tags: [javascript, html, css]
toc: true
toc_sticky: true
---
![9ade7e0309e9e9c0a4aca5d7b2e166f5](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/d7121bcf-2bcf-4b38-b76a-7e03b613f93c){: width="50%"}

개인 프로젝트를 진행할 때 혼자서 프론트 작업도 해야하다보니 프론트 개념이 많이 부족하다는 것을 깨달았다.

그래서 면접 준비 전 유튜브에 '코딩 앙마'님의 dom 강의를 보고 **"직접 dom과 node를 제어하여 todolist를 만들어보자!"** 는 계획이 있었는데 면접 끝나고 계획대로 ver.1은 완성되어 블로깅으로 남긴다.

물론 todolist도 여러 기능을 추가하면 한도 끝도 없지만 1단계는 마무리랄까.

그리고 onclick 보다는 addEventListener를 지향하기 위해 노력했다.

## 1. todolist div 생성

```html
<body>
    <div id="div-todolist-wrapper">
        <div id="div-todolist" class="form-control">
            <h1>오늘의 할 일</h1>
            <span>
                <input type="text" class="form-control" id="text" placeholder="할 일을 입력해주세요.">
                <button type="button" class="btn btn-primary" id="btn-save">저장</button>
            </span>
            <ol id="ol-todolist">
            </ol>
        </div>
    </div>
</body>
```
css는 bootswatch의 [sketchy](https://bootswatch.com/sketchy/){:target="_blank"}를 이용하였다.

[font](https://beomdolee.com/%EC%82%AC%EA%B0%81%EC%82%AC%EA%B0%81/){:target="_blank"}도 테마에 맞춰 바꿔주었다.

위의 html 소스코드는 아래와 같이 출력된다.

![image](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/e649b661-85ca-4144-8f1e-3947d3e3831d)

## 2. '저장' 버튼 클릭 시 div 내 동적 요소 생성

이제부터는 내가 강의보고 배운 것들을 응용하며 만든 부분이다.

물론 구현하면서 필요한 부분들은 검색하며 만들었다.

하나씩 짚어보자!

```js
const todolist = document.getElementById('ol-todolist');

document.querySelector('#btn-save').addEventListener('click', function() {
    // 노드 제어를 통한 동적 요소 생성
    const newLi = document.createElement('li'); // todolist ul에 들어갈 새로운 요소 추가
    const newText = document.createTextNode(text.value); // newLi에 들어갈 새로운 텍스트 노드 추가
    const newBtn = document.createElement('button'); // newLi에 들어갈 새로운 요소 추가
    newBtn.classList.add('btn', 'btn-danger'); // 버튼에 클래스 추가(bootswatch 클래스)
    newBtn.id = 'btn-del'; // 해당 요소에 id 추가
    newBtn.type = 'button'; // 해당 요소에 type 추가
    newBtn.innerText = '삭제'; // text 추가

    // appendChild는 하나의 자식만 가질 수 있지만 append는 여러 개의 자식을 가질 수 있다.
    newLi.append(newText, newBtn); // li 태그와 button 태그를 같이 추가
    todolist.appendChild(newLi);
});
```

챕터1에서 html 소스코드에 넣어놓은 ol 태그가 있다. <br> 해당 태그는 순서가 있는 목록을 만들어주며 li 태그를 이용해 item의 list를 지정할 수 있다.

getElementById를 통해 ol 요소를 가져온 후 '저장' 버튼을 눌렀을 때 위의 코드가 실행된다.

ol 요소에 새로운 li 요소를 추가하기 위해 createElement를 사용했다. <br> li 요소 생성 후 createTextNode를 통해 사용자가 입력한 텍스트를 텍스트 노드로 생성한다.

'삭제' 버튼도 필요하기 때문에 button 요소도 새로 생성 한 후 classList.add를 통해 해당 요소에 class를 추가한다. <br> 참고로 btn과 btn-danger는 bootswatch의 css를 입히기 위한 bootswatch 클래스이다.

새로 만든 삭제 버튼에 id, type, innerText로 새로운 아이디, 타입, 버튼에 들어갈 텍스트를 추가해준다.

![스크린샷 2024-03-28 150629](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/30d0d303-bb1b-48c0-ab2c-5f0b00408db9)

li라는 부모 노드에 자식 노드로 newText와 newBtn을 추가하면 <span style="background-color: #F7DDBE">주황색 테두리</span>까지 완성되고, todolist라는 id를 가진 ol 요소에 li를 추가하면 <span style="background-color: #FFDCE0">빨강색 테두리</span>까지 완성되는 것이다.(결과적으로 ol에 li 요소를 추가해줘야 클라이언트 측에서 출력되어 보여진다)

## 3. 클릭한 '삭제' 버튼에 해당하는 부모 노드(li) 삭제

```js
document.addEventListener('click', function(e) {
    if(e.target.id == 'btn-del') {
        const target = e.target.parentNode;
        const targetValue = target.innerText.replace('삭제', '');
        const parentNode = target.parentNode;
        
        if(confirm(`${targetValue}을(를) 삭제하시겠습니까?`)) {
            alert('할 일이 삭제되었습니다.');
            parentNode.removeChild(target);
        } else {
            alert('삭제가 취소되었습니다.');
        }
    }
});
```

html 문서 내에서 클릭 이벤트 발생 시 이벤트가 발생한 타겟의 id가 btn-del이라면('삭제' 버튼 이라면) 위의 if문을 실행한다.

빠른 이해를 위해 todolist에 3개의 할 일을 추가했다.

완성된 html 문서는 아래와 같다.

```html
<ol id="ol-todolist">
    <li>
        밥 짱 잘먹기
        <button class="btn btn-danger" id="btn-del" type="button">삭제</button>
    </li>
    <li>
        자바스크립트로 todolist 짱 잘만들기
        <button class="btn btn-danger" id="btn-del" type="button">삭제</button>
    </li>
    <li>
        산책 나가기
        <button class="btn btn-danger" id="btn-del" type="button">삭제</button>
    </li>
</ol>
```

- `const target = e.target.parentNode`
    - 삭제 버튼의 부모 노드를 찾는데 삭제 버튼의 부모 노드는 li 요소이다.

- `const targetValue = target.innerText.replace('삭제', '')`
    - 해당 이벤트 부모 노드의 모든 innerText를 가져오기 때문에 버튼의 '삭제' text도 같이 가져온다. 그래서 replace로 '삭제'는 없애줬다. 하지만 별로 나이스하지 않아서 추후에 수정할 예정이다.

- `const parentNode = target.parentNode`
    - 해당 노드의 부모 노드인 ol 요소를 가져온다. 부모 노드에서 이벤트가 발생한 요소를 삭제하기 위해 가져왔다.

confirm로 '밥 짱 잘먹기를 삭제하시겠습니까?'와 같은 확인 창을 띄우고 사용자가 '확인' 버튼을 누를 경우 '할 일이 삭제되었습니다.'와 같은 알림 창을 띄운 후 부모 노드에서 이벤트가 발생한 노드를 삭제한다!

## 4. 결과

결과는 아래와 같다.

![javascript로 todolist 만들기(1)](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/82088c85-2d25-47d0-aae3-793be6756446)