---
title: "The binding of Isaac 모작 (8)"
date: 2025-12-12 18:50:20 +0900
description: The binding of Isaac의 맵을 편하게 만들어보자
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

## 8. 맵에디팅
로그라이크 게임의 매력은 랜덤성이라고 생각한다. 매 판마다 달라지는 맵, 아이템들이 재미를 부각시켜준다.
The binding of Isaac에서 랜덤한 맵은 어떻게 생성될까?

맵 생성에 관해서는 이 시리즈의 4번에서 다룬적이 있다. 하지만 이는 맵의 형태에 관한것이지 방 하나하나의 장애물 몬스터 등의 세부 디자인은 정해지지 않았다.

맵에디팅을 만들기전에 원작의 맵 에디팅 방식을 참고했다

### 1. 원작의 맵 에디팅
The binding of Isaac에서 공식적으로 제공하는 에디팅 툴은'Isaac Room Editor'이다.
( 모딩 커뮤니티에서는 Basement Renovator라는 툴을 사용한다)
> ![](https://velog.velcdn.com/images/garage_keeper/post/7d2b6eef-7f4f-4fa9-a11a-c5f5702a3bd7/image.png)
> * 중앙 영역 : 에디팅 공간
> * 좌측 위의 빨간 영역 : Room Type 
>   * 특수방, 스테이지 별 일반방등으로 분류한 큰 카테고리
> * 좌측 하얀 영역 : 각 카테고리에 포함된 방(디자인)
> * 우측 상단 영역 : 각 방에 배치된 엔티티의 정보를 확인하는  창
> * 우측 하단 영역 : 방의 기본 정보확인
>   * ID, 서브타입, 난이도, 사이즈 , 이름, 비중을 조절
> * 하단 영역 : 맵에 배치할 엔티티

Editing 방법은 다음과 같다.
1. 좌측의 new 혹은 Duplicate를 눌러서 새로운 디자인을 생성
2. 방의 기본 정보를 설정
3. 방에 엔티티 (장애물, 픽업, 몬스터) 배치
4. 저장시 xml파일 생성

이러한 방식으로 사용될 방들을 쉽게 만들 수 있다.

<br>

### 2. 내가 만든 맵 에디팅

Unity에서 타일맵을 기반으로 프리팹을 사용하도록 제작하였다.
> ![](https://velog.velcdn.com/images/garage_keeper/post/b893afec-f095-4827-9ce7-781a5a0e5f9d/image.png)
> * 에디팅 영역 : Scene 뷰, 타일 팔레트를 이용해서 배치
> * Room Type 
>   * 프로젝트의 폴더 구조를 통해서 구분
>   * 각 폴더에 타입에 포함되는 방들 저장
> * Tile Palette
>   * 배치될 엔티티들 
> * Hierarchy
>   * Collision : 장애물, 픽업 배치
>   * Monster   : 몬스터 배치

<br>

### 3. 맵 로드하기
#### 1 )  맵 불러오기
MapManager.cs의 GenerateRoom 함수의 일부
``` csharp
int stageIndex = 0;
int roomId;

// 보스방
if (r.RoomType == ERoomType.Boss)
{
    roomId = GetBossRoomId(r.DeadEndType);
}
// 나머지 모든 방
else
{
    roomId = GetRoomId(r.RoomType);
}

// Room instance 생성
GameObject roomTileMap = InstantiateRoom(roomId, room.transform);
```

MapManager.cs
``` csharp
public int GetBossRoomId(int stage, EDeadEndType currentDeadEndType)
{
    int selectedId = BossToStage[Managers.Game.StageNumber];
    var selectedRoom = Managers.Data.RoomDicTotal[selectedId];
    // 소환할 수 없는 DeadEnd의 경우 예비로 남긴 방 사용
    if (selectedRoom.CannotSpawnType == currentDeadEndType)
    {
        // 예비 보스 룸 ID
        selectedId = BossToStage[Managers.Game.MAXStageNumber + 1];
    }
    return selectedId;
}

public int GetRoomId(ERoomType type)
{
    //현재 에디팅한 방이 많지 않아서 현재는 0스테이지에 모두 몰아 넣었음
    //향후에는 Manager.Game.StageNumber 사용
    int stage = 0;
    var rooms = Managers.Data.RoomDic[type][stage];
    int index = Managers.Game.RNG.RandInt(0, rooms.Count - 1);
    return rooms[index];
}

public GameObject InstantiateRoom(int roomId, Transform parent)
{
    string prefabName = Managers.Data.RoomDicTotal[roomId].PrefabName;
    GameObject instance = Managers.Resource.Instantiate(prefabName);
    instance.transform.SetParent(parent);
    return instance;
}
```

<br>

#### 2) Scene에 소환하기 (몬스터, 픽업도 유사하게 진행)
MapManager.cs
``` csharp
// Obstacle gameobject에 prefab을 소환하는 방식
public void SetObstacle(RoomClass room)
{
    // 오브젝트 스프라이트 종류를 정하기위한 rng
    RNGManager obsRng = new RNGManager(room.ObjectSeed);
    Tilemap tmp = room.TilemapCollisionPrefab.GetComponent<Tilemap>();
    // 타일맵의 크기정보
    int maxX = tmp.cellBounds.xMax;
    int minX = tmp.cellBounds.xMin;
    int maxY = tmp.cellBounds.yMax;
    int minY = tmp.cellBounds.yMin;

    for (int y = maxY - 1; y > minY + 1; y--)
    {
    	for (int x = minX + 1; x < maxX - 1; x++)
      	{
        	Vector3Int tilePos = new Vector3Int(x, y, 0);
        	TileBase tile = tmp.GetTile(tilePos);
        	if (tile == null) continue;

        	HandleTile(room, tile.name, tilePos, obsRng);
    	}
	}
    // 시드 갱신
    room.ObjectSeed = obsRng.Sn;
}
```

MapManager.cs
```csharp
private void HandleTile(RoomClass room, string tileName, Vector3Int tilePos, RNGManager rng)
{
    // 스프라이트가 여러개 있어서 하나를 골라야하는 장애물
    string[] randomObstacles = { "Rock", "Urn" };
    // 스프라이트가 고정된 장애물
    string[] fixedObstacles = { "Spike", "Fire", "Poop" };

    if (tileName == "ItemHolder")
    {
        GameObject itemHolder = Managers.Resource.Instantiate("ItemHolder", room.Obstacle.transform);
        itemHolder.GetComponent<ItemHolder>().Init(room, tilePos);
		// 보스방은 클리어 한 후에 활성화
        if (room.RoomType == ERoomType.Boss)
               itemHolder.SetActive(false);

        room.ItemHolder = itemHolder;
    }
    else if (fixedObstacles.Contains(tileName))
    {
        Managers.Object.SpawnObstacle(tilePos, tileName, room.Obstacle.transform);
    }
    else if (randomObstacles.Contains(tileName))
    {
        int spriteIndex = rng.RandInt(1, 3);
        Managers.Object.SpawnObstacle(tilePos, tileName, room.Obstacle.transform, spriteIndex);
    }
}
```

<br>

### 4. 구현 결과
  ![](https://velog.velcdn.com/images/garage_keeper/post/71d2f81e-7cdd-44f1-b781-6300433ff28b/image.png)
  Pickup, Obstacle이 잘 소환된 모습

