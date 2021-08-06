---
title: "2021-08-06-tools-typora.md"
date: 2021-08-06 10:05:42 -0400 
categories: tool
tags: typora markdown covert pdf md 타이포라

# 목차

toc: true  
toc_sticky: true
---
# Typora를 이용한 문서 변환
- 타이포라는 markdown 작성을 쉽게 하는 도구.

## markdown 
- 마크다운을 오늘 처음들어봤다면, 
  - 마크다운은 언어이다. 가벼운 마크업언어이다.
    - 마크업 언어 : 작성한 텍스트가 화면에 어떻게 보여질지를 어떤 규칙을 정해놓은 언어.
      - 예시 : "# 텍스트"에서 #과 띄어쓰기 다음에 오는 문자는  H1 크기의 제목으로 취급된다. #개수가 늘어나면 글자 크기도 바뀐다 h6까지 표현가능하다. 
  - 문서저장형식이다. md 라는 확장자로 저장된다.
  - 마크다운은 매우 작성하기 쉽고, *툴의 도움*을 받으면 여러가지 문서형식으로 변환이 자유롭다. 한번 작성으로 여러형식으로 내보내기 가능하다.
    - html, pdf는 기본 docx, LaTex, image, epub 등 다양한 형태로 변환이 가능하다.

## Typora에서 마크다운 파일을 pdf로 내보내기
아주 쉽게 파일을 내보낼 수 있습니다.
- 초간단 내보내기 : `파일 > 내보내기 > PDF`

그러나 좀 더 형식을 갖추어 내보낼 수도 있습니다.
- 표지
- 목차 --> 본문에 [toc] 라고 적어 주기만 하면 되요(Table Of Contents)
- 페이지의 Header/Footer
- 가로로 출력

위와 같은 형식을 작성한 문서에 붙여서 pdf로 만드는 방법도 있습니다.
설정만 조금 바꿔주면 되는 것이죠.

먼저 `파일 > 환경설정 > 내보내기 > PDF`를 차례대로 선택하고 그림과 같이 설정하세요. 그림은 header와 footer내용이 비워져 있지만 간단하게 채워넣기만 하면 바로 적용이 됩니다. 이 방법은 제가 가로 출력물이 필요하여 커스텀한 내용입니다. 얼마든지 본인이 원하는 형식으로 변경할 수 있습니다. 다만, 결과는 내보내기를 해봐야 알수 있기 때문에 적절한 디자인을 적용하고자 하면 어렵습니다. 다만 html, javascript 그리고 css에 대한 지식이 있으신 분이라면 더 예쁘게 설정하실 수 있습니다.

![타이포라 pdf 가로출력 설정 가이드](https://raw.githubusercontent.com/JungMockdan/jungmockdan.github.com/gh-pages/assets/images/post/%ED%83%80%EC%9D%B4%ED%8F%AC%EB%9D%BC%20pdf%20%EA%B0%80%EB%A1%9C%EC%B6%9C%EB%A0%A5%20%EC%84%A4%EC%A0%95%20%EA%B0%80%EC%9D%B4%EB%93%9C.png)

- Append Extra Content

  문서에 정의된 `YAML front matters`이나 타이포라에서 기본적으로 제공하는 ${date}, ${title}, ${author}, ${pageNo}, ${pageCount} 와 같은 인자를 사용하여 동적으로 데이터를 생성할 수 있습니다. 물론 이 인자들은 YAML front matters를 통해 재정의 할 수 도 있습니다.

```html
<meta name="title_cover" content="${title}">
<meta name="doc_info" content="${today}. ${team}. ${author}">
<div id="_export_cover">
            <div id="_cover_title" style="margin-top: 25%;text-align: center;font-size: 6rem;"></div>
            <div id="_doc_info" style="margin-top: 15% ;text-align: right;font-size: 2rem;"></div>
        </div>
<script>
    var $cover = document.querySelector("#_export_cover");
    var title = document.querySelector("meta[name='title_cover']").getAttribute("content");
    var doc_info=document.querySelector("meta[name='doc_info']").getAttribute("content")

    document.body.insertBefore($cover, document.body.childNodes[0])
    $cover.querySelector("#_cover_title").textContent = title;
    $cover.querySelector("#_doc_info").textContent = doc_info;

</script>
```
- YAML front matters

마크다운 상단에 아래를 참고 응용하여 정의하고, 위의 html 등에서 사용할 수 있습니다. 다만 `YAML front matters`를 사용하겠다고 체크해야 합니다.
```yaml
---
title: 제목
team: 매일매일더좋은나
author: 나의 이름은 홍길동
subject: 주제는 무엇으로 할까...
keywords: [키워드1, 키워드2, Tutorial, Export]
---
```

결과물은 매우 간단해 보이지만, 수십번의 Export를 통해 만들었답니다. 이 글을 보시는 분이 유용하게 쓰시기를 바랄 뿐입니다. 아직, 페이지 구분이라던가 하는 부분이  Export시에 아주 매끄럽지만은 않은 부분도 있는것이 사실 입니다. (Typora 팀이 서둘러 해결해주기를....) 하지만 그런 부분은 사소하다고 생각합니다. 충분히 좋은 툴입니다. 그럼 Typora로 효율적인 업무에 도움이 되시길 바랍니다.

