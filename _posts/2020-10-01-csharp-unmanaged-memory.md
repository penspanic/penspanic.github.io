---
layout: post
title: csharp-unmanaged memory
date: 2020-10-01 10:32 +0900
image: "empty"
category: CSharp_Dotnet
---

## Managed / Unmanaged Memory

---

### Managed Memory

CLR(Command Language Runtime)이 관리하는 메모리.  Garbage Collection의 대상이 된다.

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
		var classArray = new FileReader[5];
		classArray[0] = new FileReader();
		
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

public ref readonly struct Span<T> { }

```