---
layout: post
title:  "HTML2Canvas"
date:   2020-04-23 20:00:00 +0900
categories: HTML2Canvas
---


## HTML2Canvas

JavaScript 기반 Screenshots Library [**HTML2Canvas**](https://html2canvas.hertzen.com/)를 알아보자

글쓴이는 1년차 주니어 개발자로 라이브러리를 완벽하게 이해하진 못하지만,
나 같은 주니어 개발자가 html2canvas를 사용할 때 참고하면 도움이 되지 않을까 싶어서 이 글을 작성한다.

일단 라이브러리에 대한 설명은 [공식문서](https://html2canvas.hertzen.com/configuration)나 html2canvas 사용자가 쓴 [블로그](http://definitelytyped.org/docs/html2canvas--html2canvas/interfaces/html2canvas.html2canvasoptions.html)를 보고,
이 글은 기능을 적용햔 방식만 참고하자
글쓴이의 작업 스펙과 비슷한 사람이 보면 좋을 듯 하다.

#### 작업 스펙
- 크롬(Chrome)
- React
- Typescript (이 글에선 거의 안나올거다)
- file-saver (이미지 저장을 위해)

#### 어떤 이유로 사용?
DOM + 차트를 캡처하기 위해 사용했다.

아마 html2canvas를 사용하고자 하는 사람은 이미지가 아닌 특정 화면을 이미지로 저장 또는 공유하기 위해 사용 할 것이라 예상한다.

사용하는데 무슨 이유가 중요한가. 다만 이러한 이유로 사용했다는 것을 공유하고자 한다.
('이유' 보단 이러한 기능을 원했고, 뒤에서 어떻게 구현했는지만 참고하면 될 것 같다)

- 스크린샷 영역을 직접 맞출 필요가 없다
- Image 외에 *DOM을 이미지화 할 수 있다
  + 제목, 설명, 이미지를 넣어 함꼐 이미지화 할 수 있다
- offscreen(화면 밖)의 영역까지 찍을 수 있다


#### 다른 라이브러리는?
ScreenShot 라이브러리는 html2canvas, dom-to-image, rasterizeHTML, html2png 등이 있고,
그 중에서 대표적으로 많은 사람들이 사용하는 두 가지 **html2canvas**, **dom-to-image**를 사용해봤다.

자비로운 마음으로 배포해주신 라이브러리를 일개 주니어 개발자 따위(?)가 감히 '이게 좋다 저게 좋다' 하는건 실례겠지만 실례하겠다.
dom-to-image도 좋은 라이브러리이지만 html2canvas 코드를 보고 바로 픽 해버렸다.. 정말 잘 만들어졌다
는 장난이고,

사실 둘 다 사용하기 나름이라 뭐가 더 좋고 나쁘고는 판단하기는 어렵지만
dom-to-image에 조금 아쉬운 점이 있다면,
DOM에 background-color 스타일을 넣어주지 않으면, default background-color는 검은색이라는 것이다.
글쓴이는 불행하게도 적용하려는 프로덕트의 구조상 background-color를 흰색으로 찍으려면 구조 변경이 필요했다.
일을 키우고 싶지 않아서 바로 html2canvas로 환승했다.

**html2cavnas**는 위에서 말했지만 잘 만들어져있다..
기능들도 무난하게 사용할 수 있었고 역시 별 다른 큰 문제는 없었다.
다만, 있지만 동작하지 않는 기능이 있고, 이를 해결하기 위해 라이브러리를 커스텀해야했다. (어떤 기능을 어떻게 추가했는지는 뒤에서 설명하겠다)
이 문제는 아마 다른 라이브러리에서도 비슷할 것 같다. (개인적인 추측이다)


### 기능 구현

ScreenShot을 찍을 DOM이다.
```js
<div ref={captureRef}>
  이 안을 스크린샷 찍을거임
  ┌---------------------------┐
  |                   =       |
  |              =    =       |
  |      =       =    =    =  |
  |  =   =   =   =    =    =  |
  | ------------------------- |
  |  1   5   3   13   18   8  |
  └---------------------------┘
  ( 뭐 이런 차트가 있다고 가정하자 )
</div>
```

( 여기부터 코드는 파트별로 나누되, 이어지도록 쓰겠다 )


```js
// 이미지를 다운로드할 때 사용할거다
import { saveAs } from 'file-saver';

const capture = ({
  targetElement = captureRef.current, // 캡쳐할 영역
  captureType = 'copy' || 'export', // 이미지를 복사 or 다운로드
}) => {
```

> targetElement를 clone하지 않아도 캡쳐는 할 수 있다.
> 다만, 추가하고 싶은 Div를 appendChild 하면 실제 프로덕트에 title이 노출되버린다.
> 글쓴이는 그게 싫고, 캡쳐할 때만 title을 넣어주기 위해 clone 했다.

이 부분은 사용자 스타일로 커스텀하면 된다.
```js
  const targetClone = targetElement.cloneNode(true); // targetElement를 clone
  const captureArea = document.createElement('div'); // title과 clone할 targetElement를 옮길 영역
  const captureWrapper = document.createElement('div'); // 캡쳐할 targetElement를 여기 넣어 줄거다

  const titleDiv = document.createElement('div'); // title 쓰여질 Div
  const title = `HTML2Canvas 사용 후기`; // title 내용
  titleDiv.innerText = title; // title => div 넣어줌

 /*
  * Capture position
  * 아래 Style은 캡쳐할 때 문제가 없다면 그냥 무시해라
  * 글쓴이는 image 윗 부분에 여백이 생겨서 이미지 아래 부분이 잘리는 이슈가 있어서
  * position을 잡아주고 캡쳐했다
  */
  Object.assign(captureAreaWrapper.style, {
    position: 'fixed',
    top: '-9999px',
    left: '-9999px',
  });

  // Element 조립!
  captureArea.appendChild(titleDiv);
  captureArea.appendChild(targetClone);
  captureWrapper.appendChild(captureArea);
  document.body.appendChild(captureAreaWrapper);
```
마지막에 document.body에 `captureAreaWrapper`를 넣어준다 (맨 마지막에 제거해주자)


이 부분은 참고로만 보면 될거다.
이미 라이브러리 내에 스크롤 값을 처리해주는 로직이 있는데,
clone하면 스크롤값이 없어져버려서 직접 구현헤서 넣어주어야 했다.
```js
  // 가로 혹은 세로로 스크롤이 있어서 화면에 나오지 않고 가려진 영역이 있는 Element를 뽑을거다
  const scrollElements = {};

  function extractScrollElement(element) {
    // child Element가 없을 때까징 
    if (element.hasChildNodes) {
      element.childNodes.forEach((child) => extractScrollElement(child));
    }

    // element에 스크롤 값이 존재하면 그 element의 스크롤 값을 뽑아준다
    if (element.scrollLeft || element.scrollTop) {
      const scrollElement = element.className.split(' ').map(c => `.${c}`).join('');

      return Object.assign(scrollElements, {
        [scrollElement]: {
          x: element.scrollLeft || 0,
          y: element.scrollTop || 0,
        },
      });
    }

    return scrollElements;
  }

  const scrollPositions = extractScrollElement(targetElement);
```


html2canvas Option을 변수로 만들어서 주었다.

```js
  // Capture Width
  // width는 스크롤이 생긴 Element의 width 값을 넣어주자
  const captureWidth = targetElement.getElementsByClassName('???').clientWidth;

  const options = {
    logging: false,
    width: captureWidth,
    windowWidth: captureWidth,
    scale: 1, // scale = 이미지 퀄리티
  };
```


html2canvas는 png, jpg, pdf 파일 형식을 지원해준다.
이 글에서는 png만 구현했다

```js
  if (captureType === 'copy') {
    html2canvas(
      captureAreaWrapper, // 캡쳐할 Element
      options, // 위에서 설정해준 option
    ).then((canvas) => { // 라이브러리에서 Element를 canvas로 만들어서 내보낸다

      // 이미지를 Clipboard에 복사하기 위해 blob을 해준다
      canvas.toBlob((blob) => {
        // lint가 화낼 순 있지만 기능엔 문제없다
        navigator.clipboard.write([
          new ClipboardItem({
            'image/png': blob, // image 형식은 png
          }),
        ]);
      }, 'image/png', 1); // 1은 이미지의 퀄리티이다
    }).catch((error) => {
      console.error('Image copy failed', error);
    });
  } else if (captureType === 'download') {
    // 이미지 다운로드
    html2canvas(
      captureAreaWrapper,
      options,
    ).then((canvas) => {
      const fileName = 'Mago_html2canvas';

      saveAs(canvas.toDataURL('image/png'), `${}.png`);
    }).catch((error) => {
      console.error('Image Download Failed', error);
    });
  }
```


이미지 캡쳐 후 body에 넣어주었던 Element를 제거해줘야하는데,
setTimeout을 준 이유는
이미지가 만들어지고 캡쳐하기까지의 시간이 걸리고, 그 마지막에 캡쳐되기 전에 캡쳐할 Element가 제거되면
'Document is not focused.'라는 console 에러를 뿜어서 제거될 시간을 정해주었다.

Tip. 이미지가 만들어지는 시간은 `logging: true` 로 하면 콘솔창(log level: verbose)에서  
이미지가 만들어지고 캡쳐까지 걸리는 시간을 볼 수 있다.
 \+ 물론 이미지 크기가 클 수록 오래 걸린다.

```js
  // Download or Copy 된 후 remove element
  setTimeout(() => {
    captureAreaWrapper.blur();
    document.body.removeChild(captureAreaWrapper);
  }, 700); // 글쓴이는 0.7초 조금안되게 걸렸다
};
```