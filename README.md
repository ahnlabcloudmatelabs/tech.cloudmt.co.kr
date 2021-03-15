![Build Website](https://github.com/mate365/mate365.github.io/workflows/Build%20Website/badge.svg)
# 클라우드메이트 기술 블로그

[Hugo](https://gohugo.io)및 [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 테마 기반으로 만든 기술 블로그 입니다.

## 글 작성 준비

Hugo, Git이 먼저 설치되어 있어야 합니다. [이 문서](https://gohugo.io/getting-started/installing/)를 참고하여 Hugo 를 먼저 설치합니다. 
Mac, Linux 의 경우 Homebrew, Windows 의 경우 Chocolately, Scoop 활용하여 설치하는 것을 권장합니다. Git 또한 동일한 도구로 설치 가능합니다.

저장소를 복제한 후, 의존성을 받습니다.
```bash
git clone https://github.com/mate365/mate365.github.io.git
cd mate365.github.io
hugo mod get
```

## 저자 등록
저자 등록을 하지 않았다면 먼저 합니다. 본인 이름의 로마자 표기에 공백 대신`-`를 사용한 표기법으로 폴더를 만들고 그 안에, `_index.md` 를 생성합니다.
```bash
# 이름이 "홍길동" 이고, 로마자 표기가 Gildong Hong 인 경우, 아래처럼 gildong-hong 으로 사용.
hugo new --kind author authors/gildong-hong/_index.md
```
생성된 `_index.md` 파일을 열어. 해당 프로필 파일의 안내에 따라 프로필 이미지를 넣고 프로필 파일을 수정합니다.
```yaml
---
name: Author # 화면에 표시될 저자 이름
photo: Author.jpg # 저자 프로필 이미지 경로. index.md 와 같은 폴더에 이미지를 넣고 사용합니다.
---
저자에 대한 소개 여기에 입력. 없는 경우 지우고 비워두면 됩니다.
```

## 글 작성하기
아래 명령을 실행하여, 게시물용 디렉터리를 생성하고 그 아래에 글 본문이 담길 `index.md`파일을 생성합니다.
그리고 이 `index.md` 파일을 열어 글을 작성합니다. 사진을 첨부하는 경우 `index.md` 와 같은 폴더에 이미지 파일을 넣고 본문에 삽입합니다.
```bash
# hugo new --kind post post/원하는-게시물-주소/index.md
hugo new --kind post post/path-to-post/index.md
```

아래와 같은 파일이 생성됩니다. `authors`의 경우, 앞서 생성한 저자 프로필 폴더 이름을 사용합니다. 저자가 여러명인 경우 리스트 형태로 여러명 넣어줍니다.
카테고리의 경우 글 분류 용도이므로, 나열된 4개 카테고리중 **하나만** 선택하여 작성합니다.
```yaml
---
title: 글 제목 입력
authors:
- gildong-hong # 저자 프로필 페이지 경로 입력
date: 2020-11-26T15:55:40+09:00
categories:
# 아래 4가지 카테고리 중 하나만 선택하여 남겨두고, 나머지는 지우고 글을 계속 작성하세요.
# - Hands On # 실습 위주의 글
# - Tech # 새로운 기술 소개 또는 Hands On 이외 기술 관련 글
- Column # 엔지니어가 아니여도 쉽게 읽을 수 있는 글
# - Mate Story # 클라우드메이트 소식, 일하는 방식 소개 등
tags:
- 태그1
- 태그2
ShowToc: false # 글 개요 보여줄지 여부
TocOpen: false # 글 개요를 보여주는 경우, 펼처서 보여줄지 여부.
draft: false # 초안 작성 모드. true 설정시 커밋해도 나오지 않습니다.
---
...
```

## 작성 글 미리보기

작성한 글을 로컬에서 미리 보려면, 아래 명령을 이용합니다.
```bash
hugo serve
# 수정 하면서 미리보기 하려면 -w 옵션 사용
# hugo serve -w
```

## 글 게시하기

게시물 파일의 Front Matter 의 `draft`가 `false` 이거나, 지워져 있는지 확인 후 `main` 브랜치에 커밋하여 푸시합니다.
푸시되면 GitHub Actions 에 의해 자동을 빌드 및 배포가 진행됩니다.

저장소에 커밋을 직접 푸시하려면, 커밋 권한을 먼저 받아야 합니다. 
클라우드팀에 문의하여 본인 GitHub 계정 정보 전달 후, 저장소 커밋 권한을 먼저 받으시기 바랍니다.

사내 기술 블로그는 양식이 따로 없습니다. 자유롭게 작성하셔서 올려주세요.
