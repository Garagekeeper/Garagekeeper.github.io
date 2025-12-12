---
title: "The binding of Isaac 모작 (5)"
date: 2025-12-12 18:37:44 +0900
description: The binding of Isaac의 미니맵을 구현해보자
categories: [Portfolio, GunsGermsAndSteel] #[upper, lower]
tags: [unity, c#, addressable] #must be lower case
math: true
mermaid: true
---

![](https://velog.velcdn.com/images/garage_keeper/post/a28b81f3-b2ce-4c74-823f-d515d3ac2d1e/image.PNG)

## 5. 미니맵
미니맵은 앞서 작성한 맵 정보를 이용해서 작성된다.
>1. 생성된 방들을 기반으로 미니맵 UI에 Cell을 생성한다. (starting을 제외하고 false상태)
> 	* cell 의 Sprite는 currentRoom, Visited, UnVisited 3가지 존재
>2. 미니맵 cell은 조건에 따라서 적절한 스프라이트로 변경한다
>	* 현재 방: currentRoom
	* 인접한 방 중 클리어 X : Unvisited
    * 인접한 방 중 클리어 O : Visited
    

<br>
<br>

### 1. 미니맵 UI에 Cell을 생성한다.

 ![](https://velog.velcdn.com/images/garage_keeper/post/bdb3dd31-8286-44c1-9064-f346bc8af480/image.PNG)

UI쪽은 공부의 깊이가 부족하다보니 원초적인(?) 방법을 사용했다

* cell prefab을 생성한다.
* MinimapPannel의 자식으로 변환시킨다.
* 위치 설정을하고, 스타팅인 경우에만 활성화 시킨다.

MapManager.cs
```cs
public enum ECellType
{
    UnVisited,
    currentRoom,
    Visited,
}

public void GenerateMinimap()
{
    foreach (RoomClass room in Rooms)
    {
        //unvisited, currenet room, visited
        string[] cellSprite = { "minimap1_4", "minimap1_3", "minimap1_2", };

        //Instantiate prefab
        GameObject cell = Managers.Resource.Instantiate("Minimap_Cell");
        cell.GetComponent<Image>().sprite = Managers.Resource.Load<Sprite>(cellSprite[(int)ECellType.UnVisited]);
        cell.name = room.RoomObject.name;

        GameObject pannel = Managers.UI.PlayingUI.GetMinimapPannel();
        cell.transform.SetParent(pannel.transform);

        //set pos
        cell.transform.localPosition = new Vector3(room.YPos - 4, 4 - room.XPos) * 30;

        if (room.RoomType == ERoomType.Start) cell.SetActive(true);
        else cell.SetActive(false);
    }
}
```

<br><br>
  
### 2. 미니맵 cell은 조건에 따라서 적절한 스프라이트로 변경한다
![](https://velog.velcdn.com/images/garage_keeper/post/dda54f28-8243-433e-9b53-46321eed346c/image.PNG)


미니맵이 수정되는 경우는 2가지이다.
* 새로운 stage을 생성할 때
  * 1번 항목에서 해결
* 방을 이동할 때
  * 다시말해 currentRoom이 변경 될 때
    * 이동할 방(다음 방)의 sprite 변경
    * 지금 방 (현재 방)의 sprite 변경
    * 이동할 방(다음 방)의 인접방을 순회하며 sprite 변경
      * 비활성화 되어있는 경우 활성화
      * 특수방의 경우 ICON을 통해서 방을 표시
    * _curentRoom 변경

MapManager.cs
```cs
private RoomClass _currentRoom;

public RoomClass CurrentRoom
{
    get { return _currentRoom; }
    set
    {
        if (_currentRoom != value)
        {
            ChangeMinimapadjacencentCellSprite(value, _currentRoom);
            _currentRoom = value;
        }
    }
}
```

MapManager.cs
```cs
    public void ChangeMinimapadjacencentCellSprite(RoomClass next, RoomClass before)
    {
        if (next == null) return;

        string[] cellSprite = { "minimap1_4", "minimap1_3", "minimap1_2", };
        GameObject go = Managers.UI.PlayingUI.GetMinimapPannel();

        //다음 방
        ChangeMinimapCellSprite(go.transform.Find(next.RoomObject.name).gameObject, cellSprite[(int)ECellType.currentRoom]);

        //이전 방 (원래 있던 방)
        if (before != null)
        {
            int cellSpriteIedex = (int)ECellType.UnVisited;
            if (before.IsClear) cellSpriteIedex = (int)ECellType.Visited;
            ChangeMinimapCellSprite(go.transform.Find(before.RoomObject.name).gameObject, cellSprite[cellSpriteIedex]);
        }
       
        //바뀐 뒤 인접한 방
        for (int i = 0; i < 4; i++)
        {

            RoomClass adjacencentRoom = next._adjacencentRooms[i];
            if (adjacencentRoom != null)
            {
                Transform child = go.transform.Find(adjacencentRoom.RoomObject.name);
                if (adjacencentRoom.RoomType != ERoomType.Normal && adjacencentRoom.RoomType != ERoomType.Start)
                {
                    //roomIcon
                    GameObject temp = child.GetChild(0).gameObject;
                    temp.SetActive(true);
                    temp.GetComponent<Image>().sprite = Managers.Resource.Load<Sprite>("minimap_icons_" + (int)adjacencentRoom.RoomType);
                }

                child.gameObject.SetActive(true);
                string spriteName;
                if (adjacencentRoom.IsClear)
                    spriteName = cellSprite[(int)ECellType.Visited];
                else
                    spriteName = cellSprite[(int)ECellType.UnVisited];

                ChangeMinimapCellSprite(child.gameObject, spriteName);
            }
        }
    }
```
  
MapManager.cs
```cs
public void ChangeMinimapCellSprite(GameObject cell, string spriteName)
{
    cell.SetActive(true);
    cell.GetComponent<Image>().sprite = Managers.Resource.Load<Sprite>(spriteName);
}
```

<br><br>


### 4.여담
  
 * 현재 구현에서 Minimap_Pannel이최대 사이즈 (8x8)을 기준으로 만들어져 있다
   * stage의 크기가 훨씬 작아도 미니맵은 항상 같은 크기
     * 런타임에 조절 할 수 있는 방법이 있을까?
     * UI관해서 더 알아보자
  
  

 
