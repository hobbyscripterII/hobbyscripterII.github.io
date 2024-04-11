---
title: "naver geocoding 반환 값(json) → VO로 파싱 후 클라이언트 화면에 출력하기"
date: 2024-04-10 11:30:00 +09:00
writer: 이주영
categories: api naver
tags: [java, spring boot, json, VO, naver, maps, geocoding, api]
toc: true
toc_sticky: true
---
이전 포스팅 [naver geocoding을 활용한 주소 및 좌표 검색 API 호출하기](https://hobbyscripterii.github.io/posts/naver-geocoding%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%A3%BC%EC%86%8C-%EB%B0%8F-%EC%A2%8C%ED%91%9C-%EA%B2%80%EC%83%89-API-%ED%98%B8%EC%B6%9C%ED%95%98%EA%B8%B0/){: target="_blank"}에서는 input 폼에 주소 입력 후 '좌표 얻기' 버튼을 클릭했을 때 ajax를 통해 geocoding api를 호출하는 url로 이동하여 해당 주소의 도로명 주소와 좌표 등의 상세 정보를 얻어내었다.

이후 다시 ajax의 반환 값으로 json의 모든 정보들을 클라이언트에 보냈으나 필요한 데이터만 보내고싶어서 json 데이터를 VO에 담는 과정을 거쳤으며 해당 과정에 대한 포스팅이다.

## 1. VO 작성
VO를 작성하기 전에 json 데이터가 어떻게 넘어오는지 확인해보자.

일단 내가 갖고싶은 데이터는 도로명 주소, 지번 주소, 좌표다.

아래의 데이터를 통해 내가 빼올 데이터를 확인해보자.

> 도로명 주소 - roadAddress <br>
> 지번 주소 - jibunAddress <br>
> 좌표 x축 - x <br>
> 좌표 y축 - y

<br>

```json
  "status": "OK",
  "meta": {
    "totalCount": 1,
    "page": 1,
    "count": 1
  },
  "addresses": [
    {
      "roadAddress": "대구광역시 중구 중앙대로 412 아카데미극장", // 도로명 주소
      "jibunAddress": "대구광역시 중구 남일동 65-1 아카데미극장", // 지번 주소
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
      "x": "128.5942424", // 좌표 x축
      "y": "35.8699358", // 좌표 y축
      "distance": 0.0
    }
  ],
  "errorMessage": ""
}
```

json 타입의 데이터를 VO로 담을 때 주의할 점은 데이터 타입이 다르면 파싱 과정에서 에러가 발생하기 때문에 어떤 형식으로 구성되어있는지만 확인하면 좋을 것 같다.

<br>

내가 빼올 데이터를 VO로 작성하면 아래와 같다.

```java
@Data
@JsonIgnoreProperties(ignoreUnknown = true) // json에 없는 프로퍼티 무시
public class NaverCoordinateGetVo {
    private String status;
    private List<Addresses> addresses;
    private String errorMessage;

    @Data
    @JsonIgnoreProperties(ignoreUnknown = true)
    public static class Addresses {
        private String roadAddress;
        private String jibunAddress;
        private String x;
        private String y;
    }
}
```

NaverCoordinateGetVo는 json 데이터를 담을 클래스이며 Addresses 클래스는 json에서 도로명 주소, 지번 주소, 좌표 값을 담고있는 배열 형태의 데이터이다. 따라서 해당 클래스를 list 형태로 담아 VO에 넣어뒀다. 멤버변수명은 json에 있는 프로퍼티명과 같아야하기 때문에 동일하게 적용시켰다.

> **@JsonIgnoreProperties(ignoreUnknown = true)** - json 데이터를 java 객체에 파싱할 때 없는 프로퍼티는 무시한다.(에러가 발생하지 않는다)

## 2. controller 수정
StringBuilder를 닫기 전 json을 VO로 파싱하는 소스코드를 추가했다.

```java
@Slf4j
@Controller
public class NaverApiController {
    // application.yaml에 있는 네이버 지도 api 애플리케이션 id와 secret key
    @Value("${api.naver.map.client-id}") private String clientId;
    @Value("${api.naver.map.client-secret}") private String clientSecret;

    @GetMapping("/naver/map")
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
        httpGet.addHeader("X-NCP-APIGW-API-KEY-ID", clientId);
        httpGet.addHeader("X-NCP-APIGW-API-KEY", clientSecret);

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
            naverCoordinateGetVo = objectMapper.readValue(stringBuilder.toString(), NaverCoordinateGetVo.class);

            bufferedReader.close(); // bufferedReader에 있는 문자열을 다 읽으면 닫아준다.
        } catch (Exception e) {
            e.printStackTrace();
        }
        return naverCoordinateGetVo;
    }
}
```

<br>

이전 포스팅에 있던 소스코드와 다른 점은 아래 부분이 추가되었으며 반환 타입이 수정되었다.

```java
// json to VO 작업
ObjectMapper objectMapper = new ObjectMapper(); // json을 java 객체로 변환하기 위해 ObjectMapper 객체 생성
naverCoordinateGetVo = objectMapper.readValue(stringBuilder.toString(), NaverCoordinateGetVo.class);
```

json 데이터를 java 객체로 변환하기 위해 ObjectMapper 객체를 생성했으며 담을 VO에 ObjectMapper 참조변수를 통해 파싱 과정을 거쳤다.

ObjectMapper에서 제공하는 readValue는 json 데이터를 java 객체로 역직렬화해주는 기능을 갖고있다.

첫번째 인자에는 파싱할 json 데이터를 두번째 인자에는 담을 vo의 클래스를 명시해준다.

## 3. html 수정
html 코드도 살짝 수정했다.

```js
$.ajax({
    type: 'get', url: '/naver/map', data: {'addr': encodeURIComponent(addr)}, dataType: 'json',
    success: (data) => {
        console.log('data = ', data);
        const addr = data.addresses[0];
        const divAddrConsole = document.getElementById('div-addr-console');
        divAddrConsole.innerHTML = `<p>도로명 주소: <span id="addr">${addr.roadAddress}</span></p><p>지번 주소: <span id="addr">${addr.jibunAddress}</span></p><p>x: <span id="x">${addr.x}</span></p><p>y: <span id="y">${addr.y}</span></p>`;
    },
    error: (x, e) => { console.log(x); console.log(e); }
})
```

javascript에서 ajax를 호출하는 부분인데 나는 ajax의 반환 값으로 vo에 담긴 주소 정보와 좌표 값을 화면에 출력하고 싶었다.

따라서 div-addr-console이라는 div에 innerHTML을 사용하여 `좌표 얻기` 버튼을 클릭할 경우 주소 정보부터 좌표 값까지 찍게 만들었다.

코드에서 배열의 0번째 인덱스만 가져오는 것은 일단 여러 주소를 입력해봤을 때 Addresses에 담긴 배열의 길이가 항상 1이길래 저렇게 만들어놓긴 했는데 추후에 수정해야 할지도 모르겠다.

## 4. 호출 및 결과
이후 `대구광역시 중구 남일동 65-1`라는 지번 주소를 input 폼에 입력하고 `좌표 얻기` 버튼을 클릭했을 때 출력되는 화면은 아래와 같다.

![image](https://github.com/hobbyscripterII/repeat-restaurant/assets/135996109/99768682-7794-41a8-8385-ff3be02f6db7)

정상적으로 출력되는 것을 확인할 수 있다!

