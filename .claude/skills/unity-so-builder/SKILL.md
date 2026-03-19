---
name: unity-so-builder
description: "Unity 에디터 스크립트로 ScriptableObject 생성, 프리팹 필드 할당, 설정 일괄 변경 패턴. SO/프리팹 설정 작업 시 참조."
---

# Unity SO Builder 패턴

## 트리거
에디터 스크립트, SO 생성, 프리팹 설정, 일괄 할당, ScriptableObject, editor script

## 개요

Unity 에디터 스크립트로 ScriptableObject 생성, 프리팹 컴포넌트 필드 할당, 설정 일괄 변경을 자동화하는 표준 패턴.

---

## 기본 템플릿

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

public static class Setup{FeatureName}
{
    [MenuItem("ScrapMagnet/{Category}/{MenuName}")]
    public static void Execute()
    {
        // 1. SO 생성/로드
        // 2. 프리팹 필드 할당
        // 3. 저장

        AssetDatabase.SaveAssets();
        Debug.Log("[Setup{FeatureName}] Complete!");
    }
}
#endif
```

---

## 패턴 1: SO 에셋 생성

```csharp
var path = "Assets/_Game/Scrap_Magnet/Settings/{Name}.asset";
var so = AssetDatabase.LoadAssetAtPath<MySettings>(path);
if (so == null)
{
    so = ScriptableObject.CreateInstance<MySettings>();
    AssetDatabase.CreateAsset(so, path);
}

// 필드 설정
var serialized = new SerializedObject(so);
serialized.FindProperty("fieldName").floatValue = 1.5f;
serialized.FindProperty("stringField").stringValue = "value";
serialized.FindProperty("boolField").boolValue = true;
serialized.FindProperty("enumField").enumValueIndex = (int)MyEnum.Value;
serialized.ApplyModifiedProperties();
EditorUtility.SetDirty(so);
```

---

## 패턴 2: 프리팹 컴포넌트 필드 할당

```csharp
// 프리팹 로드
var prefab = AssetDatabase.LoadAssetAtPath<GameObject>(
    "Assets/_Game/Scrap_Magnet/Prefabs/Enemy/Enemy_Basic.prefab");
if (prefab == null) return;

// 컴포넌트 SerializedObject
var component = prefab.GetComponent<MyComponent>();
var so = new SerializedObject(component);

// 단일 참조 할당
so.FindProperty("_settings").objectReferenceValue = settingsAsset;

// 배열 할당
var arrayProp = so.FindProperty("_items");
arrayProp.arraySize = items.Length;
for (var i = 0; i < items.Length; i++)
    arrayProp.GetArrayElementAtIndex(i).objectReferenceValue = items[i];

so.ApplyModifiedProperties();
EditorUtility.SetDirty(prefab);
```

---

## 패턴 3: 에셋 검색 + 일괄 변경

```csharp
// 타입+이름으로 검색
var guids = AssetDatabase.FindAssets("t:EnemySettings Enemy_Tank",
    new[] { "Assets/_Game/Scrap_Magnet/Settings" });

foreach (var guid in guids)
{
    var path = AssetDatabase.GUIDToAssetPath(guid);
    var asset = AssetDatabase.LoadAssetAtPath<EnemySettings>(path);
    if (asset == null) continue;

    var so = new SerializedObject(asset);
    so.FindProperty("maxHealth").floatValue = 50f;
    so.ApplyModifiedProperties();
    EditorUtility.SetDirty(asset);
    Debug.Log($"Updated: {path}");
}
```

---

## 패턴 4: 프리팹 복제 + 변형

```csharp
var srcPath = "Assets/.../Original.prefab";
var dstPath = "Assets/.../Variant.prefab";

if (!AssetDatabase.LoadAssetAtPath<GameObject>(dstPath))
    AssetDatabase.CopyAsset(srcPath, dstPath);

var prefab = AssetDatabase.LoadAssetAtPath<GameObject>(dstPath);
prefab.transform.localScale = Vector3.one * 1.5f;
EditorUtility.SetDirty(prefab);
```

---

## 패턴 5: SpawnSystemSettings WaveComposition 설정

배열 of struct 설정 (복잡한 중첩 구조):

```csharp
var so = new SerializedObject(settings);
var compositions = so.FindProperty("waveCompositions");
compositions.arraySize = 7;

// 각 composition 설정
var comp = compositions.GetArrayElementAtIndex(0);
comp.FindPropertyRelative("waveNumber").intValue = 1;

var entries = comp.FindPropertyRelative("entries");
entries.arraySize = 2;
entries.GetArrayElementAtIndex(0).FindPropertyRelative("poolKey").stringValue = "Enemy_Basic";
entries.GetArrayElementAtIndex(0).FindPropertyRelative("ratio").floatValue = 0.7f;

so.ApplyModifiedPropertiesWithoutUndo();
```

---

## 실제 사용 예시 (이 프로젝트)

| 스크립트 | 용도 | 패턴 |
|---------|------|------|
| `SetupWaveVariety.cs` | WaveComposition 7단계 + DropTable 11종 | 패턴 1+5 |
| `AssignAudioClips.cs` | 프리팹 AudioClip 일괄 할당 | 패턴 2+3 |
| `SetBalanceValues.cs` | 적/터렛 SO 필드 일괄 변경 | 패턴 3 |
| `BuildStartingCards.cs` | StartingBuildData SO 생성 | 패턴 1 |
| `SetupScrapVariety.cs` | 프리팹 복제 + ScrapData 설정 | 패턴 4 |
| `BuildSkillTree.cs` | SkillTreeData SO 생성 | 패턴 1 |

---

## 주의사항

- **`#if UNITY_EDITOR` 래핑 필수** — 빌드에서 에디터 API 참조 에러 방지
- **`AssetDatabase.SaveAssets()` 호출** — 변경사항 디스크 저장
- **`EditorUtility.SetDirty()` 호출** — 변경 감지 (Undo 지원)
- **`ApplyModifiedProperties()` vs `ApplyModifiedPropertiesWithoutUndo()`** — Undo 필요 여부
- **프리팹 수정 시 루트 오브젝트에 SetDirty** — 자식 컴포넌트 변경도 루트에 마킹
- **GUID로 에셋 참조** — 경로 변경에 안전 (meta 파일의 guid)

---

## Coplay MCP에서 실행

```
mcp__coplay-mcp__execute_script(filePath: "Assets/.../MyScript.cs")
```

실행 결과는 Unity 로그로 확인:
```
mcp__coplay-mcp__get_unity_logs(search_term: "MyScript", limit: 5)
```
