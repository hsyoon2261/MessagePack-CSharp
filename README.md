# MessagePack for C# (.NET, .NET Core, Unity, Xamarin)

[![NuGet](https://img.shields.io/nuget/v/MessagePack.svg)](https://www.nuget.org/packages/messagepack)
[![NuGet](https://img.shields.io/nuget/vpre/MessagePack.svg)](https://www.nuget.org/packages/messagepack)
[![Releases](https://img.shields.io/github/release/neuecc/MessagePack-CSharp.svg)][Releases]

[![Join the chat at https://gitter.im/MessagePack-CSharp/Lobby](https://badges.gitter.im/MessagePack-CSharp/Lobby.svg)](https://gitter.im/MessagePack-CSharp/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Build Status](https://dev.azure.com/ils0086/MessagePack-CSharp/_apis/build/status/MessagePack-CSharp-CI?branchName=master)](https://dev.azure.com/ils0086/MessagePack-CSharp/_build/latest?definitionId=2&branchName=master)

The extremely fast [MessagePack](http://msgpack.org/) serializer for C#.
It is 10x faster than [MsgPack-Cli](https://github.com/msgpack/msgpack-cli) and outperforms other C# serializers. MessagePack for C# also ships with built-in support for LZ4 compression - an extremely fast compression algorithm. Performance is important, particularly in applications like games, distributed computing, microservices, or data caches.

![Perf comparison graph](https://cloud.githubusercontent.com/assets/46207/23835716/89c8ab08-07af-11e7-9183-9e9415bdc87f.png)

MessagePack has a compact binary size and a full set of general purpose expressive data types. Please have a look at the [comparison with JSON, protobuf, ZeroFormatter section](#comparison) and learn [why MessagePack C# is the fastest](#performance).

## Table of Contents

- [Installation](#installation)
    - [NuGet packages](#nuget-packages)
    - [Unity](#unity)
    - [Migration notes from v1.x](#migration-notes-from-v1x)
- [Quick Start](#quick-start)
- [Analyzer](#analyzer)
- [Built-in supported types](#built-in-supported-types)
- [Object Serialization](#object-serialization)
- [DataContract compatibility](#datacontract-compatibility)
- [Serializing readonly/immutable object members  (SerializationConstructor)](#serializing-readonlyimmutable-object-members--serializationconstructor)
- [Serialization Callback](#serialization-callback)
- [Union](#union)
- [Dynamic (Untyped) Deserialization](#dynamic-untyped-deserialization)
- [Object Type Serialization](#object-type-serialization)
- [Typeless](#typeless)
- [Security](#security)
- [Performance](#performance)
    - [Deserialization Performance for different options](#deserialization-performance-for-different-options)
- [LZ4 Compression](#lz4-compression)
    - [Attributions](#attributions)
- [Comparison with protobuf, JSON, ZeroFormatter](#comparison-with-protobuf-json-zeroformatter)
- [Hints to achieve maximum performance when using MessagePack for C#](#hints-to-achieve-maximum-performance-when-using-messagepack-for-c)
    - [Use indexed keys instead of string keys (Contractless)](#use-indexed-keys-instead-of-string-keys-contractless)
    - [Create own custom composite resolver](#create-own-custom-composite-resolver)
    - [Use native resolvers](#use-native-resolvers)
    - [Be careful when copying buffers](#be-careful-when-copying-buffers)
    - [Choosing compression](#choosing-compression)
- [Extensions](#extensions)
- [Experimental Features](#experimental-features)
- [High-Level API (`MessagePackSerializer`)](#high-level-api-messagepackserializer)
    - [Multiple MessagePack structures on a single `Stream`](#multiple-messagepack-structures-on-a-single-stream)
- [Low-Level API (`IMessagePackFormatter<T>`)](#low-level-api-imessagepackformattert)
- [Primitive API (`MessagePackWriter`, `MessagePackReader`)](#primitive-api-messagepackwriter-messagepackreader)
    - [`MessagePackReader`](#messagepackreader)
    - [`MessagePackWriter`](#messagepackwriter)
- [Main Extension Point (`IFormatterResolver`)](#main-extension-point-iformatterresolver)
- [MessagePackFormatterAttribute](#messagepackformatterattribute)
- [IgnoreFormatter](#ignoreformatter)
- [Reserved Extension Types](#reserved-extension-types)
- [Unity support](#unity-support)
- [AOT Code Generation (support for Unity/Xamarin)](#aot-code-generation-support-for-unityxamarin)
- [RPC](#rpc)
    - [MagicOnion](#magiconion)
    - [StreamJsonRpc](#streamjsonrpc)
- [How to build](#how-to-build)
- [Author Info](#author-info)

## Installation

This library is distributed via NuGet. Special [Unity support](#unity) is available, too.

이 라이브러리는 NuGet을 통해 배포됩니다. 특별 [유니티 지원](#unity)도 가능합니다. 69

We target .NET Standard 2.0 with special optimizations for .NET Core 2.1+, making it compatible with most reasonably recent .NET runtimes such as Core 2.0 and later, Framework 4.6.1 and later, Mono 5.4 and later and Unity 2018.3 and later.

.NET Core 2.1 이상에 대한 특수 최적화를 통해 .NET Standard 2.0을 대상으로 하여 Core 2.0 이상, Framework 4.6.1 이상, Mono 5.4 이상, Unity 2018.3 이상과 같은 가장 합리적인 최신 .NET 런타임과 호환되도록 합니다. 72

The library code is pure C# (with Just-In-Time IL code generation on some platforms).

라이브러리 코드는 순수 C#입니다(일부 플랫폼에서 Just-In-Time IL 코드 생성 포함). 75


### NuGet packages

To install with NuGet, just install the `MessagePack` package:

```ps1
Install-Package MessagePack
```

Install the optional C# [analyzers](doc/analyzers/index.md) package to get warnings about coding mistakes and automatic fix suggestions to save you time:

선택적 C# [analyzers](doc/analyzers/index.md) 패키지를 설치하여 코딩 실수에 대한 경고와 시간 절약을 위한 자동 수정 제안을 받으세요.

```ps1
Install-Package MessagePackAnalyzer
```

There are also a range of official and third party Extension Packages available (learn more in our [extensions section](#extensions)):

또한 다양한 공식 및 타사 확장 패키지를 사용할 수 있습니다(자세한 내용은 [확장 섹션](#extensions) 참조). 98


```ps1
Install-Package MessagePack.ReactiveProperty
Install-Package MessagePack.UnityShims
Install-Package MessagePack.AspNetCoreMvcFormatter
```

### Unity

For Unity projects, the [releases][Releases] page provides downloadable `.unitypackage` files. When using in Unity IL2CPP or Xamarin AOT environments, please carefully read the [pre-code generation section](#aot).

Unity 프로젝트의 경우 [릴리스][릴리스] 페이지에서 다운로드 가능한 `.unitypackage` 파일을 제공합니다. Unity IL2CPP 또는 Xamarin AOT 환경에서 사용할 경우 [사전 코드 생성 섹션](#aot)을 주의 깊게 읽으십시오. 111


### Migration notes from v1.x

If you were using MessagePack for C# v1.x, check out the ["How to update to our new v2.x version"](doc/migration.md) document.

## Quick Start

Define the struct or class to be serialized and annotate it with a `[MessagePackObject]` attribute.
직렬화할 구조체 또는 클래스를 정의하고 `[MessagePackObject]` 속성으로 주석을 답니다. 122
Annotate members whose values should be serialized (fields as well as properties) with `[Key]` attributes.
값을 직렬화해야 하는 멤버(필드 및 속성)를 `[Key]` 속성으로 주석을 답니다. 124


```csharp
[MessagePackObject]
public class MyClass
{
    // Key attributes take a serialization index (or string name)
    // The values must be unique and versioning has to be considered as well.
    // Keys are described in later sections in more detail.
    [Key(0)]
    public int Age { get; set; }

    [Key(1)]
    public string FirstName { get; set; }

    [Key(2)]
    public string LastName { get; set; }

    // All fields or properties that should not be serialized must be annotated with [IgnoreMember].
    [IgnoreMember]
    public string FullName { get { return FirstName + LastName; } }
}
```

Call `MessagePackSerializer.Serialize<T>/Deserialize<T>` to serialize/deserialize your object instance.

`MessagePackSerializer.Serialize<T>/Deserialize<T>`를 호출하여 개체 인스턴스를 직렬화/역직렬화합니다. 150

You can use the `ConvertToJson` method to get a human readable representation of any MessagePack binary blob.

'ConvertToJson' 메서드를 사용하여 사람이 읽을 수 있는 MessagePack 바이너리 blob을 얻을 수 있습니다. 153


```csharp
class Program
{
    static void Main(string[] args)
    {
        var mc = new MyClass
        {
            Age = 99,
            FirstName = "hoge",
            LastName = "huga",
        };

        // Call Serialize/Deserialize, that's all.
        byte[] bytes = MessagePackSerializer.Serialize(mc);
        MyClass mc2 = MessagePackSerializer.Deserialize<MyClass>(bytes);

        // You can dump MessagePack binary blobs to human readable json.
        // Using indexed keys (as opposed to string keys) will serialize to MessagePack arrays,
        // hence property names are not available.
        // [99,"hoge","huga"]
        var json = MessagePackSerializer.ConvertToJson(bytes);
        Console.WriteLine(json);
    }
}
```

By default, a `MessagePackObject` annotation is required. This can be made optional; see the [Object Serialization section](#object-serialization) and the [Formatter Resolver section](#resolvers) for details.

기본적으로 'MessagePackObject' 주석이 필요합니다. 이것은 선택 사항으로 만들 수 있습니다. 자세한 내용은 [Object Serialization 섹션](#object-serialization) 및 [Formatter Resolver 섹션](#resolvers)을 참조하세요. 185


## Analyzer

The MessagePackAnalyzer package aids with:

MessagePackAnalyzer 패키지는 다음을 지원합니다. 192

1. Automating definitions for your serializable objects.
2. Produces compiler warnings upon incorrect attribute use, member accessibility, and more.
 잘못된 속성 사용, 멤버 액세스 가능성 등에 대해 컴파일러 경고를 생성합니다. 197


![analyzergif](https://cloud.githubusercontent.com/assets/46207/23837445/ce734eae-07cb-11e7-9758-d69f0f095bc1.gif)

If you want to allow a specific custom type (for example, when registering a custom type), put `MessagePackAnalyzer.json` at the project root and change the Build Action to `AdditionalFiles`.

특정 사용자 정의 유형(예: 사용자 정의 유형 등록 시)을 허용하려면 프로젝트 루트에 'MessagePackAnalyzer.json'을 넣고 Build Action을 'AdditionalFiles'로 변경합니다.

![image](https://cloud.githubusercontent.com/assets/46207/23837427/8a8d507c-07cb-11e7-9277-5a566eb0bfde.png)

An example `MessagePackAnalyzer.json`:

```json
[ "MyNamespace.FooClass", "MyNameSpace.BarStruct" ]
```

## Built-in supported types

These types can serialize by default:

* Primitives (`int`, `string`, etc...), `Enum`s, `Nullable<>`, `Lazy<>`
* `TimeSpan`,  `DateTime`, `DateTimeOffset`
* `Guid`, `Uri`, `Version`, `StringBuilder`
* `BigInteger`, `Complex`, `Half`
* `Array[]`, `Array[,]`, `Array[,,]`, `Array[,,,]`, `ArraySegment<>`, `BitArray`
* `KeyValuePair<,>`, `Tuple<,...>`, `ValueTuple<,...>`
* `ArrayList`, `Hashtable`
* `List<>`, `LinkedList<>`, `Queue<>`, `Stack<>`, `HashSet<>`, `ReadOnlyCollection<>`, `SortedList<,>`
* `IList<>`, `ICollection<>`, `IEnumerable<>`, `IReadOnlyCollection<>`, `IReadOnlyList<>`
* `Dictionary<,>`, `IDictionary<,>`, `SortedDictionary<,>`, `ILookup<,>`, `IGrouping<,>`, `ReadOnlyDictionary<,>`, `IReadOnlyDictionary<,>`
* `ObservableCollection<>`, `ReadOnlyObservableCollection<>`
* `ISet<>`,
* `ConcurrentBag<>`, `ConcurrentQueue<>`, `ConcurrentStack<>`, `ConcurrentDictionary<,>`
* Immutable collections (`ImmutableList<>`, etc)
* Custom implementations of `ICollection<>` or `IDictionary<,>` with a parameterless constructor
* Custom implementations of `IList` or `IDictionary` with a parameterless constructor

You can add support for custom types, and there are some official/third-party extension packages for:

* ReactiveProperty
* for Unity (`Vector3`, `Quaternion`, etc...)
* F# (Record, FsList, Discriminated Unions, etc...)

Please see the [extensions section](#extensions).

`MessagePack.Nil` is the built-in type representing null/void in MessagePack for C#.

'MessagePack.Nil'은 C#용 MessagePack에서 null/void를 나타내는 기본 제공 형식입니다. 244


## Object Serialization 객체 직렬화

MessagePack for C# can serialize your own public `class` or `struct` types. By default, serializable types must be annotated with the `[MessagePackObject]` attribute and members with the `[Key]` attribute. 

C#용 MessagePack은 자신의 공개 '클래스' 또는 '구조체' 유형을 직렬화할 수 있습니다. 기본적으로 직렬화 가능한 유형은 `[MessagePackObject]` 속성으로 주석을 달고 멤버는 `[Key]` 속성으로 주석을 달아야 합니다.

Keys can be either indexes (`int`) or arbitrary strings. If all keys are indexes, arrays are used for serialization, which offers advantages in performance and binary size. Otherwise, MessagePack maps (dictionaries) will be used.

키는 인덱스(`int`) 또는 임의의 문자열일 수 있습니다. 모든 키가 인덱스인 경우 직렬화에 배열이 사용되므로 성능 및 바이너리 크기 면에서 이점이 있습니다. 그렇지 않으면 MessagePack 맵(사전)이 사용됩니다.


If you use `[MessagePackObject(keyAsPropertyName: true)]`, then members do not require explicit `Key` attributes, but string keys will be used.

`[MessagePackObject(keyAsPropertyName: true)]`를 사용하는 경우 멤버는 명시적인 `Key` 속성을 필요로 하지 않지만 문자열 키가 사용됩니다. 260

```csharp
[MessagePackObject]
public class Sample1
{
    [Key(0)]
    public int Foo { get; set; }
    [Key(1)]
    public int Bar { get; set; }
}

[MessagePackObject]
public class Sample2
{
    [Key("foo")]
    public int Foo { get; set; }
    [Key("bar")]
    public int Bar { get; set; }
}

[MessagePackObject(keyAsPropertyName: true)]
public class Sample3
{
    // No need for a Key attribute
    public int Foo { get; set; }

    // If want to ignore a public member, you can use the  IgnoreMember attribute
    [IgnoreMember]
    public int Bar { get; set; }
}

// [10,20]
Console.WriteLine(MessagePackSerializer.SerializeToJson(new Sample1 { Foo = 10, Bar = 20 }));

// {"foo":10,"bar":20}
Console.WriteLine(MessagePackSerializer.SerializeToJson(new Sample2 { Foo = 10, Bar = 20 }));

// {"Foo":10}
Console.WriteLine(MessagePackSerializer.SerializeToJson(new Sample3 { Foo = 10, Bar = 20 }));
```

All public instance members (fields as well as properties) will be serialized. If you want to ignore certain public members, annotate the member with a `[IgnoreMember]` attribute.

모든 공개 인스턴스 멤버(필드 및 속성)는 직렬화됩니다. 특정 공개 멤버를 무시하려면 `[IgnoreMember]` 속성으로 멤버에 주석을 답니다. 304

Please note that any serializable struct or class must have public accessibility; private and internal structs and classes cannot be serialized!
The default of requiring `MessagePackObject` annotations is meant to enforce explicitness and therefore may help write more robust code.

직렬화 가능한 구조체 또는 클래스에는 공용 액세스 가능성이 있어야 합니다. private 및 내부 구조체와 클래스는 직렬화할 수 없습니다! 308
'MessagePackObject' 주석을 요구하는 기본값은 명시성을 적용하기 위한 것이며 따라서 더 강력한 코드를 작성하는 데 도움이 될 수 있습니다. 309


Should you use an indexed (`int`) key or a string key?
We recommend using indexed keys for faster serialization and a more compact binary representation than string keys.
However, the additional information in the strings of string keys can be quite useful when debugging.

번역 결과
인덱싱된(`int`) 키 또는 문자열 키를 사용해야 합니까?
더 빠른 직렬화와 문자열 키보다 더 간결한 이진 표현을 위해 인덱싱된 키를 사용하는 것이 좋습니다.
그러나 문자열 키 문자열의 추가 정보는 디버깅할 때 매우 유용할 수 있습니다.

When classes change or are extended, be careful about versioning. `MessagePackSerializer` will initialize members to their `default` value if a key does not exist in the serialized binary blob, meaning members using reference types can be initialized to `null`.
If you use indexed (`int`) keys, the keys should start at 0 and should be sequential. If a later version stops using certain members, you should keep the obsolete members (C# provides an `Obsolete` attribute to annotate such members) until all other clients had a chance to update and remove their uses of these members as well. Also, when the values of indexed keys "jump" a lot, leaving gaps in the sequence, it will negatively affect the binary size, as `null` placeholders will be inserted into the resulting arrays. However, you shouldn't reuse indexes of removed members to avoid compatibility issues between clients or when trying to deserialize legacy blobs.

클래스가 변경되거나 확장될 때 버전 관리에 주의하십시오. 'MessagePackSerializer'는 직렬화된 바이너리 blob에 키가 없으면 멤버를 '기본' 값으로 초기화합니다. 즉, 참조 유형을 사용하는 멤버는 'null'로 초기화될 수 있습니다.
인덱싱된(`int`) 키를 사용하는 경우 키는 0에서 시작하고 순차적이어야 합니다. 이후 버전이 특정 멤버의 사용을 중지하는 경우 다른 모든 클라이언트가 이러한 멤버의 사용을 업데이트하고 제거할 기회가 있을 때까지 더 이상 사용되지 않는 멤버를 유지해야 합니다(C#은 이러한 멤버에 주석을 달기 위해 'Obsolete' 속성을 제공함). 또한 인덱싱된 키의 값이 많이 "점프"되어 시퀀스에 간격이 남으면 'null' 자리 표시자가 결과 배열에 삽입되므로 바이너리 크기에 부정적인 영향을 미칩니다. 그러나 클라이언트 간의 호환성 문제를 방지하거나 레거시 Blob을 역직렬화하려고 할 때 제거된 구성원의 인덱스를 재사용해서는 안 됩니다.

Example of index gaps and resulting placeholders:

```csharp
[MessagePackObject]
public class IntKeySample
{
    [Key(3)]
    public int A { get; set; }
    [Key(10)]
    public int B { get; set; }
}

// [null,null,null,0,null,null,null,null,null,null,0]
Console.WriteLine(MessagePackSerializer.SerializeToJson(new IntKeySample()));
```

If you do not want to explicitly annotate with the `MessagePackObject`/`Key` attributes and instead want to use MessagePack for C# more like e.g. [Json.NET](https://github.com/JamesNK/Newtonsoft.Json), you can make use of the contractless resolver.

`MessagePackObject`/`Key` 속성으로 명시적으로 주석을 달고 싶지 않고 대신 C#용 MessagePack을 사용하려는 경우 [Json.NET](https://github.com/JamesNK/Newtonsoft.Json)에서는 무계약 리졸버를 사용할 수 있습니다.

```csharp
public class ContractlessSample
{
    public int MyProperty1 { get; set; }
    public int MyProperty2 { get; set; }
}

var data = new ContractlessSample { MyProperty1 = 99, MyProperty2 = 9999 };
var bin = MessagePackSerializer.Serialize(
  data,
  MessagePack.Resolvers.ContractlessStandardResolver.Options);

// {"MyProperty1":99,"MyProperty2":9999}
Console.WriteLine(MessagePackSerializer.ConvertToJson(bin));

// You can also set ContractlessStandardResolver as the default.
// (Global state; Not recommended when writing library code)
MessagePackSerializer.DefaultOptions = MessagePack.Resolvers.ContractlessStandardResolver.Options;

// Now serializable...
var bin2 = MessagePackSerializer.Serialize(data);
```

If you want to serialize private members as well, you can use one of the `*AllowPrivate` resolvers.

비공개 멤버도 직렬화하려면 `*AllowPrivate` 해석기 중 하나를 사용할 수 있습니다. 373


```csharp
[MessagePackObject]
public class PrivateSample
{
    [Key(0)]
    int x;

    public void SetX(int v)
    {
        x = v;
    }

    public int GetX()
    {
        return x;
    }
}

var data = new PrivateSample();
data.SetX(9999);

// You can choose either StandardResolverAllowPrivate
// or ContractlessStandardResolverAllowPrivate
var bin = MessagePackSerializer.Serialize(
  data,
  MessagePack.Resolvers.DynamicObjectResolverAllowPrivate.Options);
```

If you want to use MessagePack for C# more like a BinaryFormatter with a typeless serialization API, use the typeless resolver and helpers. Please consult the [Typeless section](#typeless).

Resolvers are the way to add specialized support for custom types to MessagePack for C#. Please refer to the [Extension point section](#resolvers).

유형이 없는 직렬화 API가 있는 BinaryFormatter처럼 C#용 MessagePack을 사용하려면 유형이 없는 해석기와 도우미를 사용하십시오. [Typeless 섹션](#typeless)을 참조하십시오.

해석기는 C#용 MessagePack에 사용자 지정 유형에 대한 특수 지원을 추가하는 방법입니다. [확장 포인트 섹션](#resolvers)을 참조하십시오.

## DataContract compatibility

You can use `[DataContract]` annotations instead of `[MessagePackObject]` ones. If type is annotated with `DataContract`, you can use `[DataMember]` annotations instead of `[Key]` ones and `[IgnoreDataMember]` instead of `[IgnoreMember]`.

Then `[DataMember(Order = int)]` will behave the same as `[Key(int)]`, `[DataMember(Name = string)]` the same as `[Key(string)]`, and `[DataMember]` the same as `[Key(nameof(member name)]`.

Using `DataContract`, e.g. in shared libraries, makes your classes/structs independent from MessagePack for C# serialization. However, it is not supported by the analyzers nor in code generation by the `mpc` tool. Also, features like `UnionAttribute`, `MessagePackFormatter`, `SerializationConstructor`, etc can not be used. Due to this, we recommend that you use the specific MessagePack for C# annotations when possible.

`[MessagePackObject]` 주석 대신 `[DataContract]` 주석을 사용할 수 있습니다. 유형이 `DataContract`로 주석 처리된 경우 `[Key]` 대신 `[DataMember]` 주석을, `[IgnoreMember]` 대신 `[IgnoreDataMember]`를 사용할 수 있습니다.

그러면 `[DataMember(Order = int)]`는 `[Key(int)]`와 동일하게 작동하고, `[DataMember(Name = string)]`는 `[Key(string)]`과 동일하게, `[ DataMember]`는 `[Key(nameof(member name)]`과 동일합니다.

`DataContract` 사용, 예: 공유 라이브러리에서 클래스/구조체를 C# 직렬화용 MessagePack과 독립적으로 만듭니다. 그러나 분석기나 'mpc' 도구에 의한 코드 생성에서는 지원되지 않습니다. 또한 `UnionAttribute`, `MessagePackFormatter`, `SerializationConstructor` 등과 같은 기능을 사용할 수 없습니다. 이 때문에 가능한 경우 특정 C# 주석용 MessagePack을 사용하는 것이 좋습니다.

## Serializing readonly/immutable object members  (SerializationConstructor)

MessagePack for C# supports serialization of readonly/immutable objects/members. For example, this struct can be serialized and deserialized.

C#용 MessagePack은 읽기 전용/불변 개체/멤버의 직렬화를 지원합니다. 예를 들어, 이 구조체는 직렬화 및 역직렬화될 수 있습니다.

```csharp
[MessagePackObject]
public struct Point
{
    [Key(0)]
    public readonly int X;
    [Key(1)]
    public readonly int Y;

    public Point(int x, int y)
    {
        this.X = x;
        this.Y = y;
    }
}

var data = new Point(99, 9999);
var bin = MessagePackSerializer.Serialize(data);

// Okay to deserialize immutable object
var point = MessagePackSerializer.Deserialize<Point>(bin);
```

`MessagePackSerializer` will choose the constructor with the best matched argument list, using argument indexes index for index keys, or parameter names for string keys. If it cannot determine an appropriate constructor, a `MessagePackDynamicObjectResolverException: can't find matched constructor parameter` exception will be thrown.
You can specify which constructor to use manually with a `[SerializationConstructor]` annotation.

'MessagePackSerializer'는 인덱스 키의 경우 인수 인덱스 인덱스를 사용하거나 문자열 키의 경우 매개변수 이름을 사용하여 가장 일치하는 인수 목록이 있는 생성자를 선택합니다. 적절한 생성자를 결정할 수 없으면 `MessagePackDynamicObjectResolverException: 일치하는 생성자 매개변수를 찾을 수 없음` 예외가 발생합니다.
`[SerializationConstructor]` 주석으로 수동으로 사용할 생성자를 지정할 수 있습니다.

```csharp
[MessagePackObject]
public struct Point
{
    [Key(0)]
    public readonly int X;
    [Key(1)]
    public readonly int Y;

    [SerializationConstructor]
    public Point(int x)
    {
        this.X = x;
        this.Y = -1;
    }

    // If not marked attribute, used this(most matched argument)
    public Point(int x, int y)
    {
        this.X = x;
        this.Y = y;
    }
}
```

### C# 9 `record` types C9은 나중에 보자.. 

C# 9.0 record with primary constructor is similar immutable object, also supports serialize/deserialize.

```csharp
// use key as property name
[MessagePackObject(true)]public record Point(int X, int Y);

// use property: to set KeyAttribute
[MessagePackObject] public record Point([property:Key(0)] int X, [property: Key(1)] int Y);

// Or use explicit properties
[MessagePackObject]
public record Person
{
    [Key(0)]
    public string FirstName { get; init; }

    [Key(1)]
    public string LastName { get; init; }
}
```

### C# 9 `init` property setter limitations

When using `init` property setters in _generic_ classes, [a CLR bug](https://github.com/neuecc/MessagePack-CSharp/issues/1134) prevents our most efficient code generation from invoking the property setter.
As a result, you should avoid using `init` on property setters in generic classes when using the public-only `DynamicObjectResolver`/`StandardResolver`.

When using the `DynamicObjectResolverAllowPrivate`/`StandardResolverAllowPrivate` resolver the bug does not apply and you may use `init` without restriction.

## Serialization Callback 직렬화 콜백

Objects implementing the `IMessagePackSerializationCallbackReceiver` interface will received `OnBeforeSerialize` and `OnAfterDeserialize` calls during serialization/deserialization.

`IMessagePackSerializationCallbackReceiver` 인터페이스를 구현하는 객체는 직렬화/역직렬화 중에 `OnBeforeSerialize` 및 `OnAfterDeserialize` 호출을 수신합니다.


```csharp
[MessagePackObject]
public class SampleCallback : IMessagePackSerializationCallbackReceiver
{
    [Key(0)]
    public int Key { get; set; }

    public void OnBeforeSerialize()
    {
        Console.WriteLine("OnBefore");
    }

    public void OnAfterDeserialize()
    {
        Console.WriteLine("OnAfter");
    }
}
```

## Union (ㅇ?유?니온? 이 )

MessagePack for C# supports serializing interface-typed and abstract class-typed objects. It behaves like `XmlInclude` or `ProtoInclude`. In MessagePack for C# these are called `Union`. Only interfaces and abstracts classes are allowed to be annotated with `Union` attributes. Unique union keys are required.

C#용 MessagePack은 인터페이스 유형 및 추상 클래스 유형 개체 직렬화를 지원합니다. `XmlInclude` 또는 `ProtoInclude`처럼 작동합니다. C#용 MessagePack에서는 이를 'Union'이라고 합니다. 인터페이스 및 추상 클래스에만 'Union' 속성으로 주석을 추가할 수 있습니다. 고유한 유니온 키가 필요합니다.

```csharp
// Annotate inheritance types
[MessagePack.Union(0, typeof(FooClass))]
[MessagePack.Union(1, typeof(BarClass))]
public interface IUnionSample
{
}

[MessagePackObject]
public class FooClass : IUnionSample
{
    [Key(0)]
    public int XYZ { get; set; }
}

[MessagePackObject]
public class BarClass : IUnionSample
{
    [Key(0)]
    public string OPQ { get; set; }
}

// ---

IUnionSample data = new FooClass() { XYZ = 999 };

// Serialize interface-typed object.
var bin = MessagePackSerializer.Serialize(data);

// Deserialize again.
var reData = MessagePackSerializer.Deserialize<IUnionSample>(bin);

// Use with e.g. type-switching in C# 7.0
switch (reData)
{
    case FooClass x:
        Console.WriteLine(x.XYZ);
        break;
    case BarClass x:
        Console.WriteLine(x.OPQ);
        break;
    default:
        break;
}
```

Unions are internally serialized to two-element arrays.Union은 내부적으로 2요소 배열로 직렬화됩니다. 

```csharp
IUnionSample data = new BarClass { OPQ = "FooBar" };

var bin = MessagePackSerializer.Serialize(data);

// Union is serialized to two-length array, [key, object]
// [1,["FooBar"]]
Console.WriteLine(MessagePackSerializer.ConvertToJson(bin));
```

Using `Union` with abstract classes works the same way.

```csharp
[Union(0, typeof(SubUnionType1))]
[Union(1, typeof(SubUnionType2))]
[MessagePackObject]
public abstract class ParentUnionType
{
    [Key(0)]
    public int MyProperty { get; set; }
}

[MessagePackObject]
public class SubUnionType1 : ParentUnionType
{
    [Key(1)]
    public int MyProperty1 { get; set; }
}

[MessagePackObject]
public class SubUnionType2 : ParentUnionType
{
    [Key(1)]
    public int MyProperty2 { get; set; }
}
```

Please be mindful that you cannot reuse the same keys in derived types that are already present in the parent type, as internally a single flat array or map will be used and thus cannot have duplicate indexes/keys.

내부적으로 단일 평면 배열 또는 맵이 사용되므로 중복 인덱스/키를 가질 수 없으므로 상위 유형에 이미 있는 파생 유형에서 동일한 키를 재사용할 수 없습니다. 유의하십시오

## Dynamic (Untyped) Deserialization 동적할당 자료형 역직렬화

When calling `MessagePackSerializer.Deserialize<object>` or `MessagePackSerializer.Deserialize<dynamic>`, any values present in the blob will be converted to primitive values, i.e. `bool`, `char`, `sbyte`, `byte`, `short`, `int`, `long`, `ushort`, `uint`, `ulong`, `float`, `double`, `DateTime`, `string`, `byte[]`, `object[]`, `IDictionary<object, object>`.

`MessagePackSerializer.Deserialize<object>` 또는 `MessagePackSerializer.Deserialize<dynamic>`을 호출하면 blob에 있는 모든 값이 원시 값(예: `bool`, `char`, `sbyte`, `byte`)으로 변환됩니다. `short`, `int`, `long`, `ushort`, `uint`, `ulong`, `float`, `double`, `DateTime`, `string`, `byte[]`, `object[] `, `IDictionary<객체, 객체>`.

```csharp
// Sample blob.
var model = new DynamicModel { Name = "foobar", Items = new[] { 1, 10, 100, 1000 } };
var blob = MessagePackSerializer.Serialize(model, ContractlessStandardResolver.Options);

// Dynamic ("untyped")
var dynamicModel = MessagePackSerializer.Deserialize<dynamic>(blob, ContractlessStandardResolver.Options);

// You can access the data using array/dictionary indexers, as shown above
Console.WriteLine(dynamicModel["Name"]); // foobar
Console.WriteLine(dynamicModel["Items"][2]); // 100
```

Exploring object trees using the dictionary indexer syntax is the fastest option for untyped deserialization, but it is tedious to read and write.
Where performance is not as important as code readability, consider deserializing with [ExpandoObject](doc/ExpandoObject.md).

사전 인덱서 구문을 사용하여 개체 트리를 탐색하는 것은 형식화되지 않은 역직렬화를 위한 가장 빠른 옵션이지만 읽고 쓰는 것은 지루합니다.
성능이 코드 가독성만큼 중요하지 않은 경우 [ExpandoObject](doc/ExpandoObject.md)를 사용한 역직렬화를 고려하십시오.

## Object Type Serialization 객체 형식 직렬화 ㅎ..

`StandardResolver` and `ContractlessStandardResolver` can serialize `object`/anonymous typed objects.

```csharp
var objects = new object[] { 1, "aaa", new ObjectFieldType { Anything = 9999 } };
var bin = MessagePackSerializer.Serialize(objects);

// [1,"aaa",[9999]]
Console.WriteLine(MessagePackSerializer.ConvertToJson(bin));

// Support anonymous Type Serialize
var anonType = new { Foo = 100, Bar = "foobar" };
var bin2 = MessagePackSerializer.Serialize(anonType, MessagePack.Resolvers.ContractlessStandardResolver.Options);

// {"Foo":100,"Bar":"foobar"}
Console.WriteLine(MessagePackSerializer.ConvertToJson(bin2));
```

> Unity supports is limited.

When deserializing, the behavior will be the same as Dynamic (Untyped) Deserialization.

## Typeless

The typeless API is similar to `BinaryFormatter`, as it will embed type information into the blobs, so no types need to be specified explicitly when calling the API.

유형이 없는 API는 'BinaryFormatter'와 유사합니다. 유형 정보를 Blob에 포함하므로 API를 호출할 때 유형을 명시적으로 지정할 필요가 없기 때문입니다. 689


```csharp
object mc = new Sandbox.MyClass()
{
    Age = 10,
    FirstName = "hoge",
    LastName = "huga"
};

// Serialize with the typeless API
var blob = MessagePackSerializer.Typeless.Serialize(mc);

// Blob has embedded type-assembly information.
// ["Sandbox.MyClass, Sandbox",10,"hoge","huga"]
Console.WriteLine(MessagePackSerializer.ConvertToJson(bin));

// You can deserialize to MyClass again with the typeless API
// Note that no type has to be specified explicitly in the Deserialize call
// as type information is embedded in the binary blob
var objModel = MessagePackSerializer.Typeless.Deserialize(bin) as MyClass;
```

Type information is represented by the MessagePack `ext` format, type code `100`.

`MessagePackSerializer.Typeless` is a shortcut of `Serialize/Deserialize<object>(TypelessContractlessStandardResolver.Instance)`.
If you want to configure it as the default resolver, you can use `MessagePackSerializer.Typeless.RegisterDefaultResolver`.

`TypelessFormatter` can used standalone or combined with other resolvers.

```csharp
// Replaced `object` uses the typeless resolver
var resolver = MessagePack.Resolvers.CompositeResolver.Create(
    new[] { MessagePack.Formatters.TypelessFormatter.Instance },
    new[] { MessagePack.Resolvers.StandardResolver.Instance });

public class Foo
{
    // use Typeless(this field only)
    [MessagePackFormatter(typeof(TypelessFormatter))]
    public object Bar;
}
```

If a type's name is changed later, you can no longer deserialize old blobs. But you can specify a fallback name in such cases, providing a `TypelessFormatter.BindToType` function of your own.

```csharp
MessagePack.Formatters.TypelessFormatter.BindToType = typeName =>
{
    if (typeName.StartsWith("SomeNamespace"))
    {
        typeName = typeName.Replace("SomeNamespace", "AnotherNamespace");
    }

    return Type.GetType(typeName, false);
};
```

## <a name="security"></a>Security

Deserializing data from an untrusted source can introduce security vulnerabilities in your application.
Depending on the settings used during deserialization, **untrusted data may be able to execute arbitrary code** or cause a denial of service attack.
Untrusted data might come from over the network from an untrusted source (e.g. any and every networked client) or can be tampered with by an intermediary when transmitted over an unauthenticated connection, or from a local storage that might have been tampered with, or many other sources. MessagePack for C# does not provide any means to authenticate data or make it tamper-resistant. Please use an appropriate method of authenticating data before deserialization - such as a [`MAC`](https://en.wikipedia.org/wiki/Message_authentication_code) .

신뢰할 수 없는 소스의 데이터를 역직렬화하면 애플리케이션에 보안 취약점이 발생할 수 있습니다.
역직렬화 중에 사용된 설정에 따라 **신뢰할 수 없는 데이터가 임의의 코드를 실행**하거나 서비스 거부 공격을 일으킬 수 있습니다.
신뢰할 수 없는 데이터는 신뢰할 수 없는 소스(예: 네트워크로 연결된 모든 클라이언트)에서 네트워크를 통해 가져오거나 인증되지 않은 연결을 통해 전송될 때 중개자에 의해 변조될 수 있습니다. 소스. C#용 MessagePack은 데이터를 인증하거나 변조 방지할 수 있는 수단을 제공하지 않습니다. [`MAC`](https://en.wikipedia.org/wiki/Message_authentication_code) 와 같이 역직렬화 전에 데이터를 인증하는 적절한 방법을 사용하세요.


Please be very mindful of these attack scenarios; many projects and companies, and serialization library users in general, have been bitten by untrusted user data deserialization in the past.

When deserializing untrusted data, put MessagePack into a more secure mode by configuring your `MessagePackSerializerOptions.Security` property:

이러한 공격 시나리오에 매우 유의하십시오. 많은 프로젝트와 회사, 그리고 일반적으로 직렬화 라이브러리 사용자는 과거에 신뢰할 수 없는 사용자 데이터 역직렬화에 물렸습니다.

신뢰할 수 없는 데이터를 역직렬화할 때 `MessagePackSerializerOptions.Security` 속성을 구성하여 MessagePack을 보다 안전한 모드로 전환합니다.

```cs
var options = MessagePackSerializerOptions.Standard
    .WithSecurity(MessagePackSecurity.UntrustedData);

// Pass the options explicitly for the greatest control.
T object = MessagePackSerializer.Deserialize<T>(data, options);

// Or set the security level as the default.
MessagePackSerializer.DefaultOptions = options;
```

You should also avoid the Typeless serializer/formatters/resolvers for untrusted data as that opens the door for the untrusted data to potentially deserialize unanticipated types that can compromise security.

The `UntrustedData` mode merely hardens against some common attacks, but is no fully secure solution in itself.

또한 신뢰할 수 없는 데이터가 보안을 손상시킬 수 있는 예상치 못한 유형을 역직렬화할 가능성이 있으므로 신뢰할 수 없는 데이터에 대한 Typeless 직렬 변환기/포매터/리졸버를 피해야 합니다.

'UntrustedData' 모드는 일부 일반적인 공격에 대해 강화할 뿐 그 자체로 완전히 안전한 솔루션은 아닙니다.

## Performance

Benchmarks comparing MessagePack For C# to other serializers were run on `Windows 10 Pro x64 Intel Core i7-6700K 4.00GHz, 32GB RAM`. Benchmark code is [available here](https://github.com/neuecc/ZeroFormatter/tree/master/sandbox/PerformanceComparison) - and their [version info](https://github.com/neuecc/ZeroFormatter/blob/bc63cb925d/sandbox/PerformanceComparison/packages.config).
[ZeroFormatter](https://github.com/neuecc/ZeroFormatter/) and [FlatBuffers](https://google.github.io/flatbuffers/) have infinitely fast deserializers, so ignore their deserialization performance.

![image](https://cloud.githubusercontent.com/assets/46207/23835765/55fe494e-07b0-11e7-98be-5e7a9411da40.png)

 MessagePack for C# uses many techniques to improve performance. 성능을 높히기 위해 이런것들을 했구나.. 나중에 나도 성능 향상을 해야할 수도있으니 언제 읽어는보자.

* The serializer uses `IBufferWriter<byte>` rather than `System.IO.Stream` to reduce memory overhead.
* Buffers are rented from pools to reduce allocations, keeping throughput high through reduced GC pressure.
* Don't create intermediate utility instances (`*Writer/*Reader`, `*Context`, etc...)
* Utilize dynamic code generation and JIT to avoid boxing value types. Use AOT generation on platforms that prohibit JITs.
* Cached generated formatters on static generic fields (don't use dictionary-cache because dictionary lookup is overhead). See [Resolvers](https://github.com/neuecc/MessagePack-CSharp/tree/209f301e2e595ed366408624011ba2e856d23429/src/MessagePack/Resolvers)
* Heavily tuned dynamic IL code generation and JIT to avoid boxing value types. See [DynamicObjectTypeBuilder](https://github.com/neuecc/MessagePack-CSharp/blob/209f301e2e595ed366408624011ba2e856d23429/src/MessagePack/Resolvers/DynamicObjectResolver.cs#L142-L754). Use AOT generation on platforms that prohibit JIT.
* Call the Primitive API directly when IL code generation determines target types to be  primitive.
* Reduce branching of variable length formats when IL code generation knows the target type (integer/string) ranges
* Don't use the `IEnumerable<T>` abstraction to iterate over collections when possible, [see: CollectionFormatterBase](https://github.com/neuecc/MessagePack-CSharp/blob/209f301e2e595ed366408624011ba2e856d23429/src/MessagePack/Formatters/CollectionFormatter.cs#L192-L355) and derived collection formatters
* Use pre-generated lookup tables to reduce checks of mgpack type constraints, [see: MessagePackBinary](https://github.com/neuecc/MessagePack-CSharp/blob/209f301e2e595ed366408624011ba2e856d23429/src/MessagePack/MessagePackBinary.cs#L15-L212)
* Uses optimized type key dictionary for non-generic methods, [see: ThreadsafeTypeKeyHashTable](https://github.com/neuecc/MessagePack-CSharp/blob/91312921cb7fe987f48336768c898a76ac7dbb40/src/MessagePack/Internal/ThreadsafeTypeKeyHashTable.cs)
* Avoid string key decoding for lookup maps (string key and use automata based name lookup with inlined IL code generation, see: [AutomataDictionary](https://github.com/neuecc/MessagePack-CSharp/blob/bcedbce3fd98cb294210d6b4a22bdc4c75ccd916/src/MessagePack/Internal/AutomataDictionary.cs)
* To encode string keys, use pre-generated member name bytes and fixed sized byte array copies in IL, see: [UnsafeMemory.cs](https://github.com/neuecc/MessagePack-CSharp/blob/f17ddc5d107d3a2f66f60398b214ef87919ff892/src/MessagePack/Internal/UnsafeMemory.cs)

Before creating this library, I implemented a fast fast serializer with [ZeroFormatter#Performance](https://github.com/neuecc/ZeroFormatter#performance). This is a further evolved implementation. MessagePack for C# is always fast and optimized for all types (primitive, small struct, large object, any collections).

### <a name="deserialize-performance"></a>Deserialization Performance for different options

Performance varies depending on the options used. This is a micro benchmark with [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet). The target object has 9 members (`MyProperty1` ~ `MyProperty9`), values are zero.

 |              Method |        Mean | Error | Scaled |  Gen 0 | Allocated |
 |-------------------- |------------:|------:|-------:|-------:|----------:|
 |            M IntKey |    72.67 ns |    NA |   1.00 | 0.0132 |      56 B |
 |         M StringKey |   217.95 ns |    NA |   3.00 | 0.0131 |      56 B |
 |   M Typeless_IntKey |   176.71 ns |    NA |   2.43 | 0.0131 |      56 B |
 |M Typeless_StringKey |   378.64 ns |    NA |   5.21 | 0.0129 |      56 B |
 |       MsgPackCliMap | 1,355.26 ns |    NA |  18.65 | 0.1431 |     608 B |
 |     MsgPackCliArray |   455.28 ns |    NA |   6.26 | 0.0415 |     176 B |
 |         ProtobufNet |   265.85 ns |    NA |   3.66 | 0.0319 |     136 B |
 |            Hyperion |   366.47 ns |    NA |   5.04 | 0.0949 |     400 B |
 |       JsonNetString | 2,783.39 ns |    NA |  38.30 | 0.6790 |    2864 B |
 | JsonNetStreamReader | 3,297.90 ns |    NA |  45.38 | 1.4267 |    6000 B |
 |           JilString |   553.65 ns |    NA |   7.62 | 0.0362 |     152 B |
 |     JilStreamReader | 1,408.46 ns |    NA |  19.38 | 0.8450 |    3552 B |

`ÌntKey`, `StringKey`, `Typeless_IntKey`, `Typeless_StringKey` are MessagePack for C# options. All MessagePack for C# options achieve zero memory allocations in the deserialization process. `JsonNetString`/`JilString` is deserialized from strings. `JsonNetStreamReader`/`JilStreamReader` is deserialized from UTF-8 byte arrays using `StreamReader`. Deserialization is normally read from Stream. Thus, it will be restored from byte arrays (or Stream) instead of strings.

MessagePack for C# `IntKey` is the fastest. `StringKey` is slower than `IntKey` because matching the character string of property names is required. `IntKey` works by reading the array length, then `for (array length) { binary decode }`. `StringKey` works by reading map length, `for (map length) { decode key, lookup key, binary decode }`, so it requires an additional two steps (decoding of keys and lookups of keys).

String key is often a useful, contractless, simple replacement of JSON, interoperability with other languages, and more robust versioning. MessagePack for C# is also optimized for string keys as much a possible. First of all, it does not decode UTF-8 byte arrays to full string for matching with the member name; instead it will look up the byte arrays as it is (to avoid decoding costs and extra memory allocations).

And It will try to match each `long type` (per 8 character, if it is not enough, pad with 0) using [automata](https://en.wikipedia.org/wiki/Automata_theory) and inline it when generating IL code.

![image](https://user-images.githubusercontent.com/46207/29754771-216b40e2-8bc7-11e7-8310-1c3602e80a08.png)

This also avoids calculating the hash code of byte arrays, and the comparison can be made several times faster using the long type.

This is the sample of decompiled generated deserializer code, decompiled using [ILSpy](http://ilspy.net/).

![image](https://user-images.githubusercontent.com/46207/29754804-b5ba0f44-8bc7-11e7-9f6b-0c8f3c041237.png)

If the number of nodes is large, searches will use an embedded binary search.

Extra note, this is serialization benchmark result.

 |              Method |        Mean | Error | Scaled |  Gen 0 | Allocated |
 |-------------------- |------------:|------:|-------:|-------:|----------:|
 |              IntKey |    84.11 ns |    NA |   1.00 | 0.0094 |      40 B |
 |           StringKey |   126.75 ns |    NA |   1.51 | 0.0341 |     144 B |
 |     Typeless_IntKey |   183.31 ns |    NA |   2.18 | 0.0265 |     112 B |
 |  Typeless_StringKey |   193.95 ns |    NA |   2.31 | 0.0513 |     216 B |
 |       MsgPackCliMap |   967.68 ns |    NA |  11.51 | 0.1297 |     552 B |
 |     MsgPackCliArray |   284.20 ns |    NA |   3.38 | 0.1006 |     424 B |
 |         ProtobufNet |   176.43 ns |    NA |   2.10 | 0.0665 |     280 B |
 |            Hyperion |   280.14 ns |    NA |   3.33 | 0.1674 |     704 B |
 |       ZeroFormatter |   149.95 ns |    NA |   1.78 | 0.1009 |     424 B |
 |       JsonNetString | 1,432.55 ns |    NA |  17.03 | 0.4616 |    1944 B |
 | JsonNetStreamWriter | 1,775.72 ns |    NA |  21.11 | 1.5526 |    6522 B |
 |           JilString |   547.51 ns |    NA |   6.51 | 0.3481 |    1464 B |
 |     JilStreamWriter |   778.78 ns |    NA |   9.26 | 1.4448 |    6066 B |

 Of course, `IntKey` is fastest but `StringKey` also performs reasonably well.

## LZ4 Compression 

LZ4: Yann Collet이 개발한 멀티코어를 지원하는 비손실 압축 알고리즘이다. 압축률은 기존 압축 알고리즘인 Deflate 등에 비해 낮지만 속도가 최대 10배 이상으로 매우 빠르다. C로 구현이 되어있고 BSD라이센스를 따른다. 다양한 프로그래밍 언어로 포팅되거나 바인딩되어있다.

MessagePack is a fast and *compact* format but it is not compression. [LZ4](https://github.com/lz4/lz4) is an extremely fast compression algorithm, and using it MessagePack for C# can achieve extremely fast performance as well as extremely compact binary sizes!

MessagePack for C# has built-in LZ4 support. You can activate it using a modified options object and passing it into an API like this:

```cs
var lz4Options = MessagePackSerializerOptions.Standard.WithCompression(MessagePackCompression.Lz4BlockArray);
MessagePackSerializer.Serialize(obj, lz4Options);
```

`MessagePackCompression` has two modes, `Lz4Block` and `Lz4BlockArray`. Neither is a simple binary LZ4 compression, but a special compression integrated into the serialization pipeline, using MessagePack `ext` code (`Lz4BlockArray (98)` or `Lz4Block (99)`). Therefore, it is not readily compatible with compression offered in other languages.

`Lz4Block` compresses an entire MessagePack sequence as a single LZ4 block. This is the simple compression that achieves best compression ratio, at the cost of copying the entire sequence when necessary to get contiguous memory.

`Lz4BlockArray` compresses an entire MessagePack sequence as a array of LZ4 blocks. Compressed/decompressed blocks are  chunked and thus do not enter the GC's Large-Object-Heap, but the compression ratio is slightly worse.

We recommend to use `Lz4BlockArray` as the default when using compression.
For compatibility with MessagePack v1.x, use `Lz4Block`.

Regardless of which LZ4 option is set at the deserialization, both methods can be deserialized. For example, when the `Lz4BlockArray` option was used, binary data using either `Lz4Block` and `Lz4BlockArray` can be deserialized. Neither can be decompressed and hence deserialized when the compression option is set to `None`.

### Attributions

LZ4 compression support is using Milosz Krajewski's [lz4net](https://github.com/MiloszKrajewski/lz4net) code with some modifications.

## <a name="comparison"></a>Comparison with protobuf, JSON, ZeroFormatter

[protobuf-net](https://github.com/mgravell/protobuf-net) is major, widely used binary-format library on .NET. I love protobuf-net and respect their great work. But when you use protobuf-net as a general purpose serialization format, you may encounter an annoying issue.

```csharp
[ProtoContract]
public class Parent
{
    [ProtoMember(1)]
    public int Primitive { get; set; }
    [ProtoMember(2)]
    public Child Prop { get; set; }
    [ProtoMember(3)]
    public int[] Array { get; set; }
}

[ProtoContract]
public class Child
{
    [ProtoMember(1)]
    public int Number { get; set; }
}

using (var ms = new MemoryStream())
{
    // serialize null.
    ProtoBuf.Serializer.Serialize<Parent>(ms, null);

    ms.Position = 0;
    var result = ProtoBuf.Serializer.Deserialize<Parent>(ms);

    Console.WriteLine(result != null); // True, not null. but all property are zero formatted.
    Console.WriteLine(result.Primitive); // 0
    Console.WriteLine(result.Prop); // null
    Console.WriteLine(result.Array); // null
}

using (var ms = new MemoryStream())
{
    // serialize empty array.
    ProtoBuf.Serializer.Serialize<Parent>(ms, new Parent { Array = System.Array.Empty<int>() });

    ms.Position = 0;
    var result = ProtoBuf.Serializer.Deserialize<Parent>(ms);

    Console.WriteLine(result.Array == null); // True, null!
}
```

protobuf(-net) cannot handle null and empty collection correctly, because protobuf has no `null` representation (see [this SO answer from a protobuf-net author](https://stackoverflow.com/questions/21631428/protobuf-net-deserializes-empty-collection-to-null-when-the-collection-is-a-prop/21632160#21632160)).

[MessagePack's type system](https://github.com/msgpack/msgpack/blob/master/spec.md#type-system) can correctly serialize the entire C# type system. This is a strong reason to recommend MessagePack over protobuf.

Protocol Buffers have good IDL and [gRPC](https://www.grpc.io/) support. If you want to use IDL, I recommend [Google.Protobuf](https://github.com/google/protobuf/tree/master/csharp/src/Google.Protobuf) over MessagePack.

JSON is good general-purpose format. It is simple, human-readable and thoroughly-enough specified. [Utf8Json](https://github.com/neuecc/Utf8Json) - which I created as well - adopts same architecture as MessagePack for C# and avoids encoding/decoding costs as much as possible just like this library does. If you want to know more about binary vs text formats, see [Utf8Json/which serializer should be used](https://github.com/neuecc/Utf8Json#which-serializer-should-be-used).

[ZeroFormatter](https://github.com/neuecc/ZeroFormatter/) is similar as [FlatBuffers](https://google.github.io/flatbuffers/) but specialized to C#, and special in that regard. Deserialization is infinitely fast but the produced binary size is larger. And ZeroFormatter's caching algorithm requires additional memory.

For many common uses, MessagePack for C# would be a better fit.

## Hints to achieve maximum performance when using MessagePack for C#

MessagePack for C# prioritizes maximum performance by default. However, there are also some options that sacrifice performance for convenience.

### Use indexed keys instead of string keys (Contractless)

The [Deserialization Performance for different options](https://github.com/neuecc/MessagePack-CSharp#deserialize-performance) section shows the results of indexed keys (`IntKey`) vs string keys (`StringKey`) performance. Indexed keys serialize the object graph as a MessagePack array. String keys serializes the object graph as a MessagePack map.

For example this type is serialized to

```csharp
[MessagePackObject]
public class Person
{
    [Key(0)] or [Key("name")]
    public string Name { get; set;}
    [Key(1)] or [Key("age")]
    public int Age { get; set;}
}

new Person { Name = "foobar", Age = 999 }
```

* `IntKey`: `["foobar", 999]`
* `StringKey`: `{"name:"foobar","age":999}`.

 `IntKey` is always fast in both serialization and deserialization because it does not have to handle and lookup key names, and always has the smaller binary size.

`StringKey` is often a useful, contractless, simple replacement for JSON, interoperability with other languages with MessagePack support, and less error prone versioning. But to achieve maximum performance, use `IntKey`.

### Create own custom composite resolver

`CompositeResolver.Create` is an easy way to create composite resolvers. But formatter lookups have some overhead. If you create a custom resolver (or use `StaticCompositeResolver.Instance`), you can avoid this overhead.

```csharp
public class MyApplicationResolver : IFormatterResolver
{
    public static readonly IFormatterResolver Instance = new MyApplicationResolver();

    // configure your custom resolvers.
    private static readonly IFormatterResolver[] Resolvers = new IFormatterResolver[]
    {
    };

    private MyApplicationResolver() { }

    public IMessagePackFormatter<T> GetFormatter<T>()
    {
        return Cache<T>.Formatter;
    }

    private static class Cache<T>
    {
        public static IMessagePackFormatter<T> Formatter;

        static Cache()
        {
            // configure your custom formatters.
            if (typeof(T) == typeof(XXX))
            {
                Formatter = new ICustomFormatter();
                return;
            }

            foreach (var resolver in Resolvers)
            {
                var f = resolver.GetFormatter<T>();
                if (f != null)
                {
                    Formatter = f;
                    return;
                }
            }
        }
    }
}
```

> NOTE: If you are creating a library, recommend using the above custom resolver instead of `CompositeResolver.Create`. Also, libraries must not use `StaticCompositeResolver` - as it is global state - to avoid compatibility issues.

### Use native resolvers

By default, MessagePack for C# serializes GUID as string. This is much slower than the native .NET format GUID. The same applies to Decimal. If your application makes heavy use of GUID or Decimal and you don't have to worry about interoperability with other languages, you can replace them with the native serializers `NativeGuidResolver` and `NativeDecimalResolver` respectively.

Also, `DateTime` is serialized using the MessagePack timestamp format. By using the `NativeDateTimeResolver`, it is possible to maintain Kind and perform faster serialization.

### Be careful when copying buffers

`MessagePackSerializer.Serialize` returns `byte[]` in default. The final `byte[]` is copied from an internal buffer pool. That is an extra cost.  You can use `IBufferWriter<T>` or the `Stream` API to write to buffers directly. If you want to use a buffer pool outside of the serializer, you should implement custom `IBufferWriter<byte>` or use an existing one such as [`Sequence<T>`](https://github.com/AArnott/Nerdbank.Streams/blob/master/doc/Sequence.md) from the [Nerdbank.Streams](https://nuget.org/packages/Nerdbank.Streams) package.

During deserialization, `MessagePackSerializer.Deserialize(ReadOnlyMemory<byte> buffer)` is better than the `Deserialize(Stream stream)` overload. This is because the Stream API version starts by reading the data, generating a `ReadOnlySequence<byte>`, and only then starts the deserialization.

### Choosing compression

Compression is generally effective when there is duplicate data. In MessagePack, arrays containing objects using string keys (Contractless) can be compressed efficiently because compression can be applied to many duplicate property names. Indexed keys compression is not as effectively compressed as string keys, but indexed keys are smaller in the first place.

This is some example benchmark performance data;

|         Serializer |      Mean |  DataSize |
|------------------- |----------:|----------:|
|             IntKey |  2.941 us |  469.00 B |
|        IntKey(Lz4) |  3.449 us |  451.00 B |
|          StringKey |  4.340 us | 1023.00 B |
|     StringKey(Lz4) |  5.469 us |  868.00 B |

`IntKey(Lz4)` is not as effectively compressed, but performance is still somewhat degraded. On the other hand, `StringKey` can be expected to have a sufficient effect on the binary size. However, this is just an example. Compression can be quite effective depending on the data, too, or have little effect other than slowing down your program. There are also cases in which well-compressible data exists in the values (such as long strings, e.g. containing HTML data with many repeated HTML tags). It is important to verify the actual effects of compression on a case by case basis.

## Extensions

MessagePack for C# has extension points that enable you to provide optimal serialization support for custom types. There are official extension support packages.

```ps1
Install-Package MessagePack.ReactiveProperty
Install-Package MessagePack.UnityShims
Install-Package MessagePack.AspNetCoreMvcFormatter
```

The `MessagePack.ReactiveProperty` package adds support for types of the [ReactiveProperty](https://github.com/runceel/ReactiveProperty) library. It adds `ReactiveProperty<>`, `IReactiveProperty<>`, `IReadOnlyReactiveProperty<>`, `ReactiveCollection<>`, `Unit` serialization support. It is useful for save viewmodel state.

The `MessagePack.UnityShims` package provides shims for [Unity](https://unity3d.com/)'s standard structs (`Vector2`, `Vector3`, `Vector4`, `Quaternion`, `Color`, `Bounds`, `Rect`, `AnimationCurve`, `Keyframe`, `Matrix4x4`, `Gradient`, `Color32`, `RectOffset`, `LayerMask`, `Vector2Int`, `Vector3Int`, `RangeInt`, `RectInt`, `BoundsInt`) and corresponding formatters. It can enable proper communication between servers and Unity clients.

After installation, extension packages must be enabled, by creating composite resolvers. Here is an example showing how to enable all extensions.

```csharp
// Set extensions to default resolver.
var resolver = MessagePack.Resolvers.CompositeResolver.Create(
    // enable extension packages first
    ReactivePropertyResolver.Instance,
    MessagePack.Unity.Extension.UnityBlitResolver.Instance,
    MessagePack.Unity.UnityResolver.Instance,

    // finally use standard (default) resolver
    StandardResolver.Instance
);
var options = MessagePackSerializerOptions.Standard.WithResolver(resolver);

// Pass options every time or set as default
MessagePackSerializer.DefaultOptions = options;
```

For configuration details, see: [Extension Point section](#resolvers).

The `MessagePack.AspNetCoreMvcFormatter` is add-on for [ASP.NET Core MVC](https://github.com/aspnet/Mvc)'s serialization to boost up performance. This is configuration sample.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().AddMvcOptions(option =>
    {
        option.OutputFormatters.Clear();
        option.OutputFormatters.Add(new MessagePackOutputFormatter(ContractlessStandardResolver.Options));
        option.InputFormatters.Clear();
        option.InputFormatters.Add(new MessagePackInputFormatter(ContractlessStandardResolver.Options));
    });
}
```

Other authors are creating extension packages, too.

* [MagicOnion](https://github.com/Cysharp/MagicOnion) - gRPC based HTTP/2 RPC Streaming Framework
* [MasterMemory](https://github.com/Cysharp/MasterMemory) - Embedded Readonly In-Memory Document Database

You can make your own extension serializers or integrate with frameworks. Let's create and share!

* [MessagePack.FSharpExtensions](https://github.com/pocketberserker/MessagePack.FSharpExtensions) - supports F# list, set, map, unit, option, discriminated union
* [MessagePack.NodaTime](https://github.com/ARKlab/MessagePack) - Support for NodaTime types to MessagePack C#
* [WebApiContrib.Core.Formatter.MessagePack](https://github.com/WebApiContrib/WebAPIContrib.Core#formatters) - supports ASP.NET Core MVC ([details in blog post](https://www.strathweb.com/2017/06/using-messagepack-with-asp-net-core-mvc/))
* [MessagePack.MediaTypeFormatter](https://github.com/sketch7/MessagePack.MediaTypeFormatter) - MessagePack MediaTypeFormatter

## Experimental Features

MessagePack for C# has experimental features which provides you with very performant formatters. There is an official package.

```ps1
Install-Package MessagePack.Experimental
```

For detailed information, see: [Experimental.md](src/MessagePack.Experimental/Experimental.md)

# API

## High-Level API (`MessagePackSerializer`)

The `MessagePackSerializer` class is the entry point of MessagePack for C#. Static methods make up the main API of MessagePack for C#.

| API | Description |
| --- | --- |
| `Serialize<T>` | Serializes an object graph to a MessagePack binary blob. Async variant for Stream available. Non-generic overloads available. |
| `Deserialize<T>` | Deserializes a MessagePack binary to an object graph. Async variant for Stream available. Non-generic overloads available. |
| `SerializeToJson` | Serialize a MessagePack-compatible object graph to JSON instead of MessagePack. Useful for debugging. |
| `ConvertToJson` | Convert MessagePack binary to JSON. Useful for debugging.  |
| `ConvertFromJson` | Convert JSON to a MessagePack binary. |

The `MessagePackSerializer.Typeless` class offers most of the same APIs as above, but removes all type arguments from the API, forcing serialization to include the full type name of the root object. It uses the `TypelessContractlessStandardResolver`. Consider the result to be a .NET-specific MessagePack binary that isn't readily compatible with MessagePack deserializers in other runtimes.

MessagePack for C# fundamentally serializes using `IBufferWriter<byte>` and deserializes using `ReadOnlySequence<byte>` or `Memory<byte>`. Method overloads are provided to conveniently use it with common buffer types and the .NET `Stream` class, but some of these convenience overloads require copying buffers once and therefore have a certain overhead.

The high-level API uses a memory pool internally to avoid unnecessary memory allocation. If result size is under 64K, it allocates GC memory only for the return bytes.

Each serialize/deserialize method takes an optional `MessagePackSerializerOptions` parameter which can be used to specify a custom `IFormatterResolver` to use or to activate LZ4 compression support.

### Multiple MessagePack structures on a single `Stream`

To deserialize a `Stream` that contains multiple consecutive MessagePack data structures,
you can use the `MessagePackStreamReader` class to efficiently identify the `ReadOnlySequence<byte>`
for each data structure and deserialize it. For example:

```cs
static async Task<List<T>> DeserializeListFromStreamAsync<T>(Stream stream, CancellationToken cancellationToken)
{
    var dataStructures = new List<T>();
    using (var streamReader = new MessagePackStreamReader(stream))
    {
        while (await streamReader.ReadAsync(cancellationToken) is ReadOnlySequence<byte> msgpack)
        {
            dataStructures.Add(MessagePackSerializer.Deserialize<T>(msgpack, cancellationToken: cancellationToken));
        }
    }

    return dataStructures;
}
```

## Low-Level API (`IMessagePackFormatter<T>`)

The `IMessagePackFormatter<T>` interface is responsible for serializing a unique type. For example `Int32Formatter : IMessagePackFormatter<Int32>` represents Int32 MessagePack serializer.

```csharp
public interface IMessagePackFormatter<T>
{
    void Serialize(ref MessagePackWriter writer, T value, MessagePackSerializerOptions options);
    T Deserialize(ref MessagePackReader reader, MessagePackSerializerOptions options);
}
```

Many built-in formatters exists under `MessagePack.Formatters`. Your custom types are usually automatically supported with the built-in type resolvers that generate new `IMessagePackFormatter<T>` types on-the-fly using dynamic code generation. See our [AOT code generation](#aot) support for platforms that do not support this.

However, some types - especially those provided by third party libraries or the runtime itself - cannot be appropriately annotated, and contractless serialization would produce inefficient or even wrong results.
To take more control over the serialization of such custom types, write your own `IMessagePackFormatter<T>` implementation.
Here is an example of such a custom formatter implementation. Note its use of the primitive API that is described in the next section.

```csharp
/// <summary>Serializes a <see cref="FileInfo" /> by its full path as a string.</summary>
public class FileInfoFormatter : IMessagePackFormatter<FileInfo>
{
    public void Serialize(
      ref MessagePackWriter writer, FileInfo value, MessagePackSerializerOptions options)
    {
        if (value == null)
        {
            writer.WriteNil();
            return;
        }

        writer.WriteString(value.FullName);
    }

    public FileInfo Deserialize(
      ref MessagePackReader reader, MessagePackSerializerOptions options)
    {
        if (reader.TryReadNil())
        {
            return null;
        }

        options.Security.DepthStep(ref reader);

        var path = reader.ReadString();

        reader.Depth--;
        return new FileInfo(path);
    }
}
```

The `DepthStep` and `Depth--` statements provide a level of security while deserializing untrusted data
that might otherwise be able to execute a denial of service attack by sending MessagePack data that would
deserialize into a very deep object graph leading to a `StackOverflowException` that would crash the process.
This pair of statements should surround the bulk of any `IMessagePackFormatter<T>.Deserialize` method.

**Important**: A message pack formatter must *read or write exactly one data structure*.
In the above example we just read/write a string. If you have more than one element to write out,
you must precede it with a map or array header. You must read the entire map/array when deserializing.
For example:

```csharp
public class MySpecialObjectFormatter : IMessagePackFormatter<MySpecialObject>
{
    public void Serialize(
      ref MessagePackWriter writer, MySpecialObject value, MessagePackSerializerOptions options)
    {
        if (value == null)
        {
            writer.WriteNil();
            return;
        }

        writer.WriteArrayHeader(2);
        writer.WriteString(value.FullName);
        writer.WriteString(value.Age);
    }

    public MySpecialObject Deserialize(
      ref MessagePackReader reader, MessagePackSerializerOptions options)
    {
        if (reader.TryReadNil())
        {
            return null;
        }

        options.Security.DepthStep(ref reader);

        string fullName = null;
        int age = 0;

        // Loop over *all* array elements independently of how many we expect,
        // since if we're serializing an older/newer version of this object it might
        // vary in number of elements that were serialized, but the contract of the formatter
        // is that exactly one data structure must be read, regardless.
        // Alternatively, we could check that the size of the array/map is what we expect
        // and throw if it is not.
        int count = reader.ReadArrayHeader();
        for (int i = 0; i < count; i++)
        {
            switch (i)
            {
                case 0:
                    fullName = reader.ReadString();
                    break;
                case 1:
                    age = reader.ReadInt32();
                    break;
                default:
                    reader.Skip();
                    break;
            }
        }

        reader.Depth--;
        return new MySpecialObject(fullName, age);
    }
}
```

Your custom formatters must be discoverable via some `IFormatterResolver`. Learn more in our [resolvers](#resolvers) section.

You can see many other samples from [builtin formatters](https://github.com/neuecc/MessagePack-CSharp/tree/master/src/MessagePack/Formatters).

## Primitive API (`MessagePackWriter`, `MessagePackReader`)

The `MessagePackWriter` and `MessagePackReader` structs make up the lowest-level API. They read and write the primitives types defined in the MessagePack specification.

### `MessagePackReader`

A `MessagePackReader` can efficiently read from `ReadOnlyMemory<byte>` or `ReadOnlySequence<byte>` without any allocations, except to allocate a new `string` as required by the `ReadString()` method. All other methods return either value structs or `ReadOnlySequence<byte>` slices for extensions/arrays.
Reading directly from `ReadOnlySequence<byte>` means the reader can directly consume some modern high performance APIs such as `PipeReader`.

| Method | Description |
| --- | --- |
| `Skip` | Advances the reader's position past the current value. If the value is complex (e.g. map, array) the entire structure is skipped. |
| `Read*` | Read and return a value whose type is named by the method name from the current reader position. Throws if the expected type does not match the actual type. When reading numbers, the type need not match the binary-specified type exactly. The numeric value will be coerced into the desired type or throw if the integer type is too small for a large value. |
| `TryReadNil` | Advances beyond the current value if the current value is `nil` and returns `true`; otherwise leaves the reader's position unchanged and returns `false`. |
| `ReadBytes` | Returns a slice of the input sequence representing the contents of a `byte[]`, and advances the reader. |
| `ReadStringSequence` | Returns a slice of the input sequence representing the contents of a `string` without decoding it, and advances the reader. |
| `Clone` | Creates a new `MessagePackReader` with the specified input sequence and the same settings as the original reader. |
| `CreatePeekReader` | Creates a new reader with the same position as this one, allowing the caller to "read ahead" without impacting the original reader's position. |
| `NextCode` | Reads the low-level MessagePack `byte` that describes the type of the next value. Does not advance the reader. See [MessagePack format of first byte](https://github.com/msgpack/msgpack/blob/master/spec.md#overview). Its static class has `ToMessagePackType` and `ToFormatName` utility methods. `MessagePackRange` means Min-Max fix range of MessagePack format. |
| `NextMessagePackType` | Describes the `NextCode` value as a higher level category. Does not advance the reader. See [MessagePack spec of source types](https://github.com/msgpack/msgpack/blob/master/spec.md#serialization-type-to-format-conversion). |
| (others) | Other methods and properties as described by the .xml doc comment file and Intellisense. |

The `MessagePackReader` is capable of automatically interpreting both the old and new MessagePack spec.

### `MessagePackWriter`

A `MessagePackWriter` writes to a given instance of `IBufferWriter<byte>`. Several common implementations of this exist, allowing zero allocations and minimal buffer copies while writing directly to several I/O APIs including `PipeWriter`.

The `MessagePackWriter` writes the new MessagePack spec by default, but can write MessagePack compatible with the old spec by setting the `OldSpec` property to `true`.

| Method | Description |
| --- | --- |
| `Clone` | Creates a new `MessagePackWriter` with the specified underlying `IBufferWriter<byte>` and the same settings as the original writer. |
| `Flush` | Writes any buffered bytes to the underlying `IBufferWriter<byte>`. |
| `WriteNil` | Writes the MessagePack equivalent of .NET's `null` value. |
| `Write` | Writes any MessagePack primitive value in the most compact form possible. Has overloads for every primitive type defined by the MessagePack spec. |
| `Write*IntType*` | Writes an integer value in exactly the MessagePack type specified, even if a more compact format exists. |
| `WriteMapHeader` | Introduces a map by specifying the number of key=value pairs it contains. |
| `WriteArrayHeader` | Introduces an array by specifying the number of elements it contains. |
| `WriteExtensionFormat` | Writes the full content of an extension value including length, type code and content. |
| `WriteExtensionFormatHeader` | Writes just the header (length and type code) of an extension value. |
| `WriteRaw` | Copies the specified bytes directly to the underlying `IBufferWriter<byte>` without any validation. |
| (others) | Other methods and properties as described by the .xml doc comment file and Intellisense. |

`DateTime` is serialized to [MessagePack Timestamp format](https://github.com/msgpack/msgpack/blob/master/spec.md#formats-timestamp), it serialize/deserialize UTC and loses `Kind` info and requires that `MessagePackWriter.OldSpec == false`.
If you use the `NativeDateTimeResolver`, `DateTime` values will be serialized using .NET's native `Int64` representation, which preserves `Kind` info but may not be interoperable with non-.NET platforms.

## <a name="resolvers"></a>Main Extension Point (`IFormatterResolver`)

An `IFormatterResolver` is storage of typed serializers. The `MessagePackSerializer` API accepts a `MessagePackSerializerOptions` object which specifies the `IFormatterResolver` to use, allowing customization of the serialization of complex types.

| Resolver Name | Description |
| --- | --- |
| BuiltinResolver | Builtin primitive and standard classes resolver. It includes primitive(int, bool, string...) and there nullable, array and list. and some extra builtin types(`Guid`, `Uri`, `BigInteger`, etc...). |
| StandardResolver | Composited resolver. It resolves in the following order `builtin -> attribute -> dynamic enum -> dynamic generic -> dynamic union -> dynamic object -> dynamic object fallback`. This is the default of MessagePackSerializer. |
| ContractlessStandardResolver | Composited `StandardResolver`(except dynamic object fallback) -> `DynamicContractlessObjectResolver` -> `DynamicObjectTypeFallbackResolver`. It enables contractless serialization. |
| StandardResolverAllowPrivate | Same as StandardResolver but allow serialize/deserialize private members. |
| ContractlessStandardResolverAllowPrivate | Same as ContractlessStandardResolver but allow serialize/deserialize private members. |
| PrimitiveObjectResolver | MessagePack primitive object resolver. It is used fallback in `object` type and supports `bool`, `char`, `sbyte`, `byte`, `short`, `int`, `long`, `ushort`, `uint`, `ulong`, `float`, `double`, `DateTime`, `string`, `byte[]`, `ICollection`, `IDictionary`. |
| DynamicObjectTypeFallbackResolver | Serialize is used type in from `object` type, deserialize is used PrimitiveObjectResolver. |
| AttributeFormatterResolver | Get formatter from `[MessagePackFormatter]` attribute. |
| CompositeResolver | Composes several resolvers and/or formatters together in an ordered list, allowing reuse and overriding of behaviors of existing resolvers and formatters. |
| NativeDateTimeResolver | Serialize by .NET native DateTime binary format. It keeps `DateTime.Kind` that loses by standard(MessagePack timestamp) format. |
| NativeGuidResolver | Serialize by .NET native Guid binary representation. It is faster than standard(string) representation. |
| NativeDecimalResolver | Serialize by .NET native decimal binary representation. It is faster than standard(string) representation. |
| DynamicEnumResolver | Resolver of enum and there nullable, serialize there underlying type. It uses dynamic code generation to avoid boxing and boostup performance serialize there name. |
| DynamicEnumAsStringResolver | Resolver of enum and there nullable.  It uses reflection call for resolve nullable at first time. |
| DynamicGenericResolver | Resolver of generic type(`Tuple<>`, `List<>`, `Dictionary<,>`, `Array`, etc). It uses reflection call for resolve generic argument at first time. |
| DynamicUnionResolver | Resolver of interface marked by UnionAttribute. It uses dynamic code generation to create dynamic formatter. |
| DynamicObjectResolver | Resolver of class and struct made by MessagePackObjectAttribute. It uses dynamic code generation to create dynamic formatter. |
| DynamicContractlessObjectResolver | Resolver of all classes and structs. It does not needs `MessagePackObjectAttribute` and serialized key as string(same as marked `[MessagePackObject(true)]`). |
| DynamicObjectResolverAllowPrivate | Same as DynamicObjectResolver but allow serialize/deserialize private members. |
| DynamicContractlessObjectResolverAllowPrivate | Same as DynamicContractlessObjectResolver but allow serialize/deserialize private members. |
| TypelessObjectResolver | Used for `object`, embed .NET type in binary by `ext(100)` format so no need to pass type in deserialization.  |
| TypelessContractlessStandardResolver | Composited resolver. It resolves in the following order `nativedatetime -> builtin -> attribute -> dynamic enum -> dynamic generic -> dynamic union -> dynamic object -> dynamiccontractless -> typeless`. This is the default of `MessagePackSerializer.Typeless`  |

Each instance of `MessagePackSerializer` accepts only a single resolver. Most object graphs will need more than one for serialization, so composing a single resolver made up of several is often required, and can be done with the `CompositeResolver` as shown below:

```csharp
// Do this once and store it for reuse.
var resolver = MessagePack.Resolvers.CompositeResolver.Create(
    // resolver custom types first
    ReactivePropertyResolver.Instance,
    MessagePack.Unity.Extension.UnityBlitResolver.Instance,
    MessagePack.Unity.UnityResolver.Instance,

    // finally use standard resolver
    StandardResolver.Instance
);
var options = MessagePackSerializerOptions.Standard.WithResolver(resolver);

// Each time you serialize/deserialize, specify the options:
byte[] msgpackBytes = MessagePackSerializer.Serialize(myObject, options);
T myObject2 = MessagePackSerializer.Deserialize<MyObject>(msgpackBytes, options);
```

A resolver can be set as default with `MessagePackSerializer.DefaultOptions = options`, but **WARNING**:
When developing an application where you control all MessagePack-related code it may be safe to rely on this mutable static to control behavior.
For all other libraries or multi-purpose applications that use `MessagePackSerializer` you should explicitly specify the `MessagePackSerializerOptions` to use with each method invocation to guarantee your code behaves as you expect even when sharing an `AppDomain` or process with other MessagePack users that may change this static property.

Here is sample of use `DynamicEnumAsStringResolver` with `DynamicContractlessObjectResolver` (It is Json.NET-like lightweight setting.)

```csharp
// composite same as StandardResolver
var resolver = MessagePack.Resolvers.CompositeResolver.Create(
    MessagePack.Resolvers.BuiltinResolver.Instance,
    MessagePack.Resolvers.AttributeFormatterResolver.Instance,

    // replace enum resolver
    MessagePack.Resolvers.DynamicEnumAsStringResolver.Instance,

    MessagePack.Resolvers.DynamicGenericResolver.Instance,
    MessagePack.Resolvers.DynamicUnionResolver.Instance,
    MessagePack.Resolvers.DynamicObjectResolver.Instance,

    MessagePack.Resolvers.PrimitiveObjectResolver.Instance,

    // final fallback(last priority)
    MessagePack.Resolvers.DynamicContractlessObjectResolver.Instance
);
```

If you want to make an extension package, you should write both a formatter and resolver
for easier consumption.
Here is sample of a resolver:

```csharp
public class SampleCustomResolver : IFormatterResolver
{
    // Resolver should be singleton.
    public static readonly IFormatterResolver Instance = new SampleCustomResolver();

    private SampleCustomResolver()
    {
    }

    // GetFormatter<T>'s get cost should be minimized so use type cache.
    public IMessagePackFormatter<T> GetFormatter<T>()
    {
        return FormatterCache<T>.Formatter;
    }

    private static class FormatterCache<T>
    {
        public static readonly IMessagePackFormatter<T> Formatter;

        // generic's static constructor should be minimized for reduce type generation size!
        // use outer helper method.
        static FormatterCache()
        {
            Formatter = (IMessagePackFormatter<T>)SampleCustomResolverGetFormatterHelper.GetFormatter(typeof(T));
        }
    }
}

internal static class SampleCustomResolverGetFormatterHelper
{
    // If type is concrete type, use type-formatter map
    static readonly Dictionary<Type, object> formatterMap = new Dictionary<Type, object>()
    {
        {typeof(FileInfo), new FileInfoFormatter()}
        // add more your own custom serializers.
    };

    internal static object GetFormatter(Type t)
    {
        object formatter;
        if (formatterMap.TryGetValue(t, out formatter))
        {
            return formatter;
        }

        // If type can not get, must return null for fallback mechanism.
        return null;
    }
}
```

## MessagePackFormatterAttribute

MessagePackFormatterAttribute is a lightweight extension point of class, struct, interface, enum and property/field. This is like Json.NET's JsonConverterAttribute. For example, serialize private field, serialize x10 formatter.

```csharp
[MessagePackFormatter(typeof(CustomObjectFormatter))]
public class CustomObject
{
    string internalId;

    public CustomObject()
    {
        this.internalId = Guid.NewGuid().ToString();
    }

    // serialize/deserialize internal field.
    class CustomObjectFormatter : IMessagePackFormatter<CustomObject>
    {
        public void Serialize(ref MessagePackWriter writer, CustomObject value, MessagePackSerializerOptions options)
        {
            options.Resolver.GetFormatterWithVerify<string>().Serialize(ref writer, value.internalId, options);
        }

        public CustomObject Deserialize(ref MessagePackReader reader, MessagePackSerializerOptions options)
        {
            var id = options.Resolver.GetFormatterWithVerify<string>().Deserialize(ref reader, options);
            return new CustomObject { internalId = id };
        }
    }
}

// per field, member

public class Int_x10Formatter : IMessagePackFormatter<int>
{
    public int Deserialize(ref MessagePackReader reader, MessagePackSerializerOptions options)
    {
        return reader.ReadInt32() * 10;
    }

    public void Serialize(ref MessagePackWriter writer, int value, MessagePackSerializerOptions options)
    {
        writer.WriteInt32(value * 10);
    }
}

[MessagePackObject]
public class MyClass
{
    // You can attach custom formatter per member.
    [Key(0)]
    [MessagePackFormatter(typeof(Int_x10Formatter))]
    public int MyProperty1 { get; set; }
}
```

Formatter is retrieved by `AttributeFormatterResolver`, it is included in `StandardResolver`.

## IgnoreFormatter

`IgnoreFormatter<T>` is lightweight extension point of class and struct. If there exists types that can't be serialized, you can register `IgnoreFormatter<T>` that serializes those to nil/null.

```csharp
// CompositeResolver can set custom formatter.
var resolver = MessagePack.Resolvers.CompositeResolver.Create(
    new IMessagePackFormatter[]
    {
        // for example, register reflection infos (can not serialize)
        new IgnoreFormatter<MethodBase>(),
        new IgnoreFormatter<MethodInfo>(),
        new IgnoreFormatter<PropertyInfo>(),
        new IgnoreFormatter<FieldInfo>()
    },
    new IFormatterResolver[]
    {
        ContractlessStandardResolver.Instance
    });
```

## Reserved Extension Types

MessagePack for C# already used some MessagePack extension type codes, be careful to use same ext code.

| Code | Type | Use by |
| ---  | ---  | --- |
| -1 | DateTime | MessagePack-spec reserved for timestamp |
| 30 | Vector2[] | for Unity, UnsafeBlitFormatter |
| 31 | Vector3[] | for Unity, UnsafeBlitFormatter |
| 32 | Vector4[] | for Unity, UnsafeBlitFormatter |
| 33 | Quaternion[] | for Unity, UnsafeBlitFormatter |
| 34 | Color[] | for Unity, UnsafeBlitFormatter |
| 35 | Bounds[] | for Unity, UnsafeBlitFormatter |
| 36 | Rect[] | for Unity, UnsafeBlitFormatter |
| 37 | Int[] | for Unity, UnsafeBlitFormatter |
| 38 | Float[] | for Unity, UnsafeBlitFormatter |
| 39 | Double[] | for Unity, UnsafeBlitFormatter |
| 98 | All | MessagePackCompression.Lz4BlockArray |
| 99 | All | MessagePackCompression.Lz4Block |
| 100 | object | TypelessFormatter |

## Unity support

Unity lowest supported version is `2018.3`, API Compatibility Level supports both `.NET 4.x` and `.NET Standard 2.0`.

You can install the `unitypackage` from the [releases][Releases] page.
If your build targets .NET Framework 4.x and runs on mono, you can use it as is.
But if your build targets IL2CPP, you can not use `Dynamic***Resolver`, so it is required to use pre-code generation. Please see [pre-code generation section](#aot).

MessagePack for C# includes some additional `System.*.dll` libraries that originally provides in NuGet. They are located under `Plugins`. If other packages use these libraries (e.g. Unity Collections package using `System.Runtime.CompilerServices.Unsafe.dll`), to avoid conflicts, please delete the DLL under `Plugins`.

Currently `CompositeResolver.Create` does not work on IL2CPP, so it is recommended to use `StaticCompositeResolver.Instance.Register` instead.

In Unity, MessagePackSerializer can serialize `Vector2`, `Vector3`, `Vector4`, `Quaternion`, `Color`, `Bounds`, `Rect`, `AnimationCurve`, `Keyframe`, `Matrix4x4`, `Gradient`, `Color32`, `RectOffset`, `LayerMask`, `Vector2Int`, `Vector3Int`, `RangeInt`, `RectInt`, `BoundsInt` and their nullable, array and list types with the built-in extension `UnityResolver`. It is included in StandardResolver by default.

MessagePack for C# has an additional unsafe extension.  `UnsafeBlitResolver` is special resolver for extremely fast but unsafe serialization/deserialization of struct arrays.

![image](https://cloud.githubusercontent.com/assets/46207/23837633/76589924-07ce-11e7-8b26-e50eab548938.png)

x20 faster Vector3[] serialization than native JsonUtility. If use `UnsafeBlitResolver`, serialization uses a special format (ext:typecode 30~39)  for `Vector2[]`, `Vector3[]`, `Quaternion[]`, `Color[]`, `Bounds[]`, `Rect[]`. If use `UnityBlitWithPrimitiveArrayResolver`, it supports `int[]`, `float[]`, `double[]` too. This special feature is useful for serializing Mesh (many `Vector3[]`) or many transform positions.

If you want to use unsafe resolver, register `UnityBlitResolver` or `UnityBlitWithPrimitiveArrayResolver`.

Here is sample of configuration.

```csharp
StaticCompositeResolver.Instance.Register(
    MessagePack.Unity.UnityResolver.Instance,
    MessagePack.Unity.Extension.UnityBlitWithPrimitiveArrayResolver.Instance,
    MessagePack.Resolvers.StandardResolver.Instance
);

var options = MessagePackSerializerOptions.Standard.WithResolver(StaticCompositeResolver.Instance);
MessagePackSerializer.DefaultOptions = options;
```

The `MessagePack.UnityShims` NuGet package is for .NET server-side serialization support to communicate with Unity. It includes shims for Vector3 etc and the Safe/Unsafe serialization extension.

If you want to share a class between Unity and a server, you can use `SharedProject` or `Reference as Link` or a glob reference (with `LinkBase`), etc. Anyway, you need to share at source-code level. This is a sample project structure using a glob reference (recommended).

- ServerProject(.NET 4.6/.NET Core/.NET Standard)
  - \[`<Compile Include="..\UnityProject\Assets\Scripts\Shared\**\*.cs" LinkBase="Shared" />`\]
  - \[MessagePack\]
  - \[MessagePack.UnityShims\]
- UnityProject
  - \[Concrete SharedCodes\]
  - \[MessagePack\](not dll/NuGet, use MessagePack.Unity.unitypackage's sourcecode)

## <a name="aot"></a>AOT Code Generation (support for Unity/Xamarin)

By default, MessagePack for C# serializes custom objects by [generating IL](https://msdn.microsoft.com/en-us/library/system.reflection.emit.ilgenerator.aspx) on the fly at runtime to create custom, highly tuned formatters for each type. This code generation has a minor upfront performance cost.
Because strict-AOT environments such as Xamarin and Unity IL2CPP forbid runtime code generation, MessagePack provides a way for you to run a code generator ahead of time as well.

> Note: When using Unity, dynamic code generation only works when targeting .NET Framework 4.x + mono runtime.
For all other Unity targets, AOT is required.

If you want to avoid the upfront dynamic generation cost or you need to run on Xamarin or Unity, you need AOT code generation. `mpc` (MessagePackCompiler) is the code generator of MessagePack for C#. mpc uses [Roslyn](https://github.com/dotnet/roslyn) to analyze source code.

First of all, mpc requires [.NET Core 3 Runtime](https://dotnet.microsoft.com/download). The easiest way to acquire and run mpc is as a dotnet tool.

```
dotnet tool install --global MessagePack.Generator
```

Installing it as a local tool allows you to include the tools and versions that you use in your source control system. Run these commands in the root of your repo:

```
dotnet new tool-manifest
dotnet tool install MessagePack.Generator
```

Check in your `.config\dotnet-tools.json` file. On another machine you can "restore" your tool using the `dotnet tool restore` command.

Once you have the tool installed, simply invoke using `dotnet mpc` within your repo:

```
dotnet mpc --help
```

Alternatively, you can download mpc from the [releases][Releases] page, that includes platform native binaries (that don't require a separate dotnet runtime).

```
Usage: mpc [options...]

Options:
  -i, -input <String>                                Input path to MSBuild project file or the directory containing Unity source files. (Required)
  -o, -output <String>                               Output file path(.cs) or directory(multiple generate file). (Required)
  -c, -conditionalSymbol <String>                    Conditional compiler symbols, split with ','. (Default: null)
  -r, -resolverName <String>                         Set resolver name. (Default: GeneratedResolver)
  -n, -namespace <String>                            Set namespace root name. (Default: MessagePack)
  -m, -useMapMode <Boolean>                          Force use map mode serialization. (Default: False)
  -ms, -multipleIfDirectiveOutputSymbols <String>    Generate #if-- files by symbols, split with ','. (Default: null)
```

`mpc` targets C# code with `[MessagePackObject]` or `[Union]` annotations.

```cmd
// Simple Sample:
dotnet mpc -i "..\src\Sandbox.Shared.csproj" -o "MessagePackGenerated.cs"

// Use force map simulate DynamicContractlessObjectResolver
dotnet mpc -i "..\src\Sandbox.Shared.csproj" -o "MessagePackGenerated.cs" -m
```

By default, `mpc` generates the resolver as `MessagePack.Resolvers.GeneratedResolver` and formatters as`MessagePack.Formatters.*`.

Here is the full sample code to register a generated resolver in Unity.

```csharp
using MessagePack;
using MessagePack.Resolvers;
using UnityEngine;

public class Startup
{
    static bool serializerRegistered = false;

    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
    static void Initialize()
    {
        if (!serializerRegistered)
        {
            StaticCompositeResolver.Instance.Register(
                 MessagePack.Resolvers.GeneratedResolver.Instance,
                 MessagePack.Resolvers.StandardResolver.Instance
            );

            var option = MessagePackSerializerOptions.Standard.WithResolver(StaticCompositeResolver.Instance);

            MessagePackSerializer.DefaultOptions = option;
            serializerRegistered = true;
        }
    }

#if UNITY_EDITOR


    [UnityEditor.InitializeOnLoadMethod]
    static void EditorInitialize()
    {
        Initialize();
    }

#endif
}
```

In Unity, you can use MessagePack CodeGen windows at `Windows -> MessagePack -> CodeGenerator`.

![](https://user-images.githubusercontent.com/46207/69414381-f14da400-0d55-11ea-9f8d-9af448d347dc.png)

Install the .NET Core runtime, install mpc (as a .NET Core Tool as described above), and execute `dotnet mpc`. Currently this tool is experimental so please tell me your opinion.

In Xamarin, you can install the [the `MessagePack.MSBuild.Tasks` NuGet package](doc/msbuildtask.md) into your projects to pre-compile fast serialization code and run in environments where JIT compilation is not allowed.

## RPC

MessagePack advocated [MessagePack RPC](https://github.com/msgpack-rpc/msgpack-rpc), but work on it has stopped and it is not widely used.

### MagicOnion

I've created a gRPC based MessagePack HTTP/2 RPC streaming framework called [MagicOnion](https://github.com/Cysharp/MagicOnion). gRPC usually communicates with Protocol Buffers using IDL. But MagicOnion uses MessagePack for C# and does not need IDL. When communicating C# to C#, schemaless (or rather C# classes as schema) is better than using IDL.

### StreamJsonRpc

The StreamJsonRpc library is based on [JSON-RPC](https://www.jsonrpc.org/) and includes [a pluggable formatter architecture](https://github.com/microsoft/vs-streamjsonrpc/blob/master/doc/extensibility.md#alternative-formatters) and as of v2.3 includes [MessagePack support](https://github.com/microsoft/vs-streamjsonrpc/blob/master/doc/extensibility.md#message-formatterss).

## How to build

See our [contributor's guide](CONTRIBUTING.md).

## Author Info

Yoshifumi Kawai (a.k.a. neuecc) is a software developer in Japan.
He is the Director/CTO at Grani, Inc.
Grani is a mobile game developer company in Japan and well known for using C#.
He is awarding Microsoft MVP for Visual C# since 2011.
He is known as the creator of [UniRx](https://github.com/neuecc/UniRx/) (Reactive Extensions for Unity)

* Blog: [https://medium.com/@neuecc](https://medium.com/@neuecc) (English)
* Blog: [http://neue.cc/](http://neue.cc/) (Japanese)
* Twitter: [https://twitter.com/neuecc](https://twitter.com/neuecc) (Japanese)

[Releases]: https://github.com/neuecc/MessagePack-CSharp/releases
