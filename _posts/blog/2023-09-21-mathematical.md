---
layout: single
title:  "minimal mistakes 수학식 사용하기"
categories: 
    - blog
tag: [blog, minimal mistakes]
published: true

author_profile: true # 옆에뜨는 프로파일

date: 2023-09-21
last_modified_at: 2023-09-21
---


# 수학식 사용하기
<br/>
makrdown에서는 \$ 을 통해서 수학식을 표현한다. 

하지만 minimal mistakes는 기본적으로 수학식 기능이 제공되지 않는다. <br>
그렇기 떄문에 직접 추가해줘야함.

`_includes/head/custom.html`로 가서 다음 코드를 추가해줌

```html
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
```


이렇게 해도 식이 되지 않는다면<br>
`_config.yml` 파일에 맨 아래에 다음과 같이 추가해줌

```yaml
mathjax: true
```


<br>
이렇게 되면 정상작동 하는 것을 볼 수 있음

```
$$
 \pi = 180 ^{\circ}
 \tau = 360 ^{\circ}
$$
```
$$
 \pi = 180 ^{\circ}
 \tau = 360 ^{\circ}
$$

## `$...$` 로 문장 사이에 수식표현

이때 `$$ ... $$`을 통해 수학식을 쓰는것은 잘 작동하나 <br>
`$...$`를 써서 문장 도중에 수학식을 쓰는 것은 잘 작동하지 않음 이 경우에는 <br>

`$...$` 대신 `\\(...\\)` 를 사용하거나

`_includes/head/custom.html` 에 다음 코드를 추가해주면 된다.

```html
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$', '$'], ['\\(', '\\)']]
  }
});
</script>
```

<br>

잘 $3 + 3 = 6$ 작동한다.