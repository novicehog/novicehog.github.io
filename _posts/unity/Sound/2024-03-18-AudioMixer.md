---
layout: single
title:  "유니티 AudioMixer"
categories: 
    - Unity
tag: [Unity, Sound]
published: true

author_profile: true # 옆에뜨는 프로파일

search: true #검색해도 안뜨게함

date: 2024-03-18
last_modified_at: 2024-03-18

---

# 오디오 믹서

유니티를 사용하다 보니 오디오 관련해서 한 가지 고민이 생겼다.

> Audio Source가 여러 오브젝트에 흩어져 있으면 사운드 조절을 어떻게 한번에 하지?

입체적인 사운드를 위해 Spatial Blend값을 완전히 3D로 해놓았기 때문에 한 오브젝트에서 Audio Source 컴포넌트를 관리하는 것이 아닌<br>
소리를 내는 객체가 모두 Audio Source 컴포넌트가 필요했다.

그래서 Auido Source 컴포넌트를 가진 모든 오브젝트를 생성할 때부터 한 배열에 모아두어야 하나? 하고 생각했다. <br>
하지만, 이러한 문제는 유니티에서 제공하는 Audio Mixer를 통해서 손쉽게 해결이 가능했다.

## 오디오 믹서 만들고 등록
오디오 믹서는 프로젝트 창에서 생성해야 한다.

