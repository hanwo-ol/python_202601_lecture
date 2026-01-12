# 왜 이미지가 렌더링(표시가) 안되는걸까?

## 1. HTML의 기본 골격 (The Skeleton)

HTML 문서는 크게 '문서 정보'를 담는 부분과 '실제 화면'을 보여주는 부분으로 나뉩니다.

* **`<!DOCTYPE html>`**: "이 문서는 HTML5 표준으로 작성되었습니다"라고 브라우저에게 알리는 선언문입니다.
* **`<html lang="ko">`**: 웹 페이지의 시작과 끝을 나타내며, `lang="ko"`는 이 페이지의 주 언어가 한국어임을 명시합니다. (검색 엔진과 스크린 리더에게 중요합니다.)
* **`<head>`**: 브라우저 화면에는 보이지 않지만, 페이지의 제목, 글꼴 설정, 스타일(CSS) 등 **메타 정보**를 담는 머리 부분입니다.
* **`<body>`**: 사용자의 눈에 실제로 보이는 **모든 콘텐츠**(글, 이미지, 버튼 등)가 들어가는 몸통 부분입니다.

---

## 2. 레이아웃을 위한 시맨틱 태그 (Semantic Tags)

| 태그 | 역할 | 코드 내 사용 예시 |
| --- | --- | --- |
| **`<header>`** | 도입부나 제목, 메뉴를 포함하는 상단 영역 | 드라마 제목과 기본 정보(방송사 등) |
| **`<section>`** | 문서의 독립적인 주제별 영역 | 줄거리 영역, 등장인물 영역 |
| **`<footer>`** | 문서 하단의 저작권 정보나 연락처 영역 | `© SBS Drama Try Info Page` |
| **`<div>`** | 특별한 의미 없이 콘텐츠를 묶어주는 컨테이너 | 캐릭터별 이미지와 텍스트를 묶는 용도 |

---

## 3. 세부 콘텐츠 태그 설명

각 구역 안에서 데이터를 표현하기 위해 사용된 태그들의 기능입니다.

### 텍스트 관련 태그

* **`<h1>` ~ `<h3>**`: 제목(Heading)을 나타냅니다. 숫자가 작을수록 중요도가 높고 글자가 큽니다.
* **`<p>`**: 단락(Paragraph)을 의미하며, 일반적인 본문 텍스트를 쓸 때 사용합니다.
* **`<ul>` & `<li>**`: 순서가 없는 목록(Unordered List)을 만듭니다. 드라마 정보(방송사, 장르 등)를 나열할 때 사용되었습니다.
* **`<strong>`**: 텍스트를 굵게 강조하며, 중요한 내용임을 의미론적으로 나타냅니다.

### 멀티미디어 및 링크

* **`<a href="..." target="_blank">`**: 하이퍼링크를 만듭니다. `target="_blank"`는 링크를 클릭했을 때 **새 탭**에서 열리게 합니다.
* **`<img src="..." alt="...">`**: 이미지를 삽입합니다.
* `src`: 이미지 파일의 경로(주소)
* `alt`: 이미지가 보이지 않을 때 대신 나오는 설명(시각장애인을 위한 접근성 필수 요소)


---

## 4. 디자인을 입히는 `<style>` (CSS)

`<head>` 태그 안에 위치한 `<style>`은 HTML 요소들이 어떻게 보일지 결정합니다.

* **`.class-name`**: HTML에서 `class="..."`로 지정한 요소들을 선택하여 스타일을 입힙니다. (예: `.character`, `.drama-header`)
* **`display: flex;`**: 요즘 웹 디자인에서 가장 중요한 속성 중 하나로, 요소들을 가로 혹은 세로로 자유롭게 배치하게 해줍니다. (캐릭터 이미지와 설명을 가로로 배치할 때 사용됨)


```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>SBS 드라마 - 트라이(Try) 소개</title>
    <style>
        body { font-family: sans-serif; line-height: 1.6; max-width: 800px; margin: 0 auto; padding: 20px; }
        .drama-header { border-bottom: 2px solid #333; padding-bottom: 10px; }
        .section-title { background-color: #f4f4f4; padding: 5px 10px; margin-top: 20px; }
        .character { display: flex; align-items: flex-start; margin-bottom: 20px; border-bottom: 1px solid #ddd; padding-bottom: 15px; }
        .character-img { margin-right: 20px; }
        .character-img img { width: 150px; border-radius: 8px; } /* 이미지 크기 조절 */
        .character-txt h3 { margin-top: 0; color: #0056b3; }
        .synopsis { font-style: italic; color: #555; }
    </style>
</head>
<body>

    <header class="drama-header">
        <h1>트라이 (Try)</h1>
        <ul>
            <li><strong>방송사:</strong> SBS</li>
            <li><strong>장르:</strong> 휴먼, 스포츠, 청춘</li>
            <li><strong>주요 내용:</strong> 꼴찌 한양체고 럭비부와 괴짜 감독의 뜨거운 도전</li>
            <li><a href="https://programs.sbs.co.kr/drama/try/main" target="_blank">공식 홈페이지 바로가기</a></li>
        </ul>
    </header>

    <section>
        <h2 class="section-title">줄거리</h2>
        <p class="synopsis">예측불허 괴짜 감독 주가람과 만년 꼴찌 한양체고 럭비부의 좌충우돌 성장기.</p>
    </section>

    <section>
        <h2 class="section-title">등장인물</h2>

        <div class="character">
            <div class="character-img">
                <img src="https://img2.sbs.co.kr/img/sbs_cms/WE/2025/07/14/Dzo1752466948899-666-968.jpg" alt="베이지 이미지">
            </div>
            <div class="character-txt">
                <h3>베이지 (37세, 여)</h3>
                <p><strong>"주가람! 이번에는 반드시 너로부터 내 멘탈을 지키고 말 거다. "</strong></p>
                <p>01.점 차이로 전국체전 4위, 4위까지 선발되는 국대 선발전은 5위. 포기하기에는 너무 잘하고 메달을 따기에는 부족한 말 그대로 애매한 재능의 저주를 받은 선수. 그렇지만 전체 국가대표를 통틀어 가장 끈질긴 사람을 꼽으라면 아마 99.99퍼센트의 사람들은 배이지를 떠올릴 거다. 0.01이 남은 건 배이지가 스스로를 뽑지 않았기 때문이고.</p>
            </div>
        </div>

        <div class="character">
            <div class="character-img">
                <img src="https://img2.sbs.co.kr/img/sbs_cms/WE/2025/07/14/GJK1752466916491-666-968.jpg" alt="주가람 이미지">
            </div>
            <div class="character-txt">
                <h3>주가람 (37세, 남)</h3>
                <p>리트리버와 도베르만 사이 그 어딘가.  럭비선수 치곤 상당히 온순한 얼굴을 하고 있다. 헤실헤실 웃는 얼굴을 타고난 편. 근데 경기장에만 들어가면 종이 바뀐다. 눈에 번들번들 광기가 돈다. 리트리버는 주인을 지키지만, 도베르만은 적을 물어뜯는 것처럼.</p>
            </div>
        </div>
    </section>

    <footer>
        <p>&copy; SBS Drama Try Info Page</p>
    </footer>

</body>
</html>
```
