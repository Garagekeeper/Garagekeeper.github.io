---
title: "The binding of Isaac 모작 (7)"
date: 2025-12-12 18:47:35 +0900
description: The binding of Isaac의 Item을 구현해보자
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

## 7. Item
아이작에는 방대한 양의 아이템들이 있다.처음에는 Item마다 새로운 스크립트를 만드는 방식으로 했지만, 몇천 가지가 넘어가는 아이템들의 스크립트를 다 관리 할 수 없을 거 같아 효율적인 방법을 고심했다.

>아이템 요구사항은 다음과 같았다.
1. 최소한의 스크립트로 기본 스탯( 체력, 소지품, 공격력, 공격 속도 등)을 적용한다.
2. 액티브 아이템의 경우 각각의 효과를 적용할 수 있어야한다.
>
>해결방안
1. 새로운 파일을 만드는게 아니라 ItemData 인스턴스들을 Dictionary에 담아서 관리
2. 액티브 아이템의 효과는 한곳에 모아서 함수를 만든다.
	-추후에 델리게이트로 딕셔너리에 저장해서 관리하는 좋은 방법도 있음

<br>

### 1.ItemData

* Item Data: 아이템의 기본정보를 담는 클래스.
* ItemDataLoader: ILoader 인터페이스를 상속 받는 클래스 
  * Json 파일을 딕셔너리로 불러올때 사용되는 클래스

```csharp
namespace Data
{
    [Serializable]
    public class BaseData
    {
        public int DataId;
        public string Description;
        public string SpriteName;
    }

    [Serializable]
    public class ItemData : BaseData
    {
        public string Name;
        public int CoolTime;
        public EItemType Type;
        >
        public float Hp;
        //DMG
        public float DmgUp;
        public float FlatDmgUp;
        public float Multiplier;
       
        public float Tears;
        public float Range;
        public float ShotSpeed;
        public float Speed;
        public float Luck;

        public int Life;
        public string SetItem;
        public EPICKUP_TYPE PickupType;
        public int PickupCount;
        public EShotType ShotType;
       	
        //추후 델리케이트로 변환 시 삭제될 필드
        public ESpecialEffectOfActive EffectOfActive;
        public int Weight;
    }

    [Serializable]
    public class ItemDataLoader : ILoader<int, ItemData>
    {
        public List<ItemData> items = new List<ItemData>();

        public Dictionary<int, ItemData> MakeDict()
        {
            Dictionary<int, ItemData> dict = new Dictionary<int, ItemData>();
            foreach (ItemData item in items)
                dict.Add(item.DataId, item);

            return dict;
        }
    }
 }
```

<br>

### 2.Item

  * Item class는  Data를 담은 딕셔너리를 기반으로 실제 사용할 때 인스턴스화.
  * EffectOfActive 필드는 현재는 Enum
  	* 추후에는 필요없어질 필드
    * 나중에서는 id만으로 딕셔너리에서 델리케이트를 가져올 수 있도록
  
```csharp
using Data;
using static Define;

public class Item
{
    public int TemplateId { get; set; }
    public string Description {  get; set; }
    public string SpriteName { get; set; }
    public string Name { get; set; }
    public int CoolTime { get; private set; }
    public EItemType ItemType { get; set; }

    public float Hp { get; set; }
    public float DmgUp { get; set; }
    public float FlatDmgUp { get; set; }
    public float Multiplier { get; set; }
    public float Tears { get; set; }
    public float Range { get; set; }
    public float ShotSpeed { get; set; }
    public float Speed { get; set; }
    public float Luck { get; set; }
    public int Life { get; set; }
    public string SetItem { get; set; }
    public EPICKUP_TYPE PickupType { get; set; }
    public int PickupCount { get; set; }
    public EShotType ShotType { get; set; }
    public ESpecialEffectOfActive EffectOfActive { get; set; }
    public int FamiliarID { get; set; }
    public int CurrentGage { get; set; }

    public ItemData TemplateData
    {
        get
        {
            return Managers.Data.ItemDic[TemplateId];
        }
    }

    public Item() { }

    public Item(int itemId)
    {
        TemplateId = itemId;
        SpriteName = TemplateData.SpriteName;
        Description = TemplateData.Description;
        Name = TemplateData.Name;
        CoolTime = TemplateData.CoolTime;
        ItemType = TemplateData.Type;
        Hp = TemplateData.Hp;
        DmgUp = TemplateData.DmgUp;
        FlatDmgUp = TemplateData.FlatDmgUp;
        Multiplier = TemplateData.Multiplier;
        Tears = TemplateData.Tears;
        Range = TemplateData.Range;
        ShotSpeed = TemplateData.ShotSpeed;
        Speed = TemplateData.Speed;
        Luck = TemplateData.Luck;
        SetItem = TemplateData.SetItem;
        PickupType = TemplateData.PickupType;
        PickupCount = TemplateData.PickupCount;
        ShotType = TemplateData.ShotType;
        EffectOfActive = TemplateData.EffectOfActive;
        CurrentGage = CoolTime;

    }
}
```
  
<br>
<br>

