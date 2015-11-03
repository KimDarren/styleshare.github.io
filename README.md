# StyleShare 기술 블로그

## 포스트 작성 방법

1. `_post` 디렉토리 밑에 [마크다운(Markdown)](http://daringfireball.net/projects/markdown/) 문법으로 포스트를 작성합니다.
2. 파일명은 `[yyyy-mm-dd-제목.md]` 으로 작성합니다.
3. 파일의 최상단에 아래의 템플릿을 추가하여 작성하고, 그 아래에 글 본문을 작성합니다.

### 포스트 템플릿

  ```
  ---
  layout: post
  title: 글 제목
  author: 작성자 이름
  author-email: 작성자 이메일
  description: 간단한 설명
  publish: true / false
  ---
  ```

#### 참고사항
* `author-email`을 작성하면 작성자 정보에 이메일 하이퍼링크(`mailto:`)가 걸립니다.
* `publish`는 `true`일 경우에 바로 리스트에 보이고, `false`인 경우에 보이지 않습니다.

### 이미지
* `img` 폴더에 추가하여 참조하기로 합니다. (예를 들면, `/img/apple.jpg`로 저장)
