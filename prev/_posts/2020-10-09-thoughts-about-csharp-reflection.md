---
layout: post
title: C# - Reflection에 대한 고찰
date: 2020-10-09 12:02 +0900
image: empty
category: CSharp_Dotnet
---

## Reflection에 대한 고찰과 약간의 팁

---

C# 프로그래밍 씬에서 가장 흔한 오해중 하나는  

**"Reflection은 느리니까 자제해야 한다."**

라는 것.

---

처음엔 그냥 그런가보다 했다. C#에 대한 이해도가 많이 부족했으니.

하지만 필요에 의해 Reflection을 사용하게 됐고, 그것 없이 C# 프로그래밍을 하기에 불편함에 이르렀다.

제대로 이해하고 적절히 사용해야 할 때가 온 것이다.

<br>

깊게 파보고, 프로파일링 - 최적화 해보고. 그 과정들을 통해 지금은

**올바른 Reflection 사용은 현대적이고 발전된 코드를 작성할 수 있다.**

라고 생각하게 됐다.

---

### 느리다는 것

* 처리 시간이 오래 걸린다.
* 할당이 잦다 / 메모리 사용량이 많다.  
  (.Net 에선 GCHeap의 상태가 프로세스 전반에 영향을 주니)

내가 이해하고 있는 느리다는 것의 의미는 이정도이다.

그럼 위 두가지를 개선할 수 있다면?

실 예시를 들어서 보자.

### 최적화 예시

#### 1. 캐싱

##### 1.1. Assembly 내의 특정 타입의 하위 타입들을 찾을 때

```csharp
public void ForEachThatInherits(Type type, Action<Type> action)
{
  var derivedTypes = targetType.Assembly.GetTypes().Where(targetType.IsAssignableFrom).ToArray();
  foreach(Type eachType in derivedTypes)
  {
    action?.Invoke(eachType);
  }
}
```

위의 코드는 실로 엄청난 오버헤드를 가지고 있다.. 경험담.

하지만 저 표현식의 결과는 (대부분의 상황에서) 불변이다. 그렇다면 **Caching 가능!**

아래는 Result Caching 를 통한 최적화된 코드.

```csharp

public void ForEachThatInherits(Type type, Action<Type> action)
{
  var derivedTypes = DerivedTypeCache.GetDerivedTypes(type, type.Assembly);
  foreach(Type eachType in derivedTypes)
  {
    action?.Invoke(eachType);
  }
}

```

<details>
  <summary><b>필자가 작성해 사용중인 DerivedTypeCache.cs</b></summary>

  <pre>
  // no interface, abstract class included.
  public static class DerivedTypeCache
  {
      public struct DeriveTypeContext
      {
          public Assembly[] Assemblies;
          public Type[] DerivedTypes;
      }

      private static object _lockObject = new object();
      private static Dictionary&lt;(Type, Assembly[]), DeriveTypeContext&gt; _contexts = new Dictionary&lt;(Type, Assembly[]), DeriveTypeContext&gt;();
      public static ReadOnlyCollection&lt;Type&gt; GetDerivedTypes(Type type, params Assembly[] assemblies)
      {
          lock (_lockObject)
          {
              if (_contexts.TryGetValue((type, assemblies), out DeriveTypeContext context) == true)
              {
                  return new ReadOnlyCollection&lt;Type&gt;(context.DerivedTypes);
              }

              InternalInit(type, assemblies);

              return new ReadOnlyCollection&lt;Type&gt;(_contexts[(type, assemblies)].DerivedTypes);
          }
      }

      private static void InternalInit(Type type, Assembly[] assemblies)
      {
          DeriveTypeContext context = new DeriveTypeContext();
          if (assemblies != null &amp;&amp; assemblies.Length &gt; 0)
          {
              List&lt;Type&gt; temp = new List&lt;Type&gt;();
              foreach (Assembly assembly in assemblies)
              {
                  temp.AddRange(assembly.GetTypes().Where(t =&gt;
                      type.IsAssignableFrom(t) &amp;&amp; t != type &amp;&amp; t.IsInterface == false &amp;&amp; t.IsAbstract == false)
                      .OrderBy(t =&gt; t.Name)
                  );
              }

              context.Assemblies = assemblies;
              context.DerivedTypes = temp.ToArray();
          }
          else
          {
              context.DerivedTypes = type.Assembly.GetTypes().Where(t =&gt; type.IsAssignableFrom(t) &amp;&amp; t != type &amp;&amp; t.IsInterface == false &amp;&amp; t.IsAbstract == false)
                  .OrderBy(t =&gt; t.Name).ToArray();
          }

          _contexts.Add((type, assemblies), context);
      }
  }

  </pre>

