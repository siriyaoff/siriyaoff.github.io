---
layout: single
title: "HTML `<img>`와 CSS `background` property 차이"
categories:
  - Dev
tags:
  - HTML
  - CSS
---
# HTML `<img>` 태그를 사용해서 img 삽입
CSS:  
```css
img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.box {
  width: 400px;
  height: 200px;
  border: 5px solid black;
}
```

HTML:  
```html
<div class="box">
  <img src="balloons.jpg" alt=balloons">
</div>
```

Result:  
![html-img-tag](https://github.com/siriyaoff/siriyaoff.github.io/blob/master/_posts/img/html-img-tag.PNG?raw=true)

# CSS `background` property를 사용해서 img 삽입
CSS:  
```css
.box {
  width: 400px;
  height: 200px;
  border: 5px solid black;
  background-image: url(balloons.jpg);
  background-size: cover;
}
```

HTML:  
```html
<div class="box">
</div>
```

Result:  
![css-background-property](https://github.com/siriyaoff/siriyaoff.github.io/blob/master/_posts/img/css-background-property.PNG?raw=true)

# 차이점
둘 다 이미지의 aspect ratio는 유지되지만 노출되는 부분이 다르다.
- `<img>`의 경우 이미지의 position 초기값이 center,<br>`background` property의 경우 배경화면이기 때문에 초기값이 0 0으로 설정되어서 그렇다.
- `<img>`의 position은 `object-position` property를 이용해서 수정 가능하고,<br>`background`의 position은 `background-position` property를 이용해서 수정 가능하다.

즉, 위의 두 가지 방법에서 같은 결과가 나오게 하려면  
첫 번째에서 `img` 안에 `object-position: 0 0;`을 추가하거나,  
두 번째에서 `.box` 안에 `background-position: center;`를 추가하면 된다.

semantics를 고려하면 두 개를 맞춰야 할 상황이 잘 없을 것 같긴 하다.

그림은 [MDN](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Images_tasks){:target="_blank"} 참고