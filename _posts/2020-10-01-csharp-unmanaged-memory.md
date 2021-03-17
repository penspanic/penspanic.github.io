---
layout: post
title: csharp-unmanaged memory
date: 2020-10-01 10:32 +0900
image: empty
category: CSharp_Dotnet
---

## Managed / Unmanaged Memory

---

### Managed Memory

CLR(Common Language Runtime)이 관리하는 메모리.  Garbage Collection의 대상이 된다.

### Unmanaged Memory

Managed Memory가 아닌 것.

C# 에서도 C++ 에서처럼 직접 메모리를 할당 할 수 있다.

```csharp
IntPtr memory = Marshal.AllocHGlobal(100);
for (int i = 0; i < 100; ++i)
{
    ((byte*) memory)[i] = (byte)(i + 1);
}
Marshal.FreeHGlobal(memory);
```

---

## stackalloc 표현식

Stack 에 메모리가 할당됨. 그 Stack을 벗어날 때 자동으로 메모리 해제.

일시적인 계산을 위한 메모리를 사용할 때, stackalloc을 사용함으로써
Managed Heap에 할당되지 않게 하여 최적화 가능.

```csharp
public void RemoveAll() // Dash.Core.Systems.GeneratorSystem
{
    Span<int> serials = stackalloc int[_generators.Count];

    int index = 0;
    foreach (int serial in _generators.Keys)
    {
        serials[index++] = serial;
    }

    foreach (int serial in serials)
    {
        _generators.Remove(serial);
    }
}

```

---
## 사견
C#을 막연하게 C, C++ 보다 느리다고 생각하는 사람들이 많다.  
심지어 C#을 메인 스킬로 다루는 사람들까지도.

어떤 부분에서 퍼포먼스가 떨어지는 것인지, 그걸 해결하기 위한 방법이 있는지에 대해  
자세히 알아야 할 필요가 있다고 생각한다.

<br>
N년간 C# / .Net에 관심을 가저오며 지금은  
**C#으로도 *High Performance Program*을 만들 수 있다, 하지만 쉽지 않을 것이다**  
라는 결론에 도달했다.

그렇게 생각하게 된 이유를 공유하고 싶고,  
.Net 및 Managed Langauge에 대한 편견을 없애보고 싶다.
