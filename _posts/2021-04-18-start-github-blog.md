---
title:  "github 블로그 시작."
excerpt: "Github 을 이용하여 블로그를 시작해보자. "

categories:
  - Blog
tags:
  - [Blog, jekyll, Github, Git]

toc: true
toc_sticky: true
 
date: 2021-04-18
last_modified_at: 2021-04-18
---


# 사전준비.
* 여러가지테마중 https://mmistakes.github.io/minimal-mistakes/ 를 골랐음. fork를 이용해서 생성하자.
  * https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/ 가이드를 따라서 잘 해보자

* 불필요 파일 삭제
  * https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#remove-the-unnecessary 에 의해 아래것들은 삭제하자
  ```
  editorconfig
  gitattributes
  github
  /docs
  /test
  CHANGELOG.md
  minimal-mistakes-jekyll.gemspec
  README.md
  screenshot-layouts.png
  screenshot.png
  ```
* 메뉴를 수정하기 위해서 _data/navigation.yml 를 살펴보자
  * 아래처럼 되어있을 것이다.   
  ```
  main:   
    \- title: "Quick-Start Guide"   
      url: https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/    
  ```

* 아래처럼 수정해보자   
  ```   
  main:
    \- title: "Home"
      url: https://brush-up.github.io/
    \- title: "Category"
      url: https://brush-up.github.io/categories/   
  ```   
  
# 글쓰기.
## 글 등록관련
* _posts 디렉토리를 생성한다
* 해당 디렉토리에 년-월-일-글제목.md 형식으로 파일을 작성한다 (ex. 2021-04-18-first-post.md)
* 해당 파일에 머릿말(Matter Defaults) 를 최상단에 작성한다
  ```
  ---
  title:  "포스트의 제목을 쌍따옴표 안에 적는다, 적지 않으면 .md파일 이름이 title로 된다."
  excerpt: "포스트의 내용을 간략히 보여주는 용도로 사용되는듯."

  categories:
   - lang
  tags:
   - [c++, windows, Git]

  toc: true
  toc_sticky: true

  date: 2020-05-25
  last_modified_at: 2020-05-25

  published: ture
  ---
  ```
 * toc : table of contents. 포스트의 헤더들만 보여주는 목차를 사용할 것인지의 여부. ture 로 해주면 포스트의 목차가 보이게 된다
 * toc_sticky : true로 해주면 목차가 스크롤을 따라 움직이게 된다
 * published : false로 하면 포스트가 비공개로 전환된다. (물론 작성자도 못본다.)

## 카테고리, 태그 관련
* 카테고리 페이지 등록을 위해 _pages 폴더를 생성한뒤 "category-archive.md" 파일을 만들어 아래처럼 사용하자
  ```
  ---
  title: "Posts by Category"
  layout: categories
  permalink: /categories/
  author_profile: true
  ---
  ```
* tag 페이지 등록을 위해 _pages 폴더에 "tag-archive.md" 파일을 만들어 아래처럼 사용하자
  ```
  ---
  title: "Posts by Tag"
  permalink: /tags/
  layout: tags
  author_profile: true
  ---
  ```
* categories 하위에 category를 등록하기위해 _pages 폴더에 categories 폴더를 생성해서(안해도 되긴함) "category-blog.md" 등으로 파일을 만들어 아래처럼 사용하자
  ```
  ---
  title: "blog 관련"
  permalink: /categories/blog/
  layout: category
  author_profile: true
  taxonomy: blog
  ---

  blog 관련이에요.
  ```
## 이미지 첨부
* 이미지 첨부를 위해 폴더 assets 하위에 "images"를 생성해서 그 안에 넣자
* 아래처럼 사용하면 된다.
```
![image]({{ site.url }}{{ site.baseurl }}/assets/images/filename.jpg)
```

# 기타
## 여백
* 작성하다보니 좌우 여백이 너무 거슬린다. _sass/minimal-mistakes/_variables.scss 파일의 아래 부분을 수정하면 된다.
```css
$right-sidebar-width-narrow: 200px !default;
$right-sidebar-width: 300px !default;
$right-sidebar-width-wide: 400px !default;
```
## 폰트
* 글 크기도 살짝 마음에 안든다 . _sass/minimal-mistakes/_reset.scss 파일의 아래 부분을 수정하자.
```css
  @include breakpoint($large) {
    font-size: 20px;
  }

  @include breakpoint($x-large) {
    font-size: 22px;
  }
```
## 검색
* _config.yml 파일중 다음을 수정하자
```
search                   : true # 사이트 우측 상단 검색 활성화
search_full_content      : true # 제목 + 내용까지도 검색할 것인지
```
## 글 제목의 밑줄 없애자
* _sass/minimal-mistakes/_base.scss 파일을 보면 아래처럼 되어있는데
```css
/* links */
a {
  &:focus {
    @extend %tab-focus;
  }
```
* 아래처럼 수정하자
```css
a {
  /* a link 하이퍼링크 밑줄 없애기 */
  text-decoration: none;
  &:focus {
    @extend %tab-focus;
  }
```

## 기타
* _config.yml 파일중 다음을 수정하자 (포스트 읽는데 걸리는 시간, 공유 기등 등)
```
# Defaults
defaults:
  # _posts
  scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true # 해당 포스트를 읽는데 걸리는 시간을 표시할지. 개인적으로 불필요 정보라 생각.
      comments: # true
      share: true # 포스트 공유 기능을 활성화 할것인지. 개인적으로 별로..
      related: true
```


## 참고
* 참고하자 
  * https://mmistakes.github.io/minimal-mistakes/docs/posts/
  * https://mmistakes.github.io/minimal-mistakes/docs/pages/
* 참고중
  * https://ansohxxn.github.io/categories/#blog
  * https://devinlife.com/howto%20github%20pages/category-tag/
