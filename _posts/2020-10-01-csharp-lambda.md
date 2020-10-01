---
layout: post
title: CSharp-Lambda
date: 2020-10-01 00:13 +0900
image: "empty"
category: CSharp_Dotnet
---

# C# 람다의 내부 구현에 대해  


마법은 없다. Lambda를 컴파일러가 어떤식으로 구현하는지 알아둘 필요가 있다.
<br/><br/>

```csharp
public static class LambdaExample
{
    public static void Run()
    {
        int localInt = 123456789;
        string localString = "LocalString";
        ActionRunner(() =>
        {
            Console.WriteLine($"{localInt}, {localString}");
        });
    }

    private static void ActionRunner(Action action)
    {
        action?.Invoke();
    }
}
```

<details>
  
  <summary><b>Decompiled il source</b></summary>
  
<pre>
  <code class="csharp">
    // Type: CSharpExamples.LambdaExample 
    // Assembly: CSharpExamples, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
    // MVID: B420F7F8-9269-4798-8719-2A8ECDC7FBCA
    // Location: /Users/geunheepark/Projects/private/CSharpExamples/CSharpExamples/bin/Debug/netcoreapp3.1/CSharpExamples.dll
    // Sequence point data from /Users/geunheepark/Projects/private/CSharpExamples/CSharpExamples/bin/Debug/netcoreapp3.1/CSharpExamples.pdb

    .class public abstract sealed auto ansi beforefieldinit
      CSharpExamples.LambdaExample
        extends [System.Runtime]System.Object
    {

      .class nested private sealed auto ansi beforefieldinit
        '<>c__DisplayClass0_0'
          extends [System.Runtime]System.Object
      {
        .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
          = (01 00 00 00 )

        .field public int32 localInt

        .field public string localString

        .method public hidebysig specialname rtspecialname instance void
          .ctor() cil managed
        {
          .maxstack 8

          IL_0000: ldarg.0      // this
          IL_0001: call         instance void [System.Runtime]System.Object::.ctor()
          IL_0006: nop
          IL_0007: ret

        } // end of method '<>c__DisplayClass0_0'::.ctor

        .method assembly hidebysig instance void
          'Runb__0'() cil managed
        {
          .maxstack 8

          // [12 13 - 12 14]
          IL_0000: nop

          // [13 17 - 13 65]
          IL_0001: ldstr        "{0}, {1}"
          IL_0006: ldarg.0      // this
          IL_0007: ldfld        int32 CSharpExamples.LambdaExample/'<>c__DisplayClass0_0'::localInt
          IL_000c: box          [System.Runtime]System.Int32
          IL_0011: ldarg.0      // this
          IL_0012: ldfld        string CSharpExamples.LambdaExample/'<>c__DisplayClass0_0'::localString
          IL_0017: call         string [System.Runtime]System.String::Format(string, object, object)
          IL_001c: call         void [System.Console]System.Console::WriteLine(string)
          IL_0021: nop

          // [14 13 - 14 14]
          IL_0022: ret

        } // end of method '<>c__DisplayClass0_0'::'Runb__0'
      } // end of class '<>c__DisplayClass0_0'

      .method public hidebysig static void
        Run() cil managed
      {
        .maxstack 2
        .locals init (
          [0] class CSharpExamples.LambdaExample/'<>c__DisplayClass0_0' 'CS$<>8__locals0'
        )

        IL_0000: newobj       instance void CSharpExamples.LambdaExample/'<>c__DisplayClass0_0'::.ctor()
        IL_0005: stloc.0      // 'CS$<>8__locals0'

        // [8 9 - 8 10]
        IL_0006: nop

        // [9 13 - 9 38]
        IL_0007: ldloc.0      // 'CS$<>8__locals0'
        IL_0008: ldc.i4       123456789 // 0x075bcd15
        IL_000d: stfld        int32 CSharpExamples.LambdaExample/'<>c__DisplayClass0_0'::localInt

        // [10 13 - 10 48]
        IL_0012: ldloc.0      // 'CS$<>8__locals0'
        IL_0013: ldstr        "LocalString"
        IL_0018: stfld        string CSharpExamples.LambdaExample/'<>c__DisplayClass0_0'::localString

        // [11 13 - 14 16]
        IL_001d: ldloc.0      // 'CS$<>8__locals0'
        IL_001e: ldftn        instance void CSharpExamples.LambdaExample/'<>c__DisplayClass0_0'::'Runb__0'()
        IL_0024: newobj       instance void [System.Runtime]System.Action::.ctor(object, native int)
        IL_0029: call         void CSharpExamples.LambdaExample::ActionRunner(class [System.Runtime]System.Action)
        IL_002e: nop

        // [15 9 - 15 10]
        IL_002f: ret

      } // end of method LambdaExample::Run

      .method private hidebysig static void
        ActionRunner(
          class [System.Runtime]System.Action action
        ) cil managed
      {
        .maxstack 8

        // [18 9 - 18 10]
        IL_0000: nop

        // [19 13 - 19 30]
        IL_0001: ldarg.0      // action
        IL_0002: brtrue.s     IL_0006
        IL_0004: br.s         IL_000d
        IL_0006: ldarg.0      // action
        IL_0007: callvirt     instance void [System.Runtime]System.Action::Invoke()
        IL_000c: nop

        // [20 9 - 20 10]
        IL_000d: ret

      } // end of method LambdaExample::ActionRunner
    } // end of class CSharpExamples.LambdaExample
  </code>
