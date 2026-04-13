# BepInExConfigManager

IL2CPP 및 Mono Unity 게임용 BepInEx 설정을 관리하기 위한 게임 내 UI.

IL2CPP에는 BepInEx 6, Mono에는 BepInEx 5가 필요합니다.

✨ Powered by [UniverseLib](https://github.com/sinai-dev/UniverseLib)

## 배포 [![](https://img.shields.io/github/release/sinai-dev/BepInExConfigManager.svg?label=release%20notes)](../../releases/latest)

* [다운로드 (IL2CPP)](https://github.com/sinai-dev/BepInExConfigManager/releases/latest/download/BepInExConfigManager.Il2Cpp.zip)
* [다운로드 (Mono)](https://github.com/sinai-dev/BepInExConfigManager/releases/latest/download/BepInExConfigManager.Mono.zip)

## 사용 방법

* `plugins/BepInExConfigManager.{VERSION}.dll` 및 `UniverseLib.{VERSION}.dll` 파일을 `BepInEx/plugins/` 폴더에 넣으십시오.
* `patchers/BepInExConfigManager.{VERSION}.Patcher.dll` 파일을 `BepInEx/patchers/` 폴더에 넣으십시오.
* 게임을 시작하고 `F5`를 눌러 메뉴를 여십시오.
* 메뉴의 `BepInExConfigManager` 범주에서 키 바인딩을 변경하거나, `BepInEx/config/com.sinai.BepInExConfigManager.cfg` 파일을 편집하여 변경할 수 있습니다.

[![](img/preview.png)](https://raw.githubusercontent.com/sinai-dev/BepInExConfigManager/master/img/preview.png)

## 일반적인 문제와 해결 방법

이 도구는 대부분의 Unity 게임에서 별도 설정 없이 작동해야 하지만, 경우에 따라 제대로 작동하도록 설정을 조정해야 할 수도 있습니다.

설정을 조정하려면 다음 설정 파일을 여십시오: `BepInEx\config\com.sinai.bepinexconfigmanager.cfg`

다음 설정을 조정해 보고 문제가 해결되는지 확인하십시오:
* `Startup_Delay_Time` - 5~10초(또는 필요에 따라 그 이상)로 늘리십시오. 시작 중 UI가 파괴되거나 손상되는 문제를 해결할 수 있습니다.
* `Disable_EventSystem_Override` - 입력이 제대로 작동하지 않으면, 이것을 `true`로 설정해 보십시오.

이러한 해결 방법으로도 작동하지 않으면 이 저장소에 이슈를 생성해 주십시오. 제가 최선을 다해 살펴보겠습니다.

## 개발자 정보

### 고급(숨김) 설정

이 설정 관리자는 `ConfigurationManagerAttributes` 태그 또는 단순한 `"Advanced"` 태그로 정의되는 고급 설정을 지원합니다.

간단한 방법("Advanced" string tag):
```csharp
Config.Bind("Section", "숨김 설정", true, new ConfigDescription("내 설명", null, "고급"));
```

고급 방법 (공식 속성 클래스):
* [여기](https://github.com/BepInEx/BepInEx.ConfigurationManager#overriding-default-configuration-manager-behavior)에 설명된 대로 프로젝트에 `ConfigurationManagerAttributes` 클래스를 포함해야 합니다
* 이 클래스의 다른 속성들은 구현하지 않았지만, 수요가 충분하다면 추후 어느 시점에 구현할 수도 있습니다.
```csharp
Config.Bind("Section", "고급 설정", true, new ConfigDescription("설명", null,
    new ConfigurationManagerAttributes() { IsAdvanced = true }));
```

### 설정 유형 지원

UI는 기본적으로 다음 유형을 지원합니다:

* 토글: `bool`
* 숫자 입력: `int`, `float` 등 (모든 기본 숫자형)
* 문자열 입력: `string`
* 키 바인더: `UnityEngine.KeyCode` 또는 `UnityEngine.InputSystem.Key`
* 드롭다운: `enum` 또는 `AcceptableValueList`가 있는 모든 설정
* 다중 토글: `[Flags]` 속성이 있는 `enum`
* 색상 선택기: `UnityEngine.Color`
* 구조체 편집기: `UnityEngine.Vector3`, `UnityEngine.Quaternion` 등
* Toml 입력: `BepInEx.Configuration.TomlTypeConverter`에 등록된 해당 TypeConverter가 있는 그 밖의 모든 항목.

#### 슬라이더
슬라이더를 만들려면 숫자형을 사용하고, 항목을 생성할 때 `AcceptableValueRange`를 제공하면 됩니다. 예를 들면 다음과 같습니다:
```csharp
Config.Bind("Section", "정수(Int) 슬라이더", 32, new ConfigDescription("모든 숫자형에 슬라이더를 사용할 수 있습니다",
        new AcceptableValueRange<int>(0, 100))); 
```

#### 드롭다운

드롭다운은 `enum` 형식과 `AcceptableValueList`가 제공된 모든 설정에 사용됩니다.

`AcceptableValueList`를 사용하면 다른 모든 UI 처리기를 덮어쓰고 강제로 드롭다운으로 지정합니다.

```csharp
Config.Bind(new ConfigDefinition("Section", "일부 목록"), "One", new ConfigDescription("문자열 목록의 예시",
        new AcceptableValueList<string>("One", "Two", "Three", "Four", "etc..."))); 
```

#### 사용자 지정 UI 처리기
Type에 대한 Toml 입력은 해당 Type용 자체 InteractiveValue를 등록하여 덮어쓸 수 있습니다. 보다 구체적인 예시는 [기존 클래스](https://github.com/sinai-dev/BepInExConfigManager/tree/main/src/UI/InteractiveValues)를 참조하십시오.
```csharp
// 'Something'을 처리할 InteractiveValue 클래스를 정의합니다
public class InteractiveSomething : InteractiveValue
{
    // 이 생성자를 선언해야 합니다
    public InteractiveSomething(object value, Type fallbackType) : base(value, fallbackType) { }

    // 더 엄격하게 하려면 "if type == typeof(Something)"도 확인할 수 있습니다
    public override bool SupportsType(Type type) => typeof(Something).IsAssignableFrom(type);

    // 필요에 따라 다른 메서드도 재정의하십시오
}

// BasePlugin.Load 메서드에 클래스를 등록하십시오:
public class MyMod : BepInEx.IL2CPP.BasePlugin
{
    public override void Load()
    {
        InteractiveValue.RegisterIValueType<InteractiveSomething>();
    }
}
```
