---
layout: single
title:  "유니티 ObejctPool"
categories: 
    - Unity
tag: [Unity, Optimization]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-03-18
last_modified_at: 2024-03-18

---

# 오브젝트 풀링
게임에서 흔히 총알 같은 것들은 발사할 때 생성되고, 특정 시간 후 또는 무언가에 닿았을 때 삭제된다.

이때 너무 잦은 생성과 삭제를 반복할 경우 크게 `두 가지 문제가 발생`한다.

- 잦은 GC(가비지 컬렉터)의 호출로 게임의 성능이 낮아진다.
- 메모리 파편화로 인해 잉여 공간이 많아진다.

이러한 문제를 해결하기 위해서 고안된 방법이 오브젝트 풀링이다.

오브젝트 풀링은 간단하다.<br>
생성과 삭제를 반복하는 것이 아니라, `미리 어느정도 오브젝트들을 생성`하여 쌓아놓은 뒤 <br>
필요할때 마다 사용하고 다시 반납하는 방식으로 사용하는 것이다.

미리 어느정도 만들어두기 때문에 맨 처음에는 메모리를 기존보다 많이 사용하긴 하나 <br>
쌓아놓은 오브젝트들은 나중에 재사용이 확실하기 때문에 GC호출이 없고, 따로 삭제하지도 않으므로 메모리 파편화도 발생하지 않는다.


## 유니티의 오브젝트 풀링
유니티에서는 오브젝트 풀링 기능을 지원한다.

using UnityEngin.Pool을 통해 사용할 수 있으며 IObjectPool<T> 를 통해 풀을 생성한다.<br>
IObjectPool은 초기화 할 때 `네 가지의 함수`를 함께 넣어줘야 한다. (OnCreate, OnGet, OnRelease, OnDestory)
이 함수들을 통해 풀링 오브젝트의 `생성, 사용, 반환, 삭제`를 구한현다.
이 네 가지 함수만 구현해 놓으면 유니티에서 지원하는 기능을 통해 풀의 오브젝트 개수가 `유동적으로 변환`된다. (ex 현재 사용해야할 총알수가 풀이 가진 오브젝트보다 많을 시 풀 오브젝트를 더 생성)


## 코드
다음은 오브젝트 풀링의 기초적인 코드이다.

```cs
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Pool;

class Pool
{
    GameObject _prefab;
    IObjectPool<GameObject> _pool;

    Transform _root;
    Transform Root
    {
        get
        {
            if(_root == null)
            {
                GameObject go = new GameObject() { name = $"{_prefab.name}Root" };
                _root = go.transform;
            }
            return _root;
        }
    }


    public Pool(GameObject prefab)
    {
        _prefab = prefab;
        _pool = new ObjectPool<GameObject>(OnCreate, OnGet, OnRelease, OnDestroy);
    }

    // 사용하기
    public void Push(GameObject go)
    {
        _pool.Release(go);
    }

    // 꺼내기
    public GameObject Pop()
    {
        return _pool.Get();
    }

    // 풀링 안에 넣을 네 가지 함수
    #region Funcs   

    GameObject OnCreate()
    {
        GameObject go = GameObject.Instantiate(_prefab);
        go.name = _prefab.name;
        go.transform.parent = Root;

        return go;
    }
    void OnGet(GameObject go)
    {
        go.SetActive(true);
    }
    void OnRelease(GameObject go)
    {
        go.SetActive(false);
    }
    void OnDestroy(GameObject go)
    {
        GameObject.Destroy(go);
    }

    #endregion
}
```


위의 Pool클래스를 통해 풀링 매니저를 만들 수 있다. Pool이 다 만들어져 있기 때문에
간단하게 갖다 쓰기만 하면 된다. 이번에는 PoolManager를 통해서 모든 풀들을 한 번에
관리하여 사용하도록 한다.

```cs
public class PoolManager
{
    // 풀들을 담아둘 딕셔너리 변수
    Dictionary<string, Pool> _pools = new Dictionary<string, Pool>(); 

    // 반환
    public bool Push(GameObject go)
    {
        if (_pools.ContainsKey(go.name) == false)
            return false;

        _pools[go.name].Push(go);
        return true;
    }

    // 사용
    public GameObject Pop(GameObject prefab)
    {
        if(_pools.ContainsKey(prefab.name) == false)
        {
            // 기존에 사용중인 풀이 아니라면 새로 생성 
            CreatePool(prefab);
        }

        return _pools[prefab.name].Pop();
    }

    // 새로운 종류의 풀 생성
    void CreatePool(GameObject prefab)
    {
        Pool pool = new Pool(prefab);
        _pools.Add(prefab.name, pool);
    }
}
```


