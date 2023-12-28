---
layout: post
title: C# lock 에 대해
date: 2020-10-10 16:05 +0900
image: empty
categories: ["CSharp"]
tags: ["C#"]
---

## lock 내부 구현, 키워드

---

C# 프로그래밍을 하다보면 자연스레 멀티쓰레딩 환경에서 자원 획득 제한이 필요한 경우 lock을 사용했었다.

이 lock 키워드 및 블럭을 사용했을 때 내부적으로 어떻게 동작되는지 궁금해서 알아보았다.

### lock 문

[lock statement(lock 문) Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/lock-statement)

lock은 공식 문서에 `문`이라고 한다.

* Language Reference
  * Statement Keywords
    * lock Statement

위와 같은 구조로 Document에 구성되어 있다.

이 것의 동작 방식을 까보려면 역시 Decompile.

```csharp
private void DoAdd(int value)
{
    lock (_collection)
    {
        _collection.Add(value);
    }
}
```

위 코드를 Decompile시 아래의 il로 해석된다.

```il
.method private hidebysig instance void
    DoAdd(
      int32 'value'
    ) cil managed
  {
    .maxstack 2
    .locals init (
      [0] class [System.Collections]System.Collections.Generic.List`1<int32> V_0,
      [1] bool V_1
    )

    // [24 13 - 24 31]
    IL_0000: ldarg.0      // this
    IL_0001: ldfld        class [System.Collections]System.Collections.Generic.List`1<int32> Sample.LockSample::_collection
    IL_0006: stloc.0      // V_0
    IL_0007: ldc.i4.0
    IL_0008: stloc.1      // V_1
    .try
    {
      IL_0009: ldloc.0      // V_0
      IL_000a: ldloca.s     V_1
      IL_000c: call         void [System.Threading]System.Threading.Monitor::Enter(object, bool&)

      // [26 17 - 26 40]
      IL_0011: ldarg.0      // this
      IL_0012: ldfld        class [System.Collections]System.Collections.Generic.List`1<int32> Sample.LockSample::_collection
      IL_0017: ldarg.1      // 'value'
      IL_0018: callvirt     instance void class [System.Collections]System.Collections.Generic.List`1<int32>::Add(!0/*int32*/)

      // [27 13 - 27 14]
      IL_001d: leave.s      IL_0029
    } // end of .try
    finally
    {

      IL_001f: ldloc.1      // V_1
      IL_0020: brfalse.s    IL_0028
      IL_0022: ldloc.0      // V_0
      IL_0023: call         void [System.Threading]System.Threading.Monitor::Exit(object)

      IL_0028: endfinally
    } // end of finally

    // [28 9 - 28 10]
    IL_0029: ret

  } // end of method LockSample::DoAdd
```

결과를 토대로 lock 문 없이 재구성하면, 

```csharp
private void DoAdd2(int value)
{
    bool lockTaken = default;
    try
    {
        System.Threading.Monitor.Enter(_collection, ref lockTaken);
        _collection.Add(value);
    }
    finally
    {
        System.Threading.Monitor.Exit(_collection);
    }
}
```

위 코드와 거의 유사하다.

하지만 실제로 저 코드도 decompile 시 il이 완전히 동일하지는 않다.

il을 봤을 땐 lock 문 없는 아래쪽 코드가 더 효율적으로 보였다.  
왜 그런진 컴파일러의 동작까지 알아야 할텐데 그건 아직 내 역량 밖..

<details>
  
  <summary><b>il source</b></summary>
  
<pre>
  <code class="csharp">
    .method private hidebysig instance void
    DoAdd2(
      int32 'value'
    ) cil managed
  {
    .maxstack 2
    .locals init (
      [0] bool lockTaken
    )

    // [32 13 - 32 38]
    IL_0000: ldc.i4.0
    IL_0001: stloc.0      // lockTaken
    .try
    {

      // [35 17 - 35 76]
      IL_0002: ldarg.0      // this
      IL_0003: ldfld        class [System.Collections]System.Collections.Generic.List`1<int32> Sample.LockSample::_collection
      IL_0008: ldloca.s     lockTaken
      IL_000a: call         void [System.Threading]System.Threading.Monitor::Enter(object, bool&)

      // [36 17 - 36 40]
      IL_000f: ldarg.0      // this
      IL_0010: ldfld        class [System.Collections]System.Collections.Generic.List`1<int32> Sample.LockSample::_collection
      IL_0015: ldarg.1      // 'value'
      IL_0016: callvirt     instance void class [System.Collections]System.Collections.Generic.List`1<int32>::Add(!0/*int32*/)

      // [37 13 - 37 14]
      IL_001b: leave.s      IL_0029
    } // end of .try
    finally
    {

      // [40 17 - 40 60]
      IL_001d: ldarg.0      // this
      IL_001e: ldfld        class [System.Collections]System.Collections.Generic.List`1<int32> Sample.LockSample::_collection
      IL_0023: call         void [System.Threading]System.Threading.Monitor::Exit(object)

      // [41 13 - 41 14]
      IL_0028: endfinally
    } // end of finally

    // [42 9 - 42 10]
    IL_0029: ret

  } // end of method LockSample::DoAdd2
  </code>
</pre>
</details>