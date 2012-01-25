---
author: 홍형석
layout: post
title: "Octopress usage"
date: 2012-01-20 01:44
comments: true
categories: octopress
---

# octopress 에서 어떻게 글을 쓸 것인가? #

## ruby install ##

    $ rvm install 1.9.2 && rvm --default use 1.9.2
    $ gem install bundler

rvm 은 ruby 의 [perlbrew](http://perlbrew.pl/) 고요,
bundler는
[Carton](http://search.cpan.org/~miyagawa/carton-v0.9.3/lib/Carton.pod)
입니다.

## git clone && bundle install ##

저장소의 remote 정보는 아래와 같습니다.

    $ git remote -v
    octopress	git://github.com/imathis/octopress.git (fetch)
    octopress	git://github.com/imathis/octopress.git (push)
    origin	git@github.com:silexkr/silexkr.github.com.git (fetch)
    origin	git@github.com:silexkr/silexkr.github.com.git (push)

살렉스 여러분은 write 권한이 있습니다. clone 받으세여.

    $ git clone git@github.com:silexkr/silexkr.github.com.git

그리고 의존모듈을 설치 합니다.

    $ cd silexkr.github.com.git/
    $ bundle install

글을 쓰기 위해선 `source` 브랜치를 사용합니다.

    $ git checkout source

## rake new_post ##

새글을 쓰기 위해선 직접 파일을 맨들어줘도 되지만 rake 유틸리티를
이용하세요. 편합니다. 요렇게요.

    $ rake new_post["Octopress usage"]
    mkdir -p source/_posts
    Creating new post: source/_posts/2012-01-20-octopress-usage.markdown

`Octopress usage` 가 제목이고요,
`source/_posts/2012-01-20-octopress-usage.markdown` 파일이
만들어졌습니다.

## 메타정보입력 ##

`source/_posts/2012-01-20-octopress-usage.markdown` 파일을 열어보면 젤
위에 요렇게 나옵니다.

    ---
    layout: post
    title: "Octopress usage"
    date: 2012-01-20 01:44
    comments: true
    categories:
    ---

저희는 여러명이서 쓸것이기 때문에 `author` 정보도 입력하면 좋습니다.
default 는 **Silex** 입니다.

카테고리는 여러개를 입력할 수
있습니다. [Blogging Basics](http://octopress.org/docs/blogging/) 에
좋은 예가 있습니다.

```
# One category
categories: Sass

# Multiple categories example 1
categories: [CSS3, Sass, Media Queries]

# Multiple categories example 2
categories:
- CSS3
- Sass
- Media Queries
```

저는 이글을 쓰면서 요렇게 했습니다.

    ---
    author: 홍형석
    layout: post
    title: "Octopress usage"
    date: 2012-01-20 01:44
    comments: true
    categories: octopress
    ---

## 글작성 ##

markdown 문법을 따릅니다.
[Plugins](http://octopress.org/docs/blogging/plugins/)를 활용해서
이미지도 옇고 코드도 옇습니다.

## local 에서 확인 ##

    $ rake preview
    Starting to watch source with Jekyll and Compass. Starting Rack on port 4000
    ~~~ 중략 ~~~
    >>> Compass is polling for changes. Press Ctrl-C to Stop.

문서보면 `rake watch` 도 있는데여, `scss`, `css` 건드릴거 아니면
안띄워줘도 됩니다.

## rake deploy ##

로컬에서 확인이 완료 되었으면, deploy 합니다.

    $ rake deploy

github 에서 성공메세지를 받으면 [silexkr.github.com](http://silexkr.github.com/) 에서 바로
확인할 수 있습니다.(대부분 바로 반영되더라고요)

## git add, commit, push ##

deploy 와는 별도로 원본글을 커밋하고 이력을 관리합니다.

    $ git add .
    $ git commit -m "posting Octopress usage"
    $ git push origin source

*주의*할게 있는데요. `master` 브랜치가 아니라 `source` 로 push 해야
 합니다.

`master`는 원본이 변환된 static 파일들이 모여 서비스 되는 브랜치고요,
`source`에서 원본을 관리합니다.

생각해보니까 `master`는 어차피 deploy 할때 HEAD 라서 그냥 push 해도
되겠군여.

# 이제 뭘 하징? #

네 글을 써주세요.
 
# See also #
- [Blogging Basics](http://octopress.org/docs/blogging/)
