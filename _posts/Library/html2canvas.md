---
layout: Library
title:  "HTML2Canvas"
date:   2020-04-21 20:00:00 +0900
categories: HTML2Canvas
---


## HTML2Canvas

JavaScript 기반 Screenshots Library [**HTML2Canvas**](https://html2canvas.hertzen.com/)를 알아보자
우선 html2canvas는 말그대로 화면 스크린샷을 찍어주는 라이브러리다.


#### 어떤 이유로 사용?
우선 글쓴이는 대시보드를 다루는 주니어 개발자로 DOM + 차트를 캡처하기 위해 사용했다.

html2canvas를 사용하고자 하는 사람은 특정 페이지를 저장 또는 공유하기 위해 사용할거라 예상한다.

html2canvas를 사용하는 이유라 한다면 프로덕트마다 각기 다른 이유가 있겠지만, 개인적으로 이러한 이유로 사용했다는 것을 공유하고자 한다.
- 이미지에 일정한 여백으로 찍기 (대충, 여백을 좌우대칭으로 찍을 수 있다는 말)
  + ex) 이미지를 찍을 떄 바깥 흰(배경색)이 안나오게, 혹은 상하좌우 같은 크기의 여백으로 찍을 떄
- *DOM을 이미지로 찍을 수 있다
  + Data Table, Chart 등의 원하는 화면을 찍을 수 있다
  + **제목, 설명, 다른 image 등을 추가해서 찍을 수 있다
- 여러 사람이 찍어도 동일한 이미지 크기, 위치, 여백으로 찍을 수 있다
- **offscreen(화면 밖)의 영역까지 찍을 수 있다

개인적으로 '**' 표시한 부분이 html2canvas를 사용하게된 가장 큰 이유다.


#### 다른 라이브러리와 비교
알다시피 구글링하면 많은 사람들이 사용하고자하는 기능의 대표(인기있는) 라이브러리를 쉽게 찾아볼 수 있다.
스크린샷 라이브러리 중에서는 **dom-to-image**, **html2canvas**가 있다.

자비로운 마음으로 배포해주신 라이브러리를 일개 주니어 개발자 따위(?)가 감히 '이게 좋다 저게 좋다' 하는건 실례겠지만 실례하겠다.
dom-to-image도 좋은 라이브러리이지만 html2canvas 코드를 보고 바로 픽 해버렸다.. 정말 잘 만들어졌다..
는 장난이고,

먼저 **dom-to-image**를 사용해봤다.
dom-to-image은 깊게 파보지는 않았지만 일단 무난하다.
다만, 약간의 불편한 점이 있다면, DOM에 background-color 스타일을 넣어주지 않았다면 default background-color는 검은색이다..
글쓴이는 불행하게도 코드 구조상 background를 흰색으로 찍으려면 구조 변경이 필요했다.
해서 바로 html2canvas로 갈아탔다

**html2cavnas**는 위에서 말했지만 코드가 잘 만들어져있다..
기능들도 무난하게 사용할 수 있었고 큰 문제는 없었다.
다만, 일부 원하는 기능을 별도로 개선해서 사용한 부분이 있다. (이 부분은 뒤에서 어떤 기능을 어떻게 추가했는지 설명하겠다.)