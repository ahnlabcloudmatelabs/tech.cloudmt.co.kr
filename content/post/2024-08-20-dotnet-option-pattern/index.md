---
title: .NET Options pattern
authors:
- sanghun-han # 저자 프로필 페이지 경로 입력
date: 2024-08-20T09:05:32Z
categories:
- Tech # 엔지니어가 아니여도 쉽게 읽을 수 있는 글
tags:
- .NET
- Configuration
- OptionPattern
#feature_image: 'images/cover.png' # 포스트 커버 이미지 경로 (포스트에 포함된 이미지 중 하나 지정. 필드 제거하면 기본 이미지가 나옵니다.)
ShowToc: false # 글 개요 보여줄지 여부
TocOpen: false # 글 개요를 보여주는 경우, 펼처서 보여줄지 여부.
draft: false # 초안 작성 모드. true 설정시 커밋해도 나오지 않습니다.
---

하나의 어플리케이션이 호스트 되는 환경은 다양하게 구성(Configuration)될 수 있습니다. 흔한 예로 개발 환경과 운영 환경이 나눠지는 경우가 있습니다. 그리고 분산된 어플리케이션들과 통신하기 위해 각 어플리케이션에 대한 호스팅 정보를 구성하는데 사용되기도 합니다. 또한 구성 요소를 통해 서버의 설정이나 로거의 메시지 출력 수준을 조정하는 등 모듈의 동작 수준을 설정할 수 있습니다. 

이러한 경우, 코드와 구성 요소를 분리하여 코드 변경 없이 동작 수준을 조정할 수 있는 방법을 고려해야 합니다.

.NET에서는 다양한 구성 공급자를 제공하고 통합된 보기 환경을 위해 추상화된 인터페이스를 제공합니다.([Configuration - .NET | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration))

이번 글에서는 클래스를 이용하여 구성 요소에 대한 강력한 형식의 접근을 제공하는 옵션 패턴에 대해 설명합니다.

옵션 패턴을 통해 애플리케이션에서 사용하는 모든 구성 요소는 하나의 거대한 클래스가 아닌, 사용 목적에 따라 별개의 세부 클래스로 분리할 수 있습니다. 이를 통해 각 설정은 다른 설정에 종속되거나 결합되지 않으며, 설정을 사용하는 클래스는 자신이 필요로 하는 설정에만 의존하게 됩니다.

