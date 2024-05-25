---
layout: single
title:  "Finite State Machine(FSM) 유한 상태 머신"
categories: 
    - Unity
tag: [Unity]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-05-25
last_modified_at: 2024-05-25

---

이 글은  [고박사님의 How to make a Excel Importer in unity](https://www.youtube.com/watch?v=2oip0H7VgPM)를 보고 공부한 내용을 정리한 글입니다.
{: .notice--warning}

# Excel Import
유니티에서 데이터를 직접 관리하는 것은 불편한 점이 많다. <br>
이를 해결하기 위해서 간단하게 Excel을 이용해서 데이터를 받아와 유니티에서 사용할 수 있다.

## Excel Importer 설치
엑셀 임포터를 다운받아 쉽게 데이터를 불러올 수 있다. 그러기 위해서 [mikito/unity-excel-importer](https://github.com/mikito/unity-excel-importer)에서 파일을 다운 받아준다.

받은 파일에서 다음 보이는 Excel Importer 폴더를 유니티 프로젝트로 가져온다.
<img width="366" alt="image" src="https://github.com/novicehog/comments/assets/131991619/a603e432-6dec-4ff5-91c1-6a560c7f6b7c">

## Excel 생성
다음으로는 불러올 엑셀 파일을 만들어준다. <br>
엑셀 파일의 경우 첫 번째 열은 각 행의 소제목이 된다. <br>
여기서는 다음과 같이 Branch, Name, Dialog를 소제목으로 데이터를 저장하였다.

<img width="251" alt="image" src="https://github.com/novicehog/comments/assets/131991619/93695ae4-0153-4214-9c7c-03b5c2188ac7">
<br>

이때 시트의 이름도 나중에 불러올 때 중요하므로 필요한 이름으로 저장한다. 

<img width="74" alt="image" src="https://github.com/novicehog/comments/assets/131991619/09e61a9e-0f16-4923-90d7-42bde4a48557">


만든 엑셀 파일 또한 유니티 프로젝트 내로 가져온다.


## 파싱할 데이터 타입을 클래스로 정의
엑셀에서 첫 번째 열로 작성한 소제목과 같은 이름을 같는 직렬화 클래스를 생성

```cs
[System.Serializable]
public class DialogTypeEntity 
{
    public int Branch;
    public string Name;
    public string Dialog;
}
```

## 스크립터블 오브젝트 생성
엑셀 파일을 누르고 Create ExcelAssetScript로 스크립터블 오브젝트를 생성한다.

<img width="470" alt="image" src="https://github.com/novicehog/comments/assets/131991619/1adbd3e7-a828-4400-83ae-f910ec97f14b">
<br>

만들어진 파일에서 주석을 풀고 리스트 제네릭 타입으로 아까 정의해둔 클래스를 넣어준다. <br>
이때 List의 이름은 엑셀 파일을 만들 때 시트 이름과 같아야 한다.   

<img width="74" alt="image" src="https://github.com/novicehog/comments/assets/131991619/09e61a9e-0f16-4923-90d7-42bde4a48557">

<br>

```cs
[ExcelAsset]
public class EntityDBDialog : ScriptableObject
{
	public List<DialogTypeEntity> Entities; // Replace 'EntityType' to an actual type that is serializable.
}
```


이제 엑셀 파일을 ReImport하면 스크립터블 오브젝트 객체가 프로젝트 내에 생성된다.

<img width="301" alt="image" src="https://github.com/novicehog/comments/assets/131991619/65b8f5d2-51ff-4747-8afe-ee3e636c9abc">

<img width="74" alt="image" src="https://github.com/novicehog/comments/assets/131991619/e51819b9-6f81-4888-a285-cec566066853">

<img width="293" alt="image" src="https://github.com/novicehog/comments/assets/131991619/e9521c2e-b014-4921-9f6a-2b3ca8caae54">


## 사용
생성된 스크립터블 오브젝트 객체를 가져와 접근하여 각 요소들을 불러올 수 있다. <br>
여기서는 Branch이 1인 데이터들을 불러와서 출력한다.

```cs
public class Test : MonoBehaviour
{
    [SerializeField]
    EntityDBDialog mySheet;
    void Start()
    {
        // 열 개수만큼 반복
        for (int i = 0; i < mySheet.Entities.Count; i++)
        {
            // branch(분기)가 1인 데이터 들만 사용
            if (mySheet.Entities[i].Branch == 1)
            {
                print($"이름 : {mySheet.Entities[i].Name}  하고 싶은 말 : {mySheet.Entities[i].Dialog}");
            }
        }   
    }
}

```


<img width="277" alt="image" src="https://github.com/novicehog/comments/assets/131991619/a5bba167-d955-409d-aa25-c796686ad677">