### 3.아이템의 적용 
#### 1) Active
  
```csharp
//issac에는 그 방에서만 능력치를 올려주는 액티브 아이템이 있다.
//일회성 버프 아이템은 패시브 아이템처럼 적용하되, 방이 클리어되면회수 되도록한다.
//다른 액티브 아이템은 enum을통해서 별도로 효과를 적용시켜준다. 
private void HandleUsingActiveItem(Item item)
{
    if (item.EffectOfActive == ESpecialEffectOfActive.Null)
        ApplyPassiveItemEffect(item, true);
    else
    {
        transform.GetComponent<Animator>().Play("UseItem", 0, 0);
        switch (item.EffectOfActive)
        {
            case ESpecialEffectOfActive.RandomTeleport:
                 Managers.Game.TPToNormalRandom();
                 break;
            case ESpecialEffectOfActive.UncheckedRoomTeleport:
                break;
            case ESpecialEffectOfActive.Roll:
                Managers.Game.Roll(this, "item");
                break;
        }
    }
}
```

#### 2) Passive

``` csharp
public void ApplyPassiveItemEffect(Item item, bool isOneTime = false)
{
    // 1회성 아이템인지 확인
    if (isOneTime)
    {
        OneTimeActive = true;
    }

    // 기본 스택 적용
    Hp += item.Hp;
    TotalDmgUp += item.DmgUp;
    FlatDmgUp += item.FlatDmgUp;
    Multiplier = item.Multiplier;
    Tears += item.Tears;
    Range += item.Range;
    ShotSpeed += item.ShotSpeed;
    Speed += item.Speed;
    Luck += item.Luck;
    Life += item.Life;
  
    // 픽업 아이템인 경우
    if (item.PickupType == EPICKUP_TYPE.PICKUP_COIN)
    {
        Coin += item.PickupCount;
    }
    else if (item.PickupType == EPICKUP_TYPE.PICKUP_KEY)
    {
        KeyCount += item.PickupCount;
    }
    else if (item.PickupType == EPICKUP_TYPE.PICKUP_BOMB)
    {
        BombCount += item.PickupCount;
    }

    //데미지 계산
    CalcAttackDamage();
    //적용된 능력치 UI 갱신
    Managers.UI.ResfreshUIAll(this);
    }
```
  
<br>
<br>

### 4. 엑셀파일 관리 
* 엑셀파일은 CSV로 저장되며 다음과 같은 필드를 가진다.
* 스크립트를 만들지 않고, 쉽게 아이템을 추가/제거 할 수 있다.
* ItemData의 필드 순서/타입과 잘 맞춰서 설계해야한다.
  * 모든칸에 값을 넣기는 힘드니, 빈칸이면 기본값이 들어가게 설계
  