</details>

<br>

##### 1.1. 특정 조건에 맞는 Method들을 찾을 때

위 예제와 동일한 방식으로 Result Caching 할 수 있다.


```csharp

Type[] targetTypes = new[] {typeof(int), typeof(System.IO.File), typeof(System.Diagnostics.Activity)};

for (int i = 0; i < 10; ++i)
  NoCaching(targetTypes);

for (int i = 0; i < 10; ++i)
  Caching(targetTypes);

...

public void NoCaching(Type[] types)
{
    foreach (Type type in targetTypes)
        var result = type.GetMethods(BindingFlags.Public | BindingFlags.Instance);
}
public void Caching(Type[] types)
{
    foreach (Type type in targetTypes)
        var result = TypeInfoHolderHelper.GetMethods(type, (m) => m.IsPublic && m.IsStatic == false);
}
```

|    Method |     N |     Mean |    Error |   StdDev |
|---------- |------ |---------:|---------:|---------:|
| NoCaching | 10000 | 77.33 ms | 1.128 ms | 1.055 ms |
|   Caching | 10000 | 32.40 ms | 0.447 ms | 0.396 ms |

**직접 측정한 결과.** [실 코드](https://github.com/penspanic/CSharpReferences/blob/master/Benchmark/GetMethodsBenchmark.cs)

---

#### 2. 기능별 심화 사용

##### 2.1. MethodInfo Invoke 자제

MethodInfo.Invoke()는 Delegate를 생성하여 호출하는 것보다 훨씬 느리다.

[참조 : Method 호출 퍼포먼스 테스트](https://jeremybytes.blogspot.com/2014/01/improving-reflection-performance-with.html)

##### 2.2. Constructor

가장 유용한 Reflection 기능중 하나는 Type 을 가지고 해당 인스턴스를 만드는 것이다.
하지만 보편적인 Activator.CreateInstance()는 매우 느리다.
그럴 땐 Expression 을 사용해 더 빠르게 인스턴스를 생성할 수 있다.


|                    Method |     N |       Mean |     Error |    StdDev |
|-------------------------- |------ |-----------:|----------:|----------:|
|            CreateInstance | 10000 |   2.944 us | 0.0396 us | 0.0370 us |
|  CreateInstance_Activator | 10000 | 467.544 us | 9.1044 us | 8.5163 us |
| CreateInstance_Expression | 10000 | 142.403 us | 2.7698 us | 2.3129 us |

**직접 측정한 결과.** [실 코드](https://github.com/penspanic/CSharpReferences/blob/master/Benchmark/ConstructorBenchmark.cs)

##### 2.3. Boxing / Unboxing 회피등 .Net 레벨 최적화

Reflection 관련 없지만, .Net 레벨 최적화를 통해 엄청난 성능 개선을 이뤄낼 수 있다.

---

적다보니 Reflection 최적화 팁처럼 돼버렸다..

<br>

숙련된 C# 프로그래머들은 Reflection에 대한 오해를 하지 않을 거라 생각한다.  

본인이 고민했던 부분을 공유하고 싶어서 글을 작성했다.

---
**.Net 환경별로 결과가 상이할 수 있다.**
