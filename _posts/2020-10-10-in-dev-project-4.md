---
layout: post
title: 개발중인 프로젝트 구성안 - 4
date: 2020-10-10 16:05 +0900
image: empty
category: Programming
---

## 프로젝트내 코드 생성 활용

---

근래엔 Code 생성을 적극적으로 활용하고 있다.

반복 작업 최소화, 실수 방지 측면에서 효율적이라 생각한다.

그 부분들에 대해 소개하려 한다.

---


## MessagePack Code Generation

MessagePack은 [AOT Code Generation](https://github.com/neuecc/MessagePack-CSharp#aot-code-generation-support-for-unityxamarin)을 지원한다.


* Unity에서 MessagePack을 사용하기 때문에
* AOT로 직접 생성된 Formatter를 사용한 결과가 더 좋기 때문에

AOT Code Generation을 사용한다.
(mobile platform에선 il emit을 허용하지 않음 [Unity documentation](https://docs.unity3d.com/Manual/ScriptingRestrictions.html))

측정 결과가 다른 것을 경험하고 server에도 AOT 생성된 Formatter를 사용하는데.. 정확한 결과 비교는 해봐야 알 듯.

기억에 남는건 기본 MessagePack 의 Serialize 결과와 AOT Generated Formatter를 사용했을 때 결과가 달랐다.
(AOT 쪽이 byte size가 더 작음)

---

## Core State Serializer Generation

인게임 아키텍쳐에서, **Serializable State**라는 개념을 구현하기 위해 코드 생성을 사용한다.  
Serializable State에 대해선 추후에 다뤄보려 한다.

##### DotDamageBuff 객체에 대한 Serialize / Deserialize를 제공하는 Serializer
```csharp
// this code is generated by CoreSerializerGenerator. Do not edit.
    public class DotDamageBuffSerializer : AbstractStateSerializer
    {
        public override System.Type TargetType => typeof(Dash.Core.Buff.DotDamageBuff);
        public override void Serialize(ref MessagePackWriter writer, ISerializableState raw, SerializationContext context)
        {
            var value = raw as Dash.Core.Buff.DotDamageBuff;
            MessagePackSerializer.Serialize(ref writer, value.Damage);
            MessagePackSerializer.Serialize(ref writer, value.CasterSerial);
            MessagePackSerializer.Serialize(ref writer, value.BuffId);
            MessagePackSerializer.Serialize(ref writer, value.Ended);
            MessagePackSerializer.Serialize(ref writer, value.DurationSeconds);
            MessagePackSerializer.Serialize(ref writer, value.ElapsedSeconds);
        }
        public override ISerializableState Deserialize(ref MessagePackReader reader, DeserializationContext context, out long readSize)
        {
            var start = reader.Consumed;
            var __1 = reader.ReadInt32();
            var __2 = reader.ReadInt32();
            var __3 = reader.ReadInt32();
            var __4 = reader.ReadBoolean();
            var __5 = reader.ReadSingle();
            var __6 = reader.ReadSingle();
            var result = new Dash.Core.Buff.DotDamageBuff(){
Damage = __1, CasterSerial = __2, BuffId = __3, Ended = __4, DurationSeconds = __5, ElapsedSeconds = __6};
            readSize = reader.Consumed - start;
            return result;
        }
    }
```

---

## Type Definition Generation

프로젝트 곳곳에서 추상 클래스, 인터페이스를 활용하고 있다.  
ex) 게임 데이터의 다형성, 인게임 객체의 다형성

원래는 Reflection을 사용해 **특정 타입**에 대한 **하위 타입**들의 List를 구현했었는데,  
오버헤드가 커서 아래와 같은 AOT 코드를 사용한다.


##### BuffInfo의 하위 타입들을 찾고 싶을 때

```csharp
var definition = PolyTypeDefinitionHolder.Dic[typeof(BuffInfo)];

foreach(Type derivedType in definition.DerivedTypes)
{
      ...
}
```

##### 코드 생성 결과
```csharp
public class PolyTypeDefinitions : IPolyTypeDefinitions
{
    public IReadOnlyDictionary<Type, PolyTypeDefinition> Dic => _dic;
    private static Dictionary<Type, PolyTypeDefinition> _dic = new Dictionary<Type, PolyTypeDefinition> {
        {
            typeof(Dash.StaticData.Entity.BuffInfo), new PolyTypeDefinition(typeof(Dash.StaticData.Entity.BuffInfo), new[]
                {
                    typeof(Dash.StaticData.Entity.NullBuffInfo),
                    typeof(Dash.StaticData.Entity.DotDamageBuffInfo),
                    typeof(Dash.StaticData.Entity.DamageBuffInfo),
                    typeof(Dash.StaticData.Entity.StatusBuffInfo),
                    typeof(Dash.StaticData.Entity.ShieldBuffInfo),
                    typeof(Dash.StaticData.Entity.HealBuffInfo),
                    typeof(Dash.StaticData.Entity.InvincibleBuffInfo),
                    typeof(Dash.StaticData.Entity.CCBuffInfo),
                    typeof(Dash.StaticData.Entity.CooldownSpeedBuffInfo),
                })
        },
        {
            typeof(Dash.StaticData.SimulationScript), new PolyTypeDefinition(typeof(Dash.StaticData.SimulationScript), new[]
                {
                    typeof(Dash.StaticData.WaitScript),
                    typeof(Dash.StaticData.SummonCharacterScript),
                    typeof(Dash.StaticData.SummonMonsterScript),
                    typeof(Dash.StaticData.GiveAbilityScript),
    ...

```

## EntityFramework DB First

RDB 의 Table에 매칭되는 Model class를 자동 생성하여 사용한다.

**Microsoft.EntityFrameworkCore**에서 기본 제공하는 **CSharpEntityTypeGenerator**를 확장해 커스텀된 Code를 생성한다.
```shell
dotnet ef dbcontext scaffold "dbConnectionString" Pomelo.EntityFrameworkCore.MySql -v -f
--project Dash --data-annotations --output-dir "path-to-entities" --context-dir "path-to-dbcontext"
```

##### 코드 생성 결과 예시 - RDB Model
```csharp
// This code is generated by EntityTypeGenerator. Dot not Edit!
[MessagePack.MessagePackObject()]
public partial class Equipment : Common.Model.IModel
{
    ...

    public Equipment() { }
    public Equipment(Equipment other)
    {
        OidAccount = other.OidAccount;
        Serial = other.Serial;
        Grade = other.Grade;
        Id = other.Id;
        Level = other.Level;
    }


    [MessagePack.Key(0)]
    [KeyColumn]
    public ulong OidAccount { get; set; }
    [MessagePack.Key(1)]
    [KeyColumn]
    public uint Serial { get; set; }
    [MessagePack.Key(2)]
    public int Id { get; set; }
    [MessagePack.Key(3)]
    public ushort Grade { get; set; }
    [MessagePack.Key(4)]
    public byte Level { get; set; }
}
```

RDB Model은 MessagePack (de)serialize 를 지원하도록 한다.

* client, server에서 각각 별도의 class를 사용하지 않고, 이것을 그대로 사용.
* client <-> server 통신시에도 사용.
* 중복 코드 최소화의 일환.

---

<br>

아직까지는 Code 생성에 대한 큰 단점을 느껴보지 못했다.  
계속해서 더 Code 생성을 도입할 만한 부분을 찾고 있다.  

잘 활용하면 정말 좋은 방법론이란 건 확실하다.