![](https://velog.velcdn.com/images/garage_keeper/post/5995704a-b218-4c5b-8ab7-e616d8583cfd/image.png)

<br>
<br>

### 5.CSV 파일을 Json으로 변환하기
* 유니티 Tool로 구현
  
```cs
// 제네릭 헬퍼: Loader 타입 인스턴스 생성 후 첫 번째 필드(리스트)에 파싱 결과 대입 → JSON 직렬화
private static void ParseExcelDataToJson<Loader, LoaderData>(string filename)
    where Loader : new()
    where LoaderData : new()
{
    Debug.Log(filename);
    // Loader 인스턴스 생성 (e.g. ItemDataLoader)
    Loader loader = new Loader();
    // Loader 내부 첫 번째 필드(보통 List<LoaderData>) 가져오기
    FieldInfo field = loader.GetType().GetFields()[0];
    // 파싱된 리스트를 해당 필드에 설정
    field.SetValue(loader, ParseExcelDataToList<LoaderData>(filename));

    // JSON 포맷으로 문자열 직렬화
   string jsonStr = JsonConvert.SerializeObject(loader, Formatting.Indented);
    // 에셋 폴더에 JSON 파일로 저장
    File.WriteAllText($"{Application.dataPath}/@Resources/Data/JsonData/{filename}.json", jsonStr);
    // 에디터 내 에셋 갱신
   AssetDatabase.Refresh();
}

// CSV 파일을 읽어 List<LoaderData> 형태로 반환하는 메인 파싱 로직
private static List<LoaderData> ParseExcelDataToList<LoaderData>(string filename)
where LoaderData : new()
{
    List<LoaderData> loaderDatas = new List<LoaderData>();

    // CSV 파일 전체를 읽어서 줄 단위로 split
    string[] lines = File.ReadAllText($"{Application.dataPath}/@Resources/Data/ExcelData/{filename}.csv")
                         .Trim()
                         .Split("\n");

    // 첫 줄(헤더) 제외하고 데이터 행만 rows 리스트에 저장
    List<string[]> rows = new List<string[]>();
    int innerFieldCount = 0;
    for (int l = 1; l < lines.Length; l++)
    {
        string[] row = lines[l].Replace("\r", "").Split(',');
        rows.Add(row);
    }

    // 각 데이터 행마다 LoaderData 인스턴스 생성 후 필드별로 값 설정
    for (int r = 0; r < rows.Count; r++)
    {
        // 빈 행 또는 ID(첫 칼럼)가 비어 있으면 건너뜀
        if (rows[r].Length == 0) continue;
        if (string.IsNullOrEmpty(rows[r][0])) continue;

        innerFieldCount = 0;  // multi-row 처리용 커서 초기화
        LoaderData loaderData = new LoaderData();
        Type loaderDataType = typeof(LoaderData);
        BindingFlags bindingFlags = BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance;
        // 상속 계층 순서대로 모든 필드 수집
        var fields = GetFieldsInBase(loaderDataType, bindingFlags);

        // 다음 레코드 시작 인덱스 탐색: ID 칼럼이 채워진 행까지
        int nextIndex;
        for (nextIndex = r + 1; nextIndex < rows.Count; nextIndex++)
        {
            if (!string.IsNullOrEmpty(rows[nextIndex][0]))
                break;
        }

        // 각 필드에 대해 값을 파싱해서 loaderData에 설정
        for (int f = 0; f < fields.Count; f++)
        {
            FieldInfo field = loaderData.GetType().GetField(fields[f].Name);
            Type type = field.FieldType;

            // 제네릭 타입(List<T>)인 경우: 여러 행 묶음으로 리스트 생성
            if (type.IsGenericType)
            {
                Type valueType = type.GetGenericArguments()[0];
                Type genericListType = typeof(List<>).MakeGenericType(valueType);
                var genericList = Activator.CreateInstance(genericListType) as IList;

                // r부터 nextIndex-1까지 반복하며 각 행의 값을 리스트에 추가
                for (int i = r; i < nextIndex; i++)
                {
                    if (string.IsNullOrEmpty(rows[i][f + innerFieldCount])) continue;
                    bool isCustomClass = valueType.IsClass && !valueType.IsPrimitive && valueType != typeof(string);

                    if (isCustomClass)
                    {
                         // 커스텀 클래스 내부 필드까지 파싱
                        object fieldInstance = Activator.CreateInstance(valueType);
                        FieldInfo[] fieldInfos = fieldInstance.GetType().GetFields(BindingFlags.Public | BindingFlags.Instance);

                        for (int k = 0; k < fieldInfos.Length; k++)
                        {
                            FieldInfo innerField = valueType.GetFields()[k];
                            string str = rows[i][f + innerFieldCount + k];
                            object convertedValue = ConvertValue(str, innerField.FieldType);
                            if (convertedValue != null)
                                innerField.SetValue(fieldInstance, convertedValue);
                        }

                        // 다음 행이 동일 레코드인지 검사하여 innerFieldCount 조정
                        string nextStr = null;
                        if (i + 1 < rows.Count)
                        {
                            if (f + innerFieldCount < rows[i + 1].Length && string.IsNullOrEmpty(rows[i + 1][0]))
                                nextStr = rows[i + 1][f + innerFieldCount];
                        }
                        if (string.IsNullOrEmpty(nextStr) || i + 1 == nextIndex)
                            innerFieldCount = fieldInfos.Length - 1;

                        genericList.Add(fieldInstance);
                    }
                    else
                    {
                        // 기본 타입이면 바로 값 변환 후 리스트에 추가
                        object value = ConvertValue(rows[i][f], valueType);
                        genericList.Add(value);
                    }
                }

  	            // 리스트가 null이 아니면 필드에 설정
                if (genericList != null)
                    field.SetValue(loaderData, genericList);
            }
            else
            {
                // 제네릭이 아닌 경우: 기본 타입 또는 커스텀 클래스 필드 처리
                bool isCustomClass = type.IsClass && !type.IsPrimitive && type != typeof(string);
                if (isCustomClass)
                {
                    // 커스텀 클래스 인스턴스 생성 후 내부 필드 파싱
                    object fieldInstance = Activator.CreateInstance(type);
                    FieldInfo[] fieldInfos = fieldInstance.GetType().GetFields(BindingFlags.Public | BindingFlags.Instance);

                    for (int i = 0; i < fieldInfos.Length; i++)
                    {
                        FieldInfo innerField = type.GetFields()[i];
                        string val = rows[r][f + innerFieldCount + i];
                        object converted = ConvertValue(val, innerField.FieldType);
                        if (converted != null)
                            innerField.SetValue(fieldInstance, converted);
                    }

                    innerFieldCount = fieldInfos.Length - 1;
                    field.SetValue(loaderData, fieldInstance);
                }
                else
                {
                    // 기본 타입이면 ConvertValue 사용
                    object value = ConvertValue(rows[r][f], type);
                    if (value != null)
                        field.SetValue(loaderData, value);
                }
            }
        }

        // 완성된 객체를 리스트에 추가
        loaderDatas.Add(loaderData);
    }
    return loaderDatas;
}

// 문자열을 특정 타입으로 변환 (int, float, enum 등)
private static object ConvertValue(string value, Type type)
{
    if (string.IsNullOrEmpty(value)) return null;
    TypeConverter converter = TypeDescriptor.GetConverter(type);
    //Debug.Log(value);
    return converter.ConvertFromString(value);
}
```
  

 