</pre>
</details>

<br>

위의 코드는 아래와 같이 해석될 수 있다.  
```csharp
public static class LambdaExampleTranslated
{
    public class LambdaClass
    {
        public int localInt;
        public string localString;

        public void Run()
        {
            Console.WriteLine($"{localInt}, {localString}");
        }
    }

    public static void Run()
    {
        int localInt = 123456789;
        string localString = "LocalString";
        ActionRunner(new LambdaClass
		{
			localInt = localInt,
			localString = localString
		}
		.Run);
    }

    private static void ActionRunner(Action action)
    {
        action?.Invoke();
    }
}
```

---
<br/><br/>

Lambda의 변수 Capture를 위해 Lambda 객체는 Capture할 변수들을 <b>멤버</b>로 지니게 된다.

→ Capture할 변수가 많으면 많을수록, Lambda 객체 자체의 Size도 커진다.

매 Lambda 식 전달마다, Lambda 클래스 객체가 생성되고 사용후 버려지기 때문에 GC Heap에 부담을 준다.

→ 빈번한 GC 발생은 Program performance 에 악영향을 끼침.

그러므로 performance- critical 한 Code에서는 Lambda 를 대체하여 최적화를 할 필요가 있다.

```csharp
// Lambda 객체 10000회 생성!
for (int i = 0; i < 10000; ++i)
{
    ActionRunner(() =>
    {
        Console.WriteLine($"{localInt}, {localString}");
    });
}

// Lambda 객체 1회 생성!
Action action = () =>
{
    Console.WriteLine($"{localInt}, {localString}");
};
for (int i = 0; i < 10000; ++i)
{
    ActionRunner(action);
}
```

---
<br/><br/>
## Method → Delegate 변환 Overhead

Method를 Delegate로 전달시 Action<T> 등 새로운 Delegate 객체가 생성되고, 그것이 전달된다.

→ 생성 과정에서 GC Alloc, 사용 이후엔 Garbage가 되는 Overhead가 있기 때문에 조심해야 한다.

```csharp
public class DelegateConversion
{
    public void Run()
    {
        // Action 10000회 생성.
        for (int i = 0; i < 10000; ++i)
            ActionRunner(MemberMethod);

        // Action 1회 생성.
        Action action = MemberMethod;
        for (int i = 0; i < 10000; ++i)
            ActionRunner(action);
    }

    private static void ActionRunner(Action action)
    {
        action?.Invoke();
    }

    private void MemberMethod() { }
}
```

❇️ Dash/Net/SendQueue.cs 에선 Delegate 캐싱 최적화를 통해 Overhead를 줄였음.

```csharp
public class SendQueue
{
	private readonly IEventLoop _eventLoop;
	// 미리 Delegate를 캐싱하여 GC Alloc 회피.
	private readonly Action _eventLoopAction = null;
	...
	public SendQueue()
	{
		_eventLoopAction = OnEventLoop;
	}

	public void Send()
	{
		...
		_eventLoop.Execute(_eventLoopAction);
		...
	}
	public void OnEventLoop() { ... }
}
```

---
<br/>

❗컴파일러 /  CLR 버전 / 실행 환경에 따라 최적화등으로 인해 결과가 상이할 수 있다.