![오디오 믹서 생성](https://github.com/novicehog/comments/assets/131991619/2a2dca01-6820-4705-9bd7-f92fea92d915)
<br>

이렇게 오디오 믹서를 생성하면 다음과 같이 생겼는데.

![오디오 믹서 아이콘](https://github.com/novicehog/comments/assets/131991619/a3a84942-d0a0-45b6-ab7c-61cc8161008b)
<br>

더블 클릭하여 이러한 오디오 믹서 창을 볼 수 있다.
![오디오 믹서 창](https://github.com/novicehog/comments/assets/131991619/e93e5453-ddec-4242-82bd-4e4ca64aa89f)
<br>


여기서 이제 Group을 생성하여 Sound 종류별로 볼륨설정이 가능하다. <br>
나는 Music(배경음)과 SFX(효과음) 두 종류의 그룹을 만들어 사용하려한다.

![그룹 생성](https://github.com/novicehog/comments/assets/131991619/c29dcd7f-b991-4603-9790-26b361abdb4b)

<br>


이렇게 생성된 그룹은 Audio Source 컴포넌트에서 등록이 가능하다.<br>
나는 그래서 AudioSource를 가진 프리팹마다 종류에 따라 AuidoMixer를 추가하였다.

![오디오 믹서 등록](https://github.com/novicehog/comments/assets/131991619/dcdd4398-0c94-4017-bec0-c67a30a6d259)
<br>

여기까지 했다면 나는 이제 `흩어진 오디오 소스`들을 `한 번에 관리`할 수 있는 오디오 믹서가 완성된 것이다.<br>
이제는 이 오디오 믹서를 통해 `한 번에 볼륨조절`을 할 수 있다.

## 간단한 볼륨조절
스크립트 상에서 이제 어떻게 볼륨을 설정하는지를 말하겠다.

만든 그룹중에 하나를 누르고 인스펙터 창을 보면 pitch나 volume등의 값을 설정할 수 있다.<br>
여기서 원하는 값을 파라미터로 만들어서 스크립트에서 조작할 수 있다.

![파라미터등록](https://github.com/novicehog/comments/assets/131991619/1c009851-d727-4bc1-94f0-92248f20356a)
<br>

이러면 이제 오디오 믹서창 오른쪽 위에 파라미터가 하나 생기게 되는데 <br>
이때 이 이름을 원하는대로 재설정한 뒤 기억해둔다

![파라미터 이름](https://github.com/novicehog/comments/assets/131991619/242cc747-1d0c-47b4-a54c-d54eddfde4c9)
<br>

나는 MusicVolume과 SFXVolume으로 하였다.

![파라미터 이름 설정](https://github.com/novicehog/comments/assets/131991619/abff7237-9ec5-446f-9f18-ea8c4abc550c)

### 스크립트 조작
UI생성은 간단하니 넘어가고 바로 스크립트에서 변수를 조작해보겠다.<br>
코드는 다음과 같다.

```cs
using UnityEngine;
using UnityEngine.Audio;
using UnityEngine.UI;

public class VolumeUI : MonoBehaviour
{
    public AudioMixer _mixer;                   // 만들어둔 오디오 믹서를 넣어놓는다.

    const string MIXER_MUSIC = "MusicVolume";   // 위에서 설정해놓은 파라미터 이름들이다
    const string MIXER_SFX = "SFXVolume";

    public Slider musicSlider;                  // UI는 간단하니 Slider를 만들고 등록해두었다.
    public Slider SFXSlider;

    private void Awake()
    {
        musicSlider.onValueChanged.AddListener(SetMusicVolume);
        SFXSlider.onValueChanged.AddListener(SetSFXVolume);
    }

    private void SetMusicVolume(float value)
    {
        // 오디오믹서에 SetFloat함수를 통해 값을 변경할 수 있다.
        // Log10을 사용하는 이유는 slider Value는 0~1로 고정적인 간격을 가지지만
        // AudioMixer의 Volume 값은 데시벨이기 때문에 고정적인 간격을 가지지 못함.
        // 추가로 0의 값이 들어가면 0이 되기 때문에 슬라이더의 최소 값을 0.0001정도로 설정해 두는것이 좋음
        _mixer.SetFloat(MIXER_MUSIC, Mathf.Log10(value) * 20);
    }

    private void SetSFXVolume(float value)
    {
        _mixer.SetFloat(MIXER_SFX, Mathf.Log10(value) * 20);
    }
}
```
<br>



### 볼륨값 저장 추가
볼륨 사운드 경우에는 간단하게 유니티에서 제공하는 PlayerPrefs을 통해 저장하면 된다.

코드는 다음과 같다.

```cs
using UnityEngine;
using UnityEngine.Audio;
using UnityEngine.UI;

public class VolumeUI : MonoBehaviour
{
    public AudioMixer _mixer;

    const string MIXER_MUSIC = "MusicVolume";
    const string MIXER_SFX = "SFXVolume";

    const string k_MusicVolumeKey = "SaveMusicVolume";
    const string k_SFXVolumeKey = "SaveSFXVolume";

    public Slider musicSlider;
    public Slider SFXSlider;

    private void Start()
    {
        SetMusicVolume(PlayerPrefs.GetFloat(k_MusicVolumeKey));
        SetSFXVolume(PlayerPrefs.GetFloat(k_SFXVolumeKey));
    }
    private void OnEnable()
    {
        musicSlider.value = PlayerPrefs.GetFloat(k_MusicVolumeKey);
        musicSlider.onValueChanged.AddListener(SetMusicVolume);

        SFXSlider.value = PlayerPrefs.GetFloat(k_SFXVolumeKey);
        SFXSlider.onValueChanged.AddListener(SetSFXVolume);
    }

    private void OnDisable()
    {
        musicSlider.onValueChanged.RemoveListener(SetMusicVolume);
        SFXSlider.onValueChanged.RemoveListener(SetSFXVolume);
    }

    private void SetMusicVolume(float value)
    {
        PlayerPrefs.SetFloat(k_MusicVolumeKey, value);
        _mixer.SetFloat(MIXER_MUSIC, Mathf.Log10(value) * 20);
    }

    private void SetSFXVolume(float value)
    {
        PlayerPrefs.SetFloat(k_SFXVolumeKey, value);
        _mixer.SetFloat(MIXER_SFX, Mathf.Log10(value) * 20);
    }
}
```