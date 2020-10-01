---
layout: post
title: 개발중인 프로젝트 구성안 - 2
date: 2020-10-01 14:15 +0900
image: empty
category: Programming
---

## 현재 개발중인 프로젝트의 구성에 대해 - 2

---


먼저, Repository에 대해.  

---
# Repository
## client-dash
**유니티 클라이언트 프로젝트**  
**Unity 2019.4x, .Net 4.x**
 * data-dash, lib-dash

## server-dash
**로비서버, 매치서버, 인게임 서버등 서비스에 필요한 서버들의 모음**  
**.Net Core 3.x**
 * data-dash, lib-dash

## admin-dash
**관리자툴, 유저 및 게임 서비스에 대한 제어**  
**.Net Core 3.x**
 * data-dash, lib-dash

## dummyclient-dash
**더미 클라이언트. 인게임 전투, API 요청, 특수한 시나리오등을 수행하는 클라이언트**  
**.Net Core 3.x**
 * data-dash, lib-dash

## etc
**AWS Lambda 에서 실행되는 MicroService 등**
<br>

눈치 채신 분들도 있겠지만, **data-dash**, **lib-dash**는 **Submodule** 이다.  
모든 Root Project에서 data-dash, lib-dash 두가지의 submodule을 공유하며   
필요한 데이터와 소스를 공유한다.  

---
이전에 진행했던 프로젝트에서는 서버, 클라이언트의 모듈 공유를 dll형태로 했었다.  
그래서, 공용 모듈을 클라이언트에 반영하려면  
1. 공용 모듈 소스 수정
2. 공용 모듈 빌드, 복사
3. 클라이언트 실행

이 과정을 거쳐야 했고, 개발 과정에서 실수로 갱신되지 않은 모듈을 사용하는 등의
실수가 잦았다. 저 과정 자체도 꽤 오래 걸렸지만.

그래서 이번 프로젝트에선 Submodule 형태로 클라이언트와 서버가 소스 자체를
통으로 공유하는 방식을 선택했다. 거기에 Submodule Sharing을 통해, 공용 모듈
Repository에 대해 최신화 / 버전 맞추기등을 용이하게 했다.

*  공용 모듈 빌드 -> 적용된 클라이언트 재빌드 과정을  
   공용 모듈 수정 -> 클라이언트 재빌드  
   빌드 2번 -> 1번으로 축소됨.

---
## 전처리기 사용
전처리기 자체가 썩 아름다운 방법이라고 생각하진 않지만,  
효과적으로 소스 자체를 공개시키지 않는데엔 전처리기만한 방법을 아직 못 찾았다.  
그래서 이 프로젝트에선 클라이언트 | 서버 소스 구분이 필요할 경우 전처리기를 사용한다.
```csharp
void SomeMethod()
{
    #if Common_Unity
    // 클라 전용
    #endif

    #if Common_Server
    // 서버 전용
    #endif
}
```
---
## 좀더 생각해볼 점
C# 소스 자체를 통으로 공유하게 되는데, client(unity)와 server(.net core) 의 
Framework가 달라서  
생기는 사소한 문제들이 있다.

***Framework 가 다르기에 Library 버전이 다름***
```csharp
// 아래 코드는 unity client에선 호환되지 않음!
string.Join(',', System.Linq.Enumerable.Range(0, 10));

...
// Microsoft.NETCore.App/3.1.6/System.Private.CoreLib.dll
public static unsafe string Join<T>(char separator, IEnumerable<T> values) => string.JoinCore<T>(&separator, 1, values);

// /mono/4.5/mscorlib.dll
public static unsafe string Join<T>(string separator, IEnumerable<T> values)
```