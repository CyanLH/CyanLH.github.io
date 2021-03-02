---
title: AntDesign InputNumber에서 formatter에 numeral을 사용하면 안된다
date: 2021-03-03 01:03:70
category: react
thumbnail: { thumbnailSrc }
draft: false
---

Ant.Design의 InputNumber 컴포넌트에 숫자를 `10,000`이나 처럼 표시하려 했다.<br/>
처음엔 간편히 `numeral` 라이브러리를 활용할 생각으로

```jsx
<InputNumber
    formatter={value => numeral(value).format('0,000')}
/>
```

처럼 사용했지만 `backspace`를 누르면 숫자 단위가 늘어나고 소수점 이하 자리 입력이 불가능 한 등 골치아픈 버그가 생겼다.

---

다음과 같이 정규표현식을 사용해 해결

```jsx
<InputNumber
    formatter={value => `${value}`.replace(/\B(?=(\d{3})+(?!\d))/g, ',')}
    parser={value => value.replace(/\$\s?|(,*)/g, '')}
/>
```
사실 Ant.Design 문서만 잘 살펴도 이런 일이 없었을텐데 완전 불찰이었다.<br/>
그리고 새삼 정규표현식에 대한 공부가 더 필요하다고 느꼈다. 정규표현식에 대한 정리를 블로그에 포스팅 해도 좋을것 같음.
