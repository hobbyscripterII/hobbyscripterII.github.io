---
title: "naver geocoding을 활용한 주소 및 좌표 검색 API 호출하기"
date: 2024-04-09 23:06:00 +09:00
writer: 이주영
categories: api
tags: [java, spring boot, naver, api, maps, geocode]
toc: true
toc_sticky: true
---
최근에 유튜브 플레이리스트 추천 프로젝트를 진행하면서 쉬다가 풍자가 진행하는 `또간집`이라는 채널을 알게 되었다.

![image](https://i.pinimg.com/564x/9a/de/7e/9ade7e0309e9e9c0a4aca5d7b2e166f5.jpg){: width="50%" .normal}

풍자가 맛있다고 하는 맛집들을 혹시나 가게 된다면 찾기 편하게 맛집 지도가 있었으면 좋겠다는 생각이 들었고 프로젝트로 녹여내기 위해 네이버에서 제공하는 API를 사용하게 되었다.

## 0. 활용 API 및 라이브러리 목록
- **Naver Maps API** [^1]
    - 네이버에서 제공하는 지도 API이며 javascript 형태의 지도 플랫폼으로 웹 서비스 또는 애플리케이션에 지도 기능을 구현할 수 있다.
- **Naver Geocoding API** [^2]
    - 네이버에서 제공하는 주소 검색 API로 주소를 텍스트로 받아 검색 결과로 주소 목록과 세부 정보를 JSON 형태로 받아온다.
- **Apache HttpClient** [^3]
    - HTTP를 사용하여 통신하는 범용 라이브러리로 해당 라이브러리에서 제공하는 클래스들을 사용하여 쉽게 HTTP 통신이 가능하다.

## 1. 의존성 주입
위에서 3번째로 설명한 Apache에서 제공하는 HttpClient 라이브러리를 추가한다.

[Apache HttpClient](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient){: target="_blank"}

난 gradle을 사용중이므로 아래와 같이 작성했다.

```gradle
dependencies {
    implementation group: 'org.apache.httpcomponents', name: 'httpclient', version: '4.5.14'
}
```

## 2. Naver API 애플리케이션 등록
[NAVER CLOUD PLATFORM](https://www.ncloud.com/?language=ko-KR){: target="_blank"}에 들어가 우측 상단의 `콘솔`을 클릭하면 아래와 같이 페이지가 나온다.

![스크린샷 2024-04-09 212112](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/8f66ff6f-1038-4556-bd23-5ccbee30cc4a)

Services - AI·NAVER API - Maps를 순서대로 클릭한다.

<br>

![image](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/ca69bdd2-3076-4738-b3e3-6dc9a4e5be72)

난 이미 등록해놓은 애플리케이션이 있기 때문에 위와 같이 출력되는데 처음엔 아무것도 없다.

상단의 `Application 등록`을 눌러보자.

<br>

![image](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/8eb39eae-9601-47a9-a777-85b119384b11)

- **Application 이름** - repository명처럼 작성하면 된다.
- **Service 선택** - Web Dynamic Map, Geocoding, Reverse Geocoding을 선택한다. 사용하지 않을 서비스라면 굳이 체크하지 않아도된다.

![image](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/454d3a08-983d-4626-a69f-b9c28e1a8166)

- **Web 서비스 URL** - 지도를 출력할 경로와 geocoding과 통신할 컨트롤러의 url을 입력한다. 나같은 경우 `http://localhost:8080`은 네이버 지도를 띄울 경로이며 `http://localhost:8080/naver/map`은 geocoding api를 호출할 때 타고 갈 컨트롤러 url이다.
- **Android 앱 패키지 이름** - 선택 사항인데 `com.패키지명.프로젝트명`과 같이 작성하면 된다.

## 3. controller 작성
```java
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.HttpClientBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.io.BufferedReader;
import java.io.InputStreamReader;

@Slf4j
@Controller
public class NaverMapApiController {
    // application.yaml에 있는 네이버 지도 api 애플리케이션 id와 secret key
    @Value("${api.naver.map.client-id}") private String clientId;
    @Value("${api.naver.map.client-secret}") private String clientSecret;

    @GetMapping("/naver/map")
    @ResponseBody
    public String naverMap(String addr) {
        StringBuilder stringBuilder = new StringBuilder(); // 가변 문자열을 처리하기 위한 StringBuilder 객체 생성
        // url - geocoding api 호출 시 사용되는 url
        // addr - html에서 ajax로 받아오는 클라이언트 측에서 입력한 텍스트 형식의 주소
        String url = "https://naveropenapi.apigw.ntruss.com/map-geocode/v2/geocode-js?query=" + addr;

        // HttpClient - http와 통신하기 위한 객체
        HttpClient httpClient = HttpClientBuilder.create().build();
        // get 메소드 생성 + get 요청 시 통신할 url
        HttpGet httpGet = new HttpGet(url);
        // get 요청 시 header에 담을 정보(geocoding api 호출 시 필요)
        httpGet.addHeader("X-NCP-APIGW-API-KEY-ID", clientId);
        httpGet.addHeader("X-NCP-APIGW-API-KEY", clientSecret);

        try {
            // get 요청
            HttpResponse httpResponse = httpClient.execute(httpGet);
            // response 데이터 읽기
            InputStreamReader inputStreamReader = new InputStreamReader(httpResponse.getEntity().getContent(), "UTF-8");
            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

            String current = "";

            // bufferedReader의 값을 다 읽을 때 까지 while문 실행
            while ((current = bufferedReader.readLine()) != null) { // 1. bufferedReader에 있는 값이 current에 대입된다.
                stringBuilder.append(current); // 2. bufferedReader의 값이 담긴 current를 stringBuilder에 추가한다.
            }
            bufferedReader.close(); // bufferedReader에 있는 문자열을 다 읽으면 닫아준다.
        } catch (Exception e) {
            e.printStackTrace();
        }
        return stringBuilder.toString();
    }
}
```

참고로 `요청 url`과 `요청 header명`은 [여기](https://api.ncloud-docs.com/docs/ai-naver-mapsgeocoding){: target="_blank"}서 자세히 확인할 수 있다.

## 4. html 작성
html 코드 작성 전에 naver api를 사용하기 위해 다음과 같이 추가한다.

```html
<script type="text/javascript" src="https://oapi.map.naver.com/openapi/v3/maps.js?ncpClientId=YOUR_CLIENT_ID"></script>
<script type="text/javascript" src="https://oapi.map.naver.com/openapi/v3/maps.js?ncpClientId=YOUR_CLIENT_ID&submodules=geocoder"></script>
```

파라미터에 있는 ncpClientId는 위에서 만들었던 애플리케이션 정보에 있는 client id이다.

이후 알아서 코드를 작성한다.

<br>

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>또간집 맛집 지도</title>
    <link href="/common.css" rel="stylesheet">
    <script src="/common.js"></script>
    <script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>
    <script type="text/javascript" src="https://oapi.map.naver.com/openapi/v3/maps.js?ncpClientId=YOUR_CLIENT_ID"></script>
    <script type="text/javascript" src="https://oapi.map.naver.com/openapi/v3/maps.js?ncpClientId=YOUR_CLIENT_ID&submodules=geocoder"></script>
</head>

<body>
<div id="title"><span class="title-decoration">또</span><span class="title-decoration">간</span><span class="title-decoration">집</span>맛집 지도</div>
<div id="container"><div id="map"></div><div id="side"><input type="text" id="input-addr" placeholder="주소를 입력해주세요."><input type="button" id="btn-coordinate" value="좌표 얻기"><div id="div-addr-console"></div></div></div>

<script>
    var mapOptions = { center: new naver.maps.LatLng(37.3595704, 127.105399), zoom: 10 };
    var map = new naver.maps.Map('map', mapOptions);

    document.addEventListener('click', (e) => {
        if(e.target.id == 'btn-coordinate') {
            const addr = document.getElementById('input-addr').value;

            if(!addr) { alert('주소를 입력해주세요.'); }
            else {
                $.ajax({
                    type: 'get', url: '/naver/map', data: {'addr': encodeURIComponent(addr)}, dataType: 'json',
                    success: (result) => { console.log('result = ', result); },
                    error: (x, e) => { console.log(x); console.log(e); }
                })
            }
        }
    });
</script>
</body>
</html>
```

아직 프론트 작업을 안해서 간단하게 만들었다.

css는 알아서 먹이자.

ajax도 알아서 살펴보자.

적용하면 아래와 같이 화면에 나타난다.

<br>

![image](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/26cb68c9-231f-4518-9a41-ea88d92b77fe)

## 5. 호출 및 결과
일단 여기까지 작성했을 때 로직은 다음과 같다.

1. input 폼에 주소를 입력한다.
2. `좌표 얻기` 버튼을 클릭한다.
3. intelliJ IDEA의 콘솔 창 및 html F12의 콘솔 창에 좌표를 확인한다.

위의 로직을 순서대로 실행했을 때 아래와 같이 정상적으로 api가 호출되는 것을 확인할 수 있다!

<br>

![image](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/18fc9d41-a285-47b7-ab7c-b55fc9f9d287)

```json
{
    "status": "OK",
    "meta": {
    "totalCount": 1,
    "page": 1,
    "count": 1
    },
    "addresses": [
    {
        "roadAddress": "대구광역시 중구 중앙대로 412 아카데미극장",
        "jibunAddress": "대구광역시 중구 남일동 65-1 아카데미극장",
        "englishAddress": "412, Jungang-daero, Jung-gu, Daegu, Republic of Korea",
        "addressElements": [
        {
        "types": [
        "SIDO"
        ],
        "longName": "대구광역시",
        "shortName": "대구광역시",
        "code": ""
        },
        {
        "types": [
        "SIGUGUN"
        ],
        "longName": "중구",
        "shortName": "중구",
        "code": ""
        },
        {
        "types": [
        "DONGMYUN"
        ],
        "longName": "남일동",
        "shortName": "남일동",
        "code": ""
        },
        {
        "types": [
        "RI"
        ],
        "longName": "",
        "shortName": "",
        "code": ""
        },
        {
        "types": [
        "ROAD_NAME"
        ],
        "longName": "중앙대로",
        "shortName": "중앙대로",
        "code": ""
        },
        {
        "types": [
        "BUILDING_NUMBER"
        ],
        "longName": "412",
        "shortName": "412",
        "code": ""
        },
        {
        "types": [
        "BUILDING_NAME"
        ],
        "longName": "아카데미극장",
        "shortName": "아카데미극장",
        "code": ""
        },
        {
        "types": [
        "LAND_NUMBER"
        ],
        "longName": "65-1",
        "shortName": "65-1",
        "code": ""
        },
        {
        "types": [
        "POSTAL_CODE"
        ],
        "longName": "41937",
        "shortName": "41937",
        "code": ""
        }
        ],
        "x": "128.5942424",
        "y": "35.8699358",
        "distance": 0
        }
    ],
    "errorMessage": ""
}
```

클라이언트에서 입력한 주소의 좌표 또한 정상적으로 출력된다.

---
**참고 레퍼런스**
- [네이버 주소 API로 주소 → 좌표 변환 + 도로명 주소 API](https://blog.naver.com/PostView.naver?blogId=platinasnow&logNo=220732491939)

**각주**

[^1]: [Naver Cloud Platform Maps](https://www.ncloud.com/product/applicationService/maps){: target="_blank"}

[^2]: [Naver Cloud Platform Geocoding](https://api.ncloud-docs.com/docs/ai-naver-mapsgeocoding){: target="_blank"}

[^3]: [Apache HttpComponents](https://hc.apache.org/){: target="_blank"}