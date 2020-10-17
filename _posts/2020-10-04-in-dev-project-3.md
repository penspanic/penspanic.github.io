---
layout: post
title: 개발중인 프로젝트 구성안 - 3
date: 2020-10-04 21:01 +0900
image: empty
category: Programming
---

## Library, Package 활용

---

세번째 포스트는 프로젝트에서 사용중인 Library와 Package들에 대해 소개하려 한다.

---

## MessagePack
[GitHub](https://github.com/neuecc/MessagePack-CSharp)

프로젝트 시작시 **Serialization** Library를 어떤 것을 사용할지 꽤 고민했었다.  

이전 프로젝트에선 client <-> server 통신시 **Protobuf**를 사용했었는데, 이게 꽤나 불편했었다. -_-  
IDL을 수정하고, Compile하고, Compile된 결과물을 각각 project에 복사하고.  
(자동화 스크립트가 있다고 하더라도 꽤나 귀찮았다.)

이번 프로젝트에선 모든 Language를 C#으로 통일했기에, IDL을 사용할 필요가 없어져서  
Protobuf은 후보 선상에서 제외.

또한 output에 meta-data 없이 payload 만 필요하기 때문에, 그 면에서도 MessagePack이 적합하다고 판단했다.

그 이외에 ZeroFormatter나 MessagePack 정도로 좁혀져서, 아직 활발히 개발중인
**MessagePack**으로 선택!

### 사용처
* client <-> server 통신, server <-> server 통신.  
    * TCP 통신, HTTP 통신 모두 사용중.
* redis value.   
* RDB 특정 칼럼의 값.  
(list 형식 or 다형성 데이터등. migration / 가시성의 중요성 낮을 때.)
* 게임 플레이 데이터 저장 Format.

### 장점
* **IDL - Compile**이 필요 없음(Protobuf와 달리).  

* MessagePack Codec은 표준이기 때문에 Redis Client등에서 MessagePack binary를 시각화할 수 있음.

* Size가 작음. 별도로 관련 최적화할 필요성이 낮다.

* (CSharp 버전 한정) Library Performance가 잘 나온다.  
v.2 에선 ```ref struct```, ```ValueTask<T>``` 등 최신 C# 기능을 사용해 더 최적화된 모습을 볼 수 있다.


### 단점

* 부분적 Deserialize를 할 수 없다.(meta-data 를 저장하지 않는 구조이기 때문에)  

<br>

별도의 Formatter(Serializer) 코드가 없어도 동작하는데, 이는 il code gen을 통해 동작한다.

하지만 이것은 Unity il2cpp 환경에선 동작하지 않기 때문에 il code gen을 사용하지 않고  
**AOT**로 Formatter를 생성할 수 있는 기능을 제공한다. 우리도 사용중.

---
## DotNetty
[GitHub](https://github.com/Azure/DotNetty)

비동기 이벤트 기반 네트워킹 어플리케이션 프레임워크.

![설명](/assets/img/3/netty-description.png)

*꽤나 복잡한 1줄 설명.*

사실 위의 내용은 유명한 **java 진영 라이브러리 Netty**의 것이고, DotNetty는 단순 Netty의 **.Net 포팅 버전**이다.

버퍼 풀링, 이벤트 메서드, 이벤트 루프 처리 등을 모두 도맡아서 해주기 때문에  
간단한 설정으로 꽤 정교하게 세팅된 Server를 구성할 수 있다.

하지만.. 현재 지원 및 개발이 **중단**된 상태이다..
커뮤니티에선 더이상 이것을 사용하지 말고, [다른 것을 사용하라고](https://github.com/Azure/DotNetty/issues/532) 한다.

전에 사용했었기에 익숙해 여건상 현재도 사용하고 있는데, 신규 프로젝트에선 사용하지 말아야 할 듯 하다.


### 사용처
client, server에서 TCP 소켓을 사용한 네트워킹시 사용한다.

정식 유니티 지원이 되지 않지만, Dll과 Dependency Dll들을 추출하여  
Unity Project에 넣어 사용중이다.

### 장점

* 꽤나 정교하게 Configuration을 할 수 있고, 그를 통해 High performance를 낼 수 있음.  
ex) BufferSize, WaterMark, EventLoop Thread count, Buffer Allocator

### 단점

* 지원, 개발이 중단됨.
* Reference 부족(Netty의 [wiki](https://netty.io/wiki/)를 참조해야 함).

---

## ETC

### AWS SDK

AWS의 여러 서비스를 사용하기 때문에, 당연히 SDK도 사용해 활용한다.

* AWSSDK.CloudWatch
* AWSSDK.DynamoDBv2
* AWSSDK.S3
* Amazon.Lambda.Core

...

### Unity Project

너무 많아서.. 쪼금 특이한 것들만 추리자면
##### Mathematics
* Unity DOTS에서 사용하는 수학 연산 라이브러리이지만, 일반적인 용도로 쓰기에도 나쁘지 않아서 사용중.

##### Test Framework + Performance testing API
* client-side에서도 test code를 작성하고 있다. 성능 측정 / 프로파일링 용도로도 가끔씩 사용중이다.
* 공식 UI test framework는 아직 없는 것 같다. 테스트 자동화를 위해선 필요할텐데 곧 나오겠지?

#### NLog
이름대로 Logging 라이브러리. 단순한 용도로만 사용해서 아직 불편함은 느끼지 못했다.

#### NewtonSoft.Json

널리 사용되는 Json 라이브러리다.  
Unity 공식 Package 에서도 내부적으로 사용하는 듯 하다.

```json
  "dependencies": {
    "com.unity.test-framework": "1.1.0",
    "com.unity.nuget.newtonsoft-json": "2.0.0-preview"
  }
```

*com.unity.test-framework.performance@2.3.1-preview 의 Dependency.*

클라이언트와 서버가 같은 라이브러리를 사용하기 위해 .Net Core의 System.Text.Json 대신 이것을 도입했다.

현재 구버전 사용중인데, 신버전의 Performance를 확인해보고 다른 것을 쓸지 고려해봐야 할 것 같다.

Unity 환경이 아닌 .Net Core 환경에선 System.Text.Json이 더 최신식으로 보인다. [Microsoft Docs](https://docs.microsoft.com/ko-kr/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to)