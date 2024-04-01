---
title: "javascript event listener 호출 시 이전 event 중복 호출 이슈"
date: 2024-04-01 22:43:00 +09:00
writer: 이주영
categories: javascript
tags: [javascript, event listener, issue]
toc: true
toc_sticky: true
---
해당 이슈는 내게 골칫덩어리와 같은 이슈였다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/e30b7303-512b-4f23-9a77-aca00573c253){: width="50%" .normal}

나와 같은 이슈가 발생한 분들께 조금 더 도움이 되었으면 하는 마음에 조금 더 딥하게 다뤄보고자 해당 이슈를 포스팅하게 되었다.

## 1. 이슈 원인

![javascript event listener 호출 시 이전 event 중복 호출 이슈(1)](https://github.com/hobbyscripterII/about-play/assets/135996109/b69b1288-2455-4669-a14a-4e2b1c6b8c45)

해당 이슈를 설명하자면 아래와 같다.

1. 사용자가 첫번째 동영상을 등록한다.(동영상 등록은 유튜브 상단의 링크를 통해 정규식 검증 후 비디오 아이디를 추출하는 방식이다)
2. 사용자가 두번째 동영상을 등록한다.
3. (예외 발생) 사용자가 첫번째로 등록한 동영상 뒤에 두번째로 등록한 동영상이 출력된다.
4. (예외 발생) 게시글 저장 버튼을 누를 경우 첫번째로 등록한 비디오 아이디 또한 마지막에 등록한 비디오 아이디로 덧입혀진다.
5. (예외 발생) 게시글 확인 시 등록한 동영상 2개 모두 마지막에 등록한 비디오 아이디로 출력되어 동영상이 중복되어 출력된다.

![javascript event listener 호출 시 이전 event 중복 호출 이슈(2)](https://github.com/hobbyscripterII/about-play/assets/135996109/564304e8-9272-4585-8c6a-41ddc3fabc83)

해당 이슈를 다시 확인하기위해 console.log로 확인해보니 **이전에 발생한 이벤트 리스너를 한번 더 호출 후 해당 이벤트 리스너가 실행되었다.**

이로써 **이슈의 원인은 이벤트 타겟이 중복되어 실행되는 것을 확인할 수 있었다.**

## 2. 이슈 확인

이슈를 확인하기위해 소스코드를 확인해보자.

```js
    document.addEventListener('click', (e) => {
        if(e.target.id == 'btn-modal-insert-video') { insertVideo(e); }

        if(e.target.id == 'btn-video-del') {
            if(confirm('삭제된 링크는 복구할 수 없습니다. 삭제하시겠습니까?')) {
                const targetNode = e.target.parentNode.parentNode; const targetNode2 = targetNode.nextSibling.nextSibling; const targetNode3 = targetNode.previousSibling; const targetNodeParent = targetNode.parentNode;
                targetNodeParent.removeChild(targetNode); targetNodeParent.removeChild(targetNode2); targetNodeParent.removeChild(targetNode3); }
            else { alert('삭제가 취소되었습니다.'); }
        }

        if(e.target.className == 'btn-close' || e.target.className == 'btn btn-secondary modal-close') { modalClose(); }
    })

    function insertVideo(e) {
        const videoIdArr = [];
        modalOpen();

        document.getElementById('input-video-url').addEventListener('change', () => {
            const divVideo = document.querySelector('.div-video');
            const match = videoUrl.val().match(/[?&]v=([^?&]+)/);

            if(match && match[1]) {
                videoIdArr.push(match[1]);
                const videoId = videoIdArr[videoIdArr.length - 1];
                divVideo.innerHTML = `<object type="text/html" width="465px" height="300px" data="https://www.youtube.com/embed/${videoId}"></object>`;
            } else {
                $('.div-video').html('<p>해당 video를 불러올 수 없습니다. 링크 확인 후 다시 시도해주세요.</p>');
            }
        });

        document.getElementById('btn-video-save').addEventListener('click', () => {
            const videoId = videoIdArr[videoIdArr.length - 1];
            const target = e.target.closest('tr');
            console.log('target = ', target);
            const newTr = document.createElement('tr');
            newTr.innerHTML = `<th colspan="3"><object class="object-video" type="text/html" width="700px" height="400px" data="https://www.youtube.com/embed/${videoId}"></object></th>`;
            target.before(newTr);
            const descriptionEl = target.nextSibling.nextSibling.firstChild.firstChild;
            descriptionEl.dataset.videoId = `${videoId}`;

            modalClose();
        });
    }
```

**1. click 시 event listener 발생 함수 호출**

```js
document.addEventListener('click', (e) => {
```

html 문서 내 요소를 클릭할 경우 발생하는 이벤트 리스너로 `e`에는 이벤트가 발생한 타겟의 정보를 담고있다.

**2. '링크 등록' 버튼 클릭 시 insertVideo 함수 호출**

```js
if(e.target.id == 'btn-modal-insert-video') { insertVideo(e); }
```

insertVideo 함수는 아래에서 설명한다.
이벤트가 발생한 타겟의 id가 '링크 등록' 버튼일 경우 insertVideo(e)를 호출시킨다.

**3. insertVideo(e) 호출**

```js
function insertVideo(e) {
        const videoIdArr = [];
        modalOpen();

        document.getElementById('input-video-url').addEventListener('change', () => {
            const divVideo = document.querySelector('.div-video');
            const match = videoUrl.val().match(/[?&]v=([^?&]+)/);

            if(match && match[1]) {
                videoIdArr.push(match[1]);
                const videoId = videoIdArr[videoIdArr.length - 1];
                divVideo.innerHTML = `<object type="text/html" width="465px" height="300px" data="https://www.youtube.com/embed/${videoId}"></object>`;
            } else {
                $('.div-video').html('<p>해당 video를 불러올 수 없습니다. 링크 확인 후 다시 시도해주세요.</p>');
            }
        });

        document.getElementById('btn-video-save').addEventListener('click', () => {
            const videoId = videoIdArr[videoIdArr.length - 1];
            const target = e.target.closest('tr');
            console.log('target = ', target);
            const newTr = document.createElement('tr');
            newTr.innerHTML = `<th colspan="3"><object class="object-video" type="text/html" width="700px" height="400px" data="https://www.youtube.com/embed/${videoId}"></object></th>`;
            target.before(newTr);
            const descriptionEl = target.nextSibling.nextSibling.firstChild.firstChild;
            descriptionEl.dataset.videoId = `${videoId}`;

            modalClose();
        });
    }
```

핵심 로직을 설명하기 전 위의 코드도 설명해야하므로 아래에서 차차 살펴보자.

**4. 모달창 열기**

```js
modalOpen()
```

위의 코드는 아래의 함수를 호출한다.

```js
function modalOpen() { document.getElementById('modal-insert-video').style.display = 'block'; }
```

**5. 모달 창의 input에 youtube link 삽입 시 정규식 검증**

```js
        document.getElementById('input-video-url').addEventListener('change', () => {
            const divVideo = document.querySelector('.div-video');
            const match = videoUrl.val().match(/[?&]v=([^?&]+)/);

            if(match && match[1]) {
                videoIdArr.push(match[1]);
                const videoId = videoIdArr[videoIdArr.length - 1];
                divVideo.innerHTML = `<object type="text/html" width="465px" height="300px" data="https://www.youtube.com/embed/${videoId}"></object>`;
            } else {
                $('.div-video').html('<p>해당 video를 불러올 수 없습니다. 링크 확인 후 다시 시도해주세요.</p>');
            }
        });
```

'input-video-url'은 말그대로 video url을 삽입하는 부분이다. 정규식 검증을 통해 yotube link가 맞는지 확인 후 맞다면 해당 값에서 video id를 추출한다. 이후 videoIdArr이라는 배열에 video id를 넣고 동영상을 출력시킨다. 만약 사용자가 입력한 데이터가 youtube link가 아니라면 해당 video를 불러올 수 없다는 에러 메세지를 출력시킨다.

**6. 동영상 '저장' 버튼 클릭 시 클라이언트에 출력**

이슈가 발생한 코드는 아래와 같다.

```js
        document.getElementById('btn-video-save').addEventListener('click', () => {
            const videoId = videoIdArr[videoIdArr.length - 1];
            const target = e.target.closest('tr');
            console.log('target = ', target);
            const newTr = document.createElement('tr');
            newTr.innerHTML = `<th colspan="3"><object class="object-video" type="text/html" width="700px" height="400px" data="https://www.youtube.com/embed/${videoId}"></object></th>`;
            target.before(newTr);
            const descriptionEl = target.nextSibling.nextSibling.firstChild.firstChild;
            descriptionEl.dataset.videoId = `${videoId}`;

            modalClose();
        });
```

위의 코드도 아래에서 설명하겠다.

```js
const target = e.target.closest('tr');
```

이벤트 타겟이 발생한 부분은(e에 담긴 정보는) '링크 등록' 부분이며 해당 요소에서 제일 가까운 tr 태그를 찾는다.

'링크 등록' 버튼 클릭 시의 이벤트 타겟을 내가 갖고있어야 해당 요소 위에 tr을 새로 만들어 동영상 출력을 할 수 있기 때문에 동영상 '저장' 시의 이벤트 타겟은 필요 없었다.

모달 창에서 내가 이전에 클릭한 이벤트들을 linux의 history 명령어로 찾는 것 마냥 하고 싶었는데 이벤트 버블링을 사용하더라도 잘 안될 것 같아서 동영상 '저장' 시에는 이벤트 타겟은 가지고오지 않았다.(조금 더 나이스한 코드를 짜고 싶었는데 그게 잘 안되긴 했다. 방법이 있다면 수정하고 싶다)

```js
const newTr = document.createElement('tr');
newTr.innerHTML = `<th colspan="3"><object class="object-video" type="text/html" width="700px" height="400px" data="https://www.youtube.com/embed/${videoId}"></object></th>`;
target.before(newTr);
```

새로운 tr 요소를 만들어 tr 요소에 사용자가 입력한 video id를 넣고 '링크 등록' 버튼의 이전 노드에 새로운 tr 요소를 추가한다.

```js
const descriptionEl = target.nextSibling.nextSibling.firstChild.firstChild;
descriptionEl.dataset.videoId = `${videoId}`;
```

플레이리스트 동영상 설명을 간략하게 적을 수 있는 textarea의 data 속성에 video id를 추가한다.(이것은 추후 게시글 등록 시 테이블 내 tr을 반복문으로 돌며 플레이리스트의 설명글과 video id를 빼내오기위한 작업이다)

**7. 작업 완료 후 모달창 닫기**

```js
modalClose();
```

위의 코드는 아래의 함수를 호출한다.

```js
function modalOpen() { document.getElementById('modal-insert-video').style.display = 'none'; }
```

여기까지 확인했을 때 문제가 되어보이는 것이 있다면 클릭 시 이벤트를 계속 끌고오는 것이다. 하지만 위에서 언급했듯이 모달 창에서 내가 이전에 클릭한 이벤트들을 linux의 history 명령어로 찾는 것 마냥 할 수 있는 능력이 안됐기에 '링크 등록' 버튼을 클릭할 경우 발생할 수 있는 모든 경우의 수를 해당 함수 내에 끌고 들어왔다.

## 3. 이슈 해결

동영상 '저장' 버튼을 클릭했을 경우 이전에 등록한 비디오 링크가 있다면 이전에 실행된 이벤트 타겟이 한번 더 실행된다는 것은 알고 있었다.

그리고 내가 작성한 소스코드에서 한 메소드에 많은 기능들을 넣은 것도 알고있긴 했는데 지금 로직으로는 바꿀 방법이 없었다.

해당 이슈를 해결하기 위해 이벤트 버블링, 이벤트 캡처링, 이벤트 전파 중단 등 여러 포스팅을 확인했지만 내가 원하는 방법이 나오지 않았다.

그러다 인파님의 포스팅을 보고 하나 발견한 것이 있었다.

바로 `addEventListener`의 3번째 인자 값에 `once: true`를 주는 것이였다.

```js
document.getElementById('btn-video-save').addEventListener('click', () => {
    const videoId = videoIdArr[videoIdArr.length - 1];
    const target = e.target.closest('tr');
    const newTr = document.createElement('tr');
    newTr.innerHTML = `<th colspan="3"><object class="object-video" type="text/html" width="700px" height="400px" data="https://www.youtube.com/embed/${videoId}"></object></th>`;
    target.before(newTr);
    const descriptionEl = target.nextSibling.nextSibling.firstChild.firstChild;
    descriptionEl.dataset.videoId = `${videoId}`;

    modalClose();
}, { once : true });
```

해당 방법으로 이벤트 리스너를 한번만 호출하여 여러번 발생하는 이벤트를 방지할 수 있었으며 이후  addEventListener에 대해 다시 한번 더 알아보았다.

### addEventListener의 인자 값 목록

```js
addEventListener(type, listener, useCapture);
```

- **type** - 수신할 이벤트 유형을 나타낸다.(click, change 등이 해당된다)
- **listener** - 지정한 이벤트를 수신할 객체이다. 나는 주로 위에서 익명 함수를 통해 구현했다.
- **options** - 해당 이벤트의 특징을 지정할 수 있다. 옵션은 아래와 같다.
    - **capture** - 이벤트 대상은 dom 트리의 하위에 위치한 자손 event target으로 캡처링 단계에서 event를 잡아낼 때 true로 명시하여 잡아낼 수 있다. 기본 값은 false이다.
    - **once** - 해당 함수가 최대 1번만 동작해야 함을 나타내는 boolean 값으로 `true로 지정할 경우 해당 함수를 호출한 후에 스스로를 대상에서 제거한다.` 기본 값은 false이다.
    - **passive** - 해당 함수에서 preventDefault()를 절대 호출하지 않을 것을 나타내는 boolean 값으로 기본 값은 false이다.

> `preventDefault()` 현재 이벤트의 기본 동작을 중단시킨다.

---
**참고 레퍼런스** <br>
[이벤트 리스너 제거 & 이벤트 한번만 실행 방법](https://inpa.tistory.com/entry/JS-%F0%9F%93%9A-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EC%A0%9C%EA%B1%B0-%ED%95%9C%EB%B2%88%EB%A7%8C-%EC%8B%A4%ED%96%89%EB%90%98%EA%B2%8C-%ED%95%98%EA%B8%B0-removeEventListener-once)