글에서 사용한 코드는 [여기](https://github.com/lifecycle67/OptionPatternExample)를 참조 바랍니다.

# 구성 사용하기

json 파일을 구성 요소로 사용하고 옵션 패턴을 통해 값을 읽는 방법을 알아보겠습니다.

먼저 사용할 json 파일은 다음과 같습니다.

```json
{
  "TestSettingValue": "just string value",
  "CustomConfigurationOptions": {
    "Deadline": "00:00:30",
    "Enabled": true,
    "Retry": 2,
    "Level": "Warning"
  }
}
```

옵션 패턴은 IOpions<TOpions> 형식으로 제공됩니다. (IOpions 외에 다른 인터페이스 형식도 사용할 수 있습니다. 다른 인터페이스 형식에 대해서는 [여기](https://www.notion.so/4c13aee3dc6141c89e36fdbc994d1971?pvs=21)에서 설명합니다.)  CustomConfigurationOptions의 값을 사용하는 시나리오를 생각해봅시다. 먼저 CustomConfigurationOptions 클래스를 만듭니다.

```csharp
public class CustomConfigurationOptions
{
    public TimeSpan Deadline { get; set; }
    public bool Enabled { get; set; }
    public int Retry { get; set; }
    public string Level { get; set; }
}
```

옵션 패턴에 사용되는 클래스는 다음과 같은 제약 사항을 따라야 합니다

- 인자 없는 퍼블릭 생성자를 사용하는 비 추상 클래스입니다
- 읽기 쓰기 가능한 퍼블릭 속성을 가지고 있어야 합니다.

json 파일에 설정된 구성 요소를 불러들이고 클래스를 통해 값을 읽습니다. program.cs에 다음 코드를 추가합니다.

```csharp
HostApplicationBuilder builder = new HostApplicationBuilder();
///builder.Configuration.Sources.Clear()를 통해 기본값으로 추가된 구성 공급자를 제거합니다
///기본 구성 공급자 목록은 https://tinyurl.com/apu8ux35 페이지에서 remark 항목을 참조합니다
builder.Configuration.Sources.Clear(); 
builder.Configuration.AddJsonFile("appsettings.json");

//appsettings.json의 구성 요소 중 CustomConfigurationOptions 요소를 가져옵니다
var options = builder.Configuration.GetSection(nameof(CustomConfigurationOptions))
                                   .Get<CustomConfigurationOptions>();

Console.WriteLine($"CustomConfigurationOptions.Deadline:{options.Deadline}");
Console.WriteLine($"CustomConfigurationOptions.Enabled:{options.Enabled}");
```

<aside>
💡 예제 코드에서는 HostApplicationBuilder의 Configuration 속성을 통해 json파일을 읽고 옵션 인터페이스를 사용했습니다.
다른 방법으로는 ConfigurationBuilder를 사용할 수 있습니다. ConfigurationBuilder의 경우 Build 메서드를 통해 [IConfigurationRoot](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.iconfigurationroot?view=net-8.0)를 가져올 수 있습니다. HostApplicationBuilder.Configuration의 형식은 [ConfigurationManager](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.configurationmanager?view=net-8.0&devlangs=csharp&f1url=%3FappId%3DDev17IDEF1%26l%3DEN-US%26k%3Dk(Microsoft.Extensions.Configuration.ConfigurationManager)%3Bk(DevLang-csharp)%26rd%3Dtrue)입니다. 이 클래스는 IConfigurationBuilder이자 IConfigurationRoot입니다. ConfigurationBuilder는 Build 메서드를 명시적으로 호출하는 반면, ConfigurationManager는 구성 공급자가 추가되거나 삭제될 때 내부적으로 Build를 호출한다는 차이점이 있습니다.
이 둘을 비교하는 좀 더 자세한 내용은 [.NET Core 탐색 1부 - .NET 6의 ConfigurationManager 내부 보기 | Andrew Lock - 👨‍🏫 튜토리얼, 팁, 강좌 - 닷넷데브 (dotnetdev.kr)](https://forum.dotnetdev.kr/t/net-core-1-net-6-configurationmanager-andrew-lock/2180) 이 글을 참조 바랍니다.

</aside>

# 종속성 주입

종속성 주입 컨테이너를 통해 옵션 패턴을 사용할 수도 있습니다. 옵션 클래스를 종속성 주입 컨테이너에 등록하기 위해서 [Configure](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.optionsconfigurationservicecollectionextensions.configure?view=net-8.0) 메서드를 사용합니다. program.cs의 코드를 작성합니다

```csharp
HostApplicationBuilder builder = new HostApplicationBuilder();
builder.Configuration.Sources.Clear(); 
builder.Configuration.AddJsonFile("appsettings.json");

var options = builder.Configuration.GetSection(nameof(CustomConfigurationOptions))
                                   .Get<CustomConfigurationOptions>();
                                   
//CustomConfigurationOptions 요소를 종속성 주입 컨테이너에 등록합니다
builder.Services.Configure<CustomConfigurationOptions>(
    builder.Configuration.GetSection(nameof(CustomConfigurationOptions)));

//종속성 주입 컨테이너에 TestService 클래스를 등록합니다
builder.Services.AddTransient<TestService>();

///TestService 클래스에 CustomConfigurationOptions이 주입됨을 확인하기 위해,
///TestService의 종속성을 해결합니다.
var serviceProvider = builder.Services.BuildServiceProvider();
var testService = serviceProvider.GetRequiredService<TestService>();

var host = builder.Build();
host.Run();
```

TestService 클래스 코드는 다음과 같습니다

```csharp
public class TestService
{
    private readonly CustomConfigurationOptions _options;
    public TestService(IOptions<CustomConfigurationOptions> options)
    {
        _options = options.Value;

        Console.WriteLine($"CustomConfigurationOptions.Deadline:{_options.Deadline}");
        Console.WriteLine($"CustomConfigurationOptions.Enabled:{_options.Enabled}");
    }
}
```

종속성 주입 컨테이너를 통해 TestService에 CustomConfigurationOptions가 제공됨을 확인할 수 있습니다. TestService는 이제 CustomConfigurationOptions 형식을 통해 필요한 구성 요소를 명시적으로 주입받습니다. 이로 인해 개발자는 다른 구성 요소를 고려할 필요 없이 필요한 구성만을 사용하여 TestService의 로직을 작성할 수 있습니다. 이는 코드의 명확성과 유지보수성을 크게 향상시킵니다.

# Options interfaces

옵션 패턴에서는 세 가지 인터페이스를 사용할 수 있습니다. 지금까지 코드에서 사용한 IOptions<TOptions> 외에도 IOptionsSnapshot<TOptions>와 IOptionsMonitor<TOptions>가 있습니다. 각 형식의 특징은 다음과 같습니다.

- IOptions<TOptions>
    - 어플리케이션이 동작한 이후에는 구성 데이터를 읽을 수 없습니다.
    - 명명된 옵션(named options)을 지원하지 않습니다.
    - 싱글톤으로 등록되어 대상 서비스의 수명 주기에 관계 없이 주입할 수 있습니다. (DI에서 개체의 수명주기에 관한 주의 사항은 [Captive dependency](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines#captive-dependency) 항목을 참조바랍니다)
- IOptionsSnapshot<TOptions>
    - 재 계산이 필요한 옵션이 scope나 transient 수명 주기를 가진 서비스에 주입될 때 유용합니다.
    - Scoped로 등록됩니다. 따라서 Singleton 서비스에서는 주입될  수 없습니다.
    - 명명된 옵션(named options)을 지원합니다.
- IOptionsMonitor<TOptions>
    - 옵션을 검색하거나 옵션 알림을 관리하는데 사용됩니다.
    - 싱글톤으로 등록되어 대상 서비스의 수명 주기에 관계 없이 주입할 수 있습니다.
    - 변경 알림, 명명된 옵션, 구성 다시 불러오기, 선택적 옵션 무효화 등을 지원합니다.

# 런타임에서 구성 값 갱신

IOptionSnapshot<TOptions>을 사용하면 애플리케이션 실행 후에도 구성 변경 사항을 반영할 수 있습니다. 이 인터페이스는 서비스에 주입될 때 TOptions 인스턴스를 생성하고 옵션을 바인딩하며, 생성된 인스턴스는 해당 scope의 수명 동안 유지됩니다. 요청 시 변경된 옵션 값을 반영해 새로운 인스턴스를 생성하므로, 최신 구성을 주입할 수 있습니다. 그러나 서로 다른 Scope에서 요청이 자주 발생할  경우, 인스턴스 생성과 옵션 바인딩에 따른 성능 비용을 고려해야 합니다. (https://github.com/dotnet/runtime/issues/36130) IOptionSnapshot<TOptions>의 Scoped 수명 주기를 가지므로, 이를 주입 받는 서비스 역시 Scoped 또는 Transient 수명 주기를 가져야 합니다.

IOptionSnapshot을 사용하는 ScopedService 클래스를 추가합니다

```csharp
public class ScopedService
{
    private readonly CustomConfigurationOptions _options;

    public ScopedService(IOptionsSnapshot<CustomConfigurationOptions> options)
    {
        _options = options.Value;

        Console.WriteLine($"IOptionsSnapshot CustomConfigurationOptions.Deadline:{_options.Deadline}");
        Console.WriteLine($"IOptionsSnapshot CustomConfigurationOptions.Enabled:{_options.Enabled}");
        Console.WriteLine($"IOptionsSnapshot CustomConfigurationOptions.Retry:{_options.Retry}");
        Console.WriteLine($"IOptionsSnapshot CustomConfigurationOptions.Level:{_options.Level}");
    }
}
```

program.cs에 다음 코드를 추가합니다

```csharp
//생략됨..

builder.Services.AddScoped<ScopedService>();

//scope 범위를 지정해서 IOptionsSnapshot 인스턴스를 다시 생성합니다
using (var scope = serviceProvider.CreateScope())
{
    scope.ServiceProvider.GetRequiredService<ScopedService>();
}

Console.Write("appsettings.json에서 CustomConfigurationOptions의 값을 변경 후 enter를 입력합니다 : ");
Console.ReadLine();

//scope 범위를 지정해서 IOptionsSnapshot 인스턴스를 다시 생성합니다
using (var scope = serviceProvider.CreateScope())
{
    scope.ServiceProvider.GetRequiredService<ScopedService>();
}
//IOption 의 경우 구성 값이 변경되었는지 확인합니다
serviceProvider.GetRequiredService<TestService>();

//생략됨..
```

appsettings.json 파일의 내용이 수정되었을 때 자동으로 다시 읽기 위해, program.cs에서 AddJsonFile 메서드 호출 시 추가 인자를 설정합니다.

```csharp
~~builder.Configuration.AddJsonFile("appsettings.json");~~
builder.Configuration.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
```

optional 매개변수는 true나 false 어느 값으로 설정해도 무방합니다. 중요한 것은 reloadOnChange 매개변수를 true로 설정하는 것입니다. 이렇게 하면 파일 변경 시 자동으로 구성을 다시 로드합니다.

결과를 확인합니다.

```
IOptionsSnapshot CustomConfigurationOptions.Deadline:00:00:30
IOptionsSnapshot CustomConfigurationOptions.Enabled:True
IOptionsSnapshot CustomConfigurationOptions.Retry:2
IOptionsSnapshot CustomConfigurationOptions.Level:Warning

appsettings.json에서 CustomConfigurationOptions의 값을 변경 후 enter를 입력합니다 :

IOptionsSnapshot CustomConfigurationOptions.Deadline:00:01:30
IOptionsSnapshot CustomConfigurationOptions.Enabled:False
IOptionsSnapshot CustomConfigurationOptions.Retry:5
IOptionsSnapshot CustomConfigurationOptions.Level:Information

IOptions CustomConfigurationOptions.Deadline:00:00:30
IOptions CustomConfigurationOptions.Enabled:True
```

IOptionsMonitor<TOptions>를 사용해도 구성 변경 사항을 실시간으로 반영할 수 있습니다. 앞서 종속성 주입을 설명할 때, 옵션 인터페이스를 종속성 주입 컨테이너에 등록하기 위해 IServiceCollection.Configure 메서드를 호출했습니다. 이 메서드는 TOptions 인스턴스 변경 통지를 위한 [IChangeToken](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.primitives.ichangetoken)을 가져오는데 사용하는 [IOptionsChangeTokenSource<TOptions>](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.options.ioptionschangetokensource-1)도 같이 컨테이너에 등록합니다. IChangeToken을 통해 구성 파일 변경 시 IOptionsMonitor 는 내부 캐쉬의 값을 변경합니다. 

IOptionsMonitor<TOptions>를 사용하는 MonitorService 클래스를 추가합니다.

```csharp
public class MonitorService
{
    private readonly IOptionsMonitor<CustomConfigurationOptions> _options;

    public MonitorService(IOptionsMonitor<CustomConfigurationOptions> options)
    {
        _options = options;
    }

    public void DisplayOption()
    {
        Console.WriteLine($"IOptionsMonitor CustomConfigurationOptions.Deadline:{_options.CurrentValue.Deadline}");
        Console.WriteLine($"IOptionsMonitor CustomConfigurationOptions.Enabled:{_options.CurrentValue.Enabled}");
        Console.WriteLine($"IOptionsMonitor CustomConfigurationOptions.Retry:{_options.CurrentValue.Retry}");
        Console.WriteLine($"IOptionsMonitor CustomConfigurationOptions.Level:{_options.CurrentValue.Level}");
    }
}
```

program.cs에 IOptionsMonitorService 메서드를 추가한 후, Main 메서드에서 이를 호출합니다.

```csharp
private static void IOptionsMonitorService(ServiceProvider serviceProvider)
{
    var monitorService = serviceProvider.GetRequiredService<MonitorService>();
    monitorService.DisplayOption();

    Console.Write("appsettings.json에서 CustomConfigurationOptions의 값을 변경 후 enter를 입력합니다 : ");
    Console.ReadLine();

    monitorService.DisplayOption();
}
```

 

출력 값을 확인합니다. 동일한 monitorService 인스턴스로 구성 파일의 변경된 내용을 표시할 수 있습니다.

```
IOptionsMonitor CustomConfigurationOptions.Deadline:00:01:30
IOptionsMonitor CustomConfigurationOptions.Enabled:True
IOptionsMonitor CustomConfigurationOptions.Retry:2
IOptionsMonitor CustomConfigurationOptions.Level:Warning
appsettings.json에서 CustomConfigurationOptions의 값을 변경 후 enter를 입력합니다 :
IOptionsMonitor CustomConfigurationOptions.Deadline:00:11:30
IOptionsMonitor CustomConfigurationOptions.Enabled:False
IOptionsMonitor CustomConfigurationOptions.Retry:1
IOptionsMonitor CustomConfigurationOptions.Level:Information
```

IOptionsMonitor 인터페이스는 OnChange() 멤버를 공개합니다. 이 메서드는 TOptions가 변경될 때 호출되는 리스너를 등록합니다. 구성 요소의 값이 변경될 때 리스너를 통해 필요한 동작을 설정할 수 있습니다. OnChange 메서드를 사용할 경우, 메서드의 반환 형식인 IDisposable을 주의해야 합니다. 서비스의 수명이 종료될 경우 명시적으로 OnChange 메서드의 반환 객체의 Dispose 메서드를 호출해야 합니다. 호출하지 않을 경우 의존 서비스의 메모리 누수 위험이 있습니다. (IOptionsMonitor 구현 객체는 OnChange 메서드를 호출한 의존 서비스에 대한 참조를 가집니다. IOptionsMonitor의 수명은 Singleton이므로 서비스의 수명이 예상대로 동작하지 않게 됩니다.)

MonitorService에 OnChange 메서드를 사용하는 코드를 추가합니다.

```csharp
public class MonitorService
{
    private readonly IOptionsMonitor<CustomConfigurationOptions> _options;
    ***IDisposable? _disposableOnChanges;***

    public MonitorService(IOptionsMonitor<CustomConfigurationOptions> options)
    {
        _options = options;

        ///OnChange 함수는 IDisposable 형식을 반환합니다.
        ///메모리 누수를 방지하기 위해서 서비스의 수명이 끝날 때 IDisposable.Dispose를 호출해야 합니다.
        ***_disposableOnChanges = options.OnChange((opt, str) => { });***
    }

    public void DisplayOption()
    {
        Console.WriteLine($"IOptionsMonitor CustomConfigurationOptions.Deadline:{_options.CurrentValue.Deadline}");
        Console.WriteLine($"IOptionsMonitor CustomConfigurationOptions.Enabled:{_options.CurrentValue.Enabled}");
        Console.WriteLine($"IOptionsMonitor CustomConfigurationOptions.Retry:{_options.CurrentValue.Retry}");
        Console.WriteLine($"IOptionsMonitor CustomConfigurationOptions.Level:{_options.CurrentValue.Level}");
    }

    ***public MonitorService ReturnSelf()
    {
        return this;
    }

    public void Dispose()
    {
        _disposableOnChanges?.Dispose();
    }***
}
```

program.cs도 수정하여 다량의 MonitorService를 만든 뒤 Dispose 호출 유무에 따라 dotMemory같은 도구를 통해 메모리 상태를 확인합니다. 

```csharp
private static void RequestManyIOptionsMonitorService(ServiceProvider serviceProvider)
{
    for (int i = 0; i < 200; i++)
    {
        var monitorService = serviceProvider.GetRequiredService<MonitorService>().ReturnSelf();
        ///호출 유무에 따른 메모리 누수를 확인합니다.
        //monitorService.Dispose();
    }
}
```

# Named options

동일한 속성을 여러 구성 섹션에서 사용할 때, 옵션의 이름을 구분하여 같은 형식에 바인딩할 수 있습니다.

동일한 속성을 사용하는 다른 섹션을 appsettings.json에 추가합니다.

```json
"Options": {
  "Base": {
    "Enabled": true,
    "Url": "https://www.google.com/"
  },
  "Derive": {
    "Enabled": false,
    "Url": "https://www.microsoft.com/"
  }
}
```

Base , Derive 두 섹션에서 사용하기 위한  OptionFeatures클래스를 작성합니다.

```csharp
public class OptionFeatures
{
    public const string Base = nameof(Base);
    public const string Derive = nameof(Derive);

    public bool Enabled { get; set; }
    public string Url { get; set; }
}
```

OptionFeatures를 사용하는 서비스 클래스를 작성합니다. IOptions<TOptions>는 Named options를 지원하지 않습니다. 예제에서는 IOptionsSnapshot<TOptions> 형식으로 사용합니다.

```csharp
public class NamedOptionsService
{
    public readonly OptionFeatures _baseFeature;
    public readonly OptionFeatures _deriveFeature;

    public NamedOptionsService(IOptionsSnapshot<OptionFeatures> namedOptionFeatures)
    {
        _baseFeature = namedOptionFeatures.Get(OptionFeatures.Base);
        _deriveFeature = namedOptionFeatures.Get(OptionFeatures.Derive);

        Console.WriteLine($"Base OptionFeatures.Url:{_baseFeature.Url}");
        Console.WriteLine($"Base OptionFeatures.Enabled:{_baseFeature.Enabled}");
        Console.WriteLine($"Derive OptionFeatures.Url:{_deriveFeature.Url}");
        Console.WriteLine($"Derive OptionFeatures.Enabled:{_deriveFeature.Enabled}");
    }
}
```

program.cs에 다음 코드를 추가합니다.

```csharp
//생략됨..

builder.Services.Configure<OptionFeatures>(OptionFeatures.Base,
                                     builder.Configuration.GetSection("OptionFeatures:Base"));
builder.Services.Configure<OptionFeatures>(OptionFeatures.Derive,
                                     builder.Configuration.GetSection("OptionFeatures:Derive"));
                                     
builder.Services.AddTransient<NamedOptionsService>();        

var serviceProvider = builder.Services.BuildServiceProvider();
serviceProvider.GetRequiredService<NamedOptionsService>();                             
//생략됨..
```

# Options validation

바인딩 모델의 속성에 DataAnnotaion을 설정하여 옵션의 값이 유효한 지 검증할 수 있습니다. 

appsettings.json 파일에 다음과 같이 섹션을 추가합니다.

```json
"OptionsValidationSection": {
  "Title": "Just Title",
  "Email": "example@test.com",
  "Qty": 10,
  "DueDate": "2024-07-07"
}
```

구성 요소가 바인딩 될 ValidateOption 클래스를 추가하고 DataAnnotation을 설정합니다.

```csharp
public class ValidateOption
{
    public const string SectionName = "OptionsValidationSection";
    [Required]
    [RegularExpression("^[a-zA-Z'\\s]{1,50}$")]
    public string Title { get; set; }
    [Required]
    [EmailAddress(ErrorMessage = "이메일 형식이 아닙니다")]
    public string Email { get; set; }
    [Required]
    [Range(0, 1000)]
    public int Qty { get; set; }
    [Required]
    [Range(typeof(DateTime), "2000-01-01", "2030-12-31")]
    public DateTime DueDate { get; set; }
}
```

서비스 클래스를 작성합니다. 검증 조건에 맞지 않을 경우 메세지를 출력합니다.

```csharp
public class ValidationService
{
    private readonly IOptions<ValidateOption> _options;

    public ValidationService(IOptions<ValidateOption> options)
    {
        _options = options;

        try
        {
            ValidateOption validateSection = _options.Value;
            Console.WriteLine($"valid OptionsValidationSection.Title:{validateSection.Title}");
            Console.WriteLine($"valid OptionsValidationSection.Email:{validateSection.Email}");
            Console.WriteLine($"valid OptionsValidationSection.Qty:{validateSection.Qty}");
            Console.WriteLine($"valid OptionsValidationSection.DueDate:{validateSection.DueDate}");
        }
        catch (OptionsValidationException ex)
        {
            foreach (var failure in ex.Failures)
            {
                Console.WriteLine($"Validation error : {failure}");
            }
        }
    }
}
```

구성 섹션을 바인딩하고 유효성 검증을 수행하도록 program.cs에 코드를 작성합니다. AddOptions를 호출하여 바인딩 될 모델을 가져온 뒤, 구성 섹션의 이름을 통해 구성 요소의 값을 가져오고 바인딩 합니다. ValidDataAnnotations 메서드를 호출하여 유효성 검증을 활성화 합니다.

```csharp
//생략됨..

builder.Services.AddOptions<ValidateOption>()
                .Bind(builder.Configuration.GetSection(ValidateOption.SectionName))
                .ValidateDataAnnotations();
                
builder.Services.AddTransient<ValidationService>();

var serviceProvider = builder.Services.BuildServiceProvider();
serviceProvider.GetRequiredService<ValidationService>();

var host = builder.Build();
host.Run();
```

검증에 필요한 추가적인 논리를 Validate 메서드를 통해 전달할 수 있습니다.

```csharp
builder.Services.AddOptions<ValidateOption>()
                .Bind(builder.Configuration.GetSection(ValidateOption.SectionName))
                .ValidateDataAnnotations()
                .Validate(config =>
                {
                    if (config.Qty >= 200)
                        return config.DueDate < DateTime.Parse("2025-12-31 23:59:59");
                    return true;
                }, "Qty가 200이상 일 경우 DueDate는 2025-12-31 23:59:59 이전이어야 합니다.");
```

유효성 검증은 런타임에 해당 구성 요소가 사용될 때 수행됩니다. 대신 프로그램이 실행될 때 검증을 수행하려면 ValidateOnStart 메서드를 호출합니다.

```csharp
builder.Services.AddOptions<ValidateOption>()
                .Bind(builder.Configuration.GetSection(ValidateOption.SectionName))
                .ValidateDataAnnotations()
                .ValidateOnStart();
```

.NET 8부터는 AddOptionsWithValidateOnStart 메서드를 사용할 수 있습니다.

```csharp
builder.Services.AddOptionsWithValidateOnStart<ValidateOption>()
                .Bind(builder.Configuration.GetSection(ValidateOption.SectionName))
                .ValidateDataAnnotations();
```

DataAnnotations를 활용한 유효성 검증 외에도 IValidateOptions 인터페이스를 구현한 클래스를 작성하여 검증 논리를 추가할 수 있습니다.

```csharp
public class CustomeValidatation : IValidateOptions<ValidateOption>
{
    private readonly IConfiguration _configuration;

    public CustomeValidatation(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public ValidateOptionsResult Validate(string? name, ValidateOption options)
    {
        StringBuilder? sb = null;

        ///Qty에 대한 검증 조건을 추가합니다. 
        ///ValidateDataAnnotations 메서드를 호출했다면
        ///ValidateOption 클래스에 DataAnnotation으로 설정한 조건과 중복 적용 됩니다.
        if (options.Qty < 0 || options.Qty > 50)
            (sb ??= new()).AppendLine($"The field {nameof(options.Qty)}({options.Qty}) must be between 0 and 50");

        if (sb == null)
            return ValidateOptionsResult.Success;

        return ValidateOptionsResult.Fail(sb.ToString());
    }
}
```

program.cs에서 CustomeValidatation을 등록하는 코드를 추가합니다.

```csharp
//생략됨...
builder.Services.Configure<ValidateOption>(
    builder.Configuration.GetSection(ValidateOption.SectionName));
//생략됨...
```

지금까지 강력한 형식을 통해 구성 요소를 사용할 수 있는 다양한 방법을 제공하는 옵션 패턴에 대해 살펴보았습니다. 글에서 설명한 내용 외에도 [주입된 서비스를 통해 구성 요소 값을 설정](https://learn.microsoft.com/en-us/dotnet/core/extensions/options#use-di-services-to-configure-options)하거나, [실행 후에 구성 값을 재설정](https://learn.microsoft.com/en-us/dotnet/core/extensions/options#options-post-configuration) 할 수도 있습니다. 

 

# Reference

- [Configuration providers - .NET | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration-providers)
- [Options pattern - .NET | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/extensions/options)
- [Options pattern in ASP.NET Core | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-8.0&source=recommendations)
- [.NET Core 탐색 1부 - .NET 6의 ConfigurationManager 내부 보기 | Andrew Lock - 👨‍🏫 튜토리얼, 팁, 강좌 - 닷넷데브 (dotnetdev.kr)](https://forum.dotnetdev.kr/t/net-core-1-net-6-configurationmanager-andrew-lock/2180)
- https://github.com/dotnet/runtime/issues/36130
- [Understanding IOptions, IOptionsMonitor, and IOptionsSnapshot in .NET 7 | Code4IT](https://www.code4it.dev/blog/ioptions-ioptionsmonitor-ioptionssnapshot/)
- [Options Pattern Validation in .NET Core with Examples (c-sharpcorner.com)](https://www.c-sharpcorner.com/article/options-pattern-validation-in-net-core-with-examples/)
