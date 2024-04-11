---
title: "naver maps, geocoding, search api를 활용하여 검색한 업체 지도 위에 표시하기"
date: 2024-04-11 10:30:00 +09:00
writer: 이주영
categories: api naver
tags: [java, spring boot, json, VO, naver, maps, geocoding, search, api]
toc: true
toc_sticky: true
---
이전 포스팅 [naver geocoding 반환 값(json) VO로 파싱 후 클라이언트 화면에 출력하기](https://hobbyscripterii.github.io/posts/naver-geocoding-%EB%B0%98%ED%99%98-%EA%B0%92(json)-VO%EB%A1%9C-%ED%8C%8C%EC%8B%B1-%ED%9B%84-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8-%ED%99%94%EB%A9%B4%EC%97%90-%EC%B6%9C%EB%A0%A5%ED%95%98%EA%B8%B0/){: target="_blank"}에서 주소를 입력하면 해당 주소의 도로명 주소와 좌표가 찍히게 끔 했었는데 이런 로직을 구현하면서 한가지 생각이 들었다.

<br>

![image](https://i.pinimg.com/originals/42/1a/d7/421ad737247cd308d3a2d7007ccad50b.gif){: width="50%" .normal}

**"이 로직은 내가 무조건 주소만 입력해야만 좌표를 받을 수 있는거네? 그럼 내가 주소를 검색해야하는데 그건 너무 비효율적인 것 같아."**

그러다가 네이버 검색 api를 알게되었고 네이버 검색 api 기능 중 검색했을 때 지역 내 업체를 찾아주는 `지역`이라는 기능을 사용하기로 마음먹었다.

그렇게되어 작성해본 이번 기능의 로직은 아래와 같다. 로직의 흐름은 클라이언트 입장이 아닌 개발자 입장으로 작성했다.

> 1. 사용자가 input 폼에 `맛집 이름`을 적고 `맛집 검색하기` 버튼을 클릭한다.
> 2. ajax를 통해 네이버 검색 api(maps)를 호출하여 `맛집 이름`을 보낸다.
> 3. 네이버 검색 api의 반환 값(json)을 VO로 파싱한 후 return 값으로 받는다.
> 4. 동적 div 폼을 만들어 해당 이름에 해당하는 맛집 리스트를 화면에 출력한다.
> 5. 특정 업체를 클릭했을 경우 ajax를 통해 네이버 좌표 api(geocoding)를 호출하여 특정 업체의 `주소`를 보낸다.
> 6. 네이버 지도 api를 통해 return 받은 좌표에 마커를 찍는다.

이번 포스팅은 길어질 것 같은데 그래도 차근차근 살펴보자!

## 1. 네이버 검색 api 애플리케이션 등록
해당 내용은 구글에 많을테니 생략하도록 하겠다.

## 2. 맛집 이름 검색 시 동적 div 생성 후 출력
### 2-1. js 작성
```js
document.addEventListener('click', (e) => {
    if(e.target.id == 'btn-coordinate') {
        const addr = document.getElementById('input-addr').value;

        if(!addr) { alert('업체명을 입력해주세요.'); } // search 사용 시에는 업체명으로 변경
        else {
            $.ajax({
                type: 'get', url: '/naver/search', data: {'addr': encodeURIComponent(addr)}, dataType: 'json',
                success: (data) => {
                    const items = data.items, divAddrConsole = document.getElementById('div-addr-console');

                    items.forEach((i) => {
                        const newDiv = document.createElement('div');
                        newDiv.classList.add('div-restaurant-info');
                        newDiv.setAttribute('data-addr', `${i.address}`);
                        newDiv.setAttribute('onclick', `findCoordinate(this)`);
                        const newP = document.createElement('p');
                        const title_ = i.title.replaceAll('<b>', '');
                        const title = title_.replaceAll('</b>', '');
                        const titleNode = document.createTextNode(`업체명: ${title}`);
                        newP.append(titleNode);
                        const newP2 = document.createElement('p');
                        const categoryNode = document.createTextNode(`카테고리: ${i.category}`);
                        newP2.append(categoryNode);
                        const newP3 = document.createElement('p');
                        const addressNode = document.createTextNode(`주소: ${i.address}`);
                        newP3.append(addressNode);
                        newDiv.append(newP);
                        newDiv.append(newP2);
                        newDiv.append(newP3);
                        divAddrConsole.append(newDiv);
                    });
                },
                error: (x, e) => { console.log(x); console.log(e); }
            })
        }
    }
});
```

`맛집 검색하기` 버튼을 클릭할 경우 버튼의 아이디를 통해 해당 타겟에서 이벤트가 발생했음을 감지하고 위의 로직을 실행한다.

업체명이 있을 경우 ajax를 통해 검색 api를 호출한다.

newDiv라는 새로운 요소를 생성하고 css를 먹인 div-restaurant-info 클래스를 추가한다.

data-addr은 데이터 속성인데 이름을 지정하고 해당 속성에 ajax에서 반환된 주소 값을 넣었다.

onclick 속성도 추가해서 만들어놨던 메소드를 추가한다. findCoordinate(this)는 해당 div를 클릭할 경우 네이버 좌표 api를 호출하는 메소드이다. 밑에서 다룰 예정이다.

이렇게 setAttribute를 사용하면 아래와 같이 속성을 지정해준다.

```html
<div id="div-addr-console">
    <div class="div-restaurant-info" data-addr="대구광역시 중구 동성로3가 111 해쉬Hash" onclick="findCoordinate(this)">
        <p>업체명: 해쉬</p>
        <p>카테고리: 이탈리아음식&gt;스파게티,파스타전문</p>
        <p>주소: 대구광역시 중구 동성로3가 111 해쉬Hash</p>
    </div>
    <!-- 생략 -->
</div>
```

### 2-2. controller 작성
```java
    @GetMapping("/search")
    @ResponseBody
    public NaverSearchGetVo naverSearch(String addr) {
        StringBuilder stringBuilder = new StringBuilder(); // 가변 문자열을 처리하기 위한 StringBuilder 객체 생성
//        NaverCoordinateGetVo naverCoordinateGetVo = new NaverCoordinateGetVo();
        NaverSearchGetVo naverSearchGetVo = new NaverSearchGetVo();

        // url - geocoding api 호출 시 사용되는 url
        // addr - html에서 ajax로 받아오는 클라이언트 측에서 입력한 텍스트 형식의 주소
        String url = "https://openapi.naver.com/v1/search/local.json?query=" + addr + "&display=10&start=1&sort=sim";

        // HttpClient - http와 통신하기 위한 객체
        HttpClient httpClient = HttpClientBuilder.create().build();
        // get 메소드 생성 + get 요청 시 통신할 url
        HttpGet httpGet = new HttpGet(url);
        // get 요청 시 header에 담을 정보(geocoding api 호출 시 필요)
        httpGet.addHeader("X-Naver-Client-Id", searchClientId);
        httpGet.addHeader("X-Naver-Client-Secret", searchClientSecret);

        try {
            // get 요청
            HttpResponse httpResponse = httpClient.execute(httpGet);

            // response 데이터 읽기
            InputStreamReader inputStreamReader = new InputStreamReader(httpResponse.getEntity().getContent(), "UTF-8");
            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

            // bufferedReader의 값을 다 읽을 때 까지 while문 실행
            String current = "";

            while ((current = bufferedReader.readLine()) != null) { // 1. bufferedReader에 있는 값이 current에 대입된다.
                stringBuilder.append(current); // 2. bufferedReader의 값이 담긴 current를 stringBuilder에 추가한다.
            }

            // json to vo 작업
            ObjectMapper objectMapper = new ObjectMapper(); // json을 java 객체로 변환하기 위해 ObjectMapper 객체 생성
            naverSearchGetVo = objectMapper.readValue(stringBuilder.toString(), NaverSearchGetVo.class); // 역직렬화

            bufferedReader.close(); // bufferedReader에 있는 문자열을 다 읽으면 닫아준다.
        } catch (Exception e) {
            e.printStackTrace();
        }
        return naverSearchGetVo;
    }
```
contorller 설명은 주석을 통해 확인하도록 한다.

<br>

여기까지 로직을 구현했을 때 아래와 같이 출력된다.

![image](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/17345e37-2633-4aa6-80f8-84a3a35f20e5)

## 3. 특정 업체 클릭 시 네이버 좌표 api 호출
### 3-1. js 작성
특정 업체(특정 div 폼)을 클릭할 경우 실행되는 js 코드는 아래와 같다.

```js
function findCoordinate(item) {
    const addr = item.dataset.addr;

    $.ajax({
        type: 'get', url: '/naver/maps', data: {'addr': encodeURIComponent(addr)}, dataType: 'json',
        success: (data) => {
            const addr = data.addresses[0];
            const divAddrConsole = document.getElementById('div-addr-console');

            // 마커 표시
            var map = new naver.maps.Map('map', { center: new naver.maps.LatLng(addr.y, addr.x), zoom: 19, zoomControl: true });
            var marker = new naver.maps.Marker({ position: new naver.maps.LatLng(addr.y, addr.x), map: map });

            // 정보창 표시
            var contentString = [`<div class="iw_inner"><p>${addr.jibunAddress}</p><p><a href="https://search.naver.com/search.naver?query=${addr.jibunAddress}" target="_blank">해당 주소 네이버로 검색하기</a></p></div>`].join('');
            var infowindow = new naver.maps.InfoWindow({ content: contentString });
            naver.maps.Event.addListener(marker, "click", (e) => {
                if (infowindow.getMap()) { infowindow.close(); }
                else { infowindow.open(map, marker); }
            });
            infowindow.open(map, marker);
        },
        error: (x, e) => { console.log(x); console.log(e); }
    })
}
```

특정 div 폼을 클릭했을 때 `onclick="findCoordinate(this)"`가 실행되는데 this에는 클릭한 폼의 정보들이 담겨져있다. 

`const addr = item.dataset.addr;` 이 부분은 특정 div 폼을 클릭했을 때 해당 데이터 속성 중 addr이라는 이름을 가진 속성의 데이터를 가져오는 작업이다. 내가 위에서 주소를 넣어놨으므로 주소를 가져오는 것이다.

이후 해당 주소를 네이버 좌표 api에 보내면서 return 값으로 주소 정보와 좌표를 얻어온다.

이후 네이버 좌표 api 레퍼런스를 통해 마커와 정보창을 화면에 출력시켰다.

### 3-2. controller 작성
```java
@GetMapping("/maps")
    @ResponseBody
    public NaverCoordinateGetVo naverMap(String addr) {
        StringBuilder stringBuilder = new StringBuilder(); // 가변 문자열을 처리하기 위한 StringBuilder 객체 생성
        NaverCoordinateGetVo naverCoordinateGetVo = new NaverCoordinateGetVo();

        // url - geocoding api 호출 시 사용되는 url
        // addr - html에서 ajax로 받아오는 클라이언트 측에서 입력한 텍스트 형식의 주소
        String url = "https://naveropenapi.apigw.ntruss.com/map-geocode/v2/geocode-js?query=" + addr;

        // HttpClient - http와 통신하기 위한 객체
        HttpClient httpClient = HttpClientBuilder.create().build();
        // get 메소드 생성 + get 요청 시 통신할 url
        HttpGet httpGet = new HttpGet(url);
        // get 요청 시 header에 담을 정보(geocoding api 호출 시 필요)
        httpGet.addHeader("X-NCP-APIGW-API-KEY-ID", mapsClientId);
        httpGet.addHeader("X-NCP-APIGW-API-KEY", mapsClientSecret);

        try {
            // get 요청
            HttpResponse httpResponse = httpClient.execute(httpGet);

            // response 데이터 읽기
            InputStreamReader inputStreamReader = new InputStreamReader(httpResponse.getEntity().getContent(), "UTF-8");
            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

            // bufferedReader의 값을 다 읽을 때 까지 while문 실행
            String current = "";

            while ((current = bufferedReader.readLine()) != null) { // 1. bufferedReader에 있는 값이 current에 대입된다.
                stringBuilder.append(current); // 2. bufferedReader의 값이 담긴 current를 stringBuilder에 추가한다.
            }

            // json to vo 작업
            ObjectMapper objectMapper = new ObjectMapper(); // json을 java 객체로 변환하기 위해 ObjectMapper 객체 생성
            naverCoordinateGetVo = objectMapper.readValue(stringBuilder.toString(), NaverCoordinateGetVo.class); // 역직렬화

            bufferedReader.close(); // bufferedReader에 있는 문자열을 다 읽으면 닫아준다.
        } catch (Exception e) {
            e.printStackTrace();
        }
        return naverCoordinateGetVo;
    }
```

api 호출 시 중복되는 코드가 많아 정리를 하긴 해야하는데 포스팅 이후에 진행할거라 일단 이렇게 확인해보자. 처음에 설명했던 controller와 로직이 거의 같다.

<br>

이렇게 작성된 로직을 실행하면 아래와 같이 출력된다.

![image](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/0a6be87a-bf55-4892-8409-c1dfac3fdd78)

## 4. 결과
해당 로직을 구현하기 위해 총 3가지 api를 사용해봤다.

네이버 지도 api(maps) <br>
네이버 좌표 api(geocoding) <br>
네이버 검색 api(search)

이렇게 호출한 api로 아래와 같이 멋있는 기능을 구현할 수 있었다!

![naver maps, geocoding, search api를 활용하여 검색한 업체 지도 위에 표시하기](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/e0983d79-3e1b-4628-b81d-fc43dea0307c)
