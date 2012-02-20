---
author: Hyungsuk Hong
layout: post
title: "backbone.js"
date: 2012-02-17 19:39
comments: true
categories: [backbone.js, require.js]
---

## 배경 ##

아 [backbone.js][backbone.js] 가 만들어진 배경이 아니라, 제가 사용하게 된
배경입니다. 회사에서 진행하는 `javascript` 를 나름 무겁게 사용하는
프로젝트가 있습니다. "갑"님의 요구사항이 날이 갈수록 복잡해 짐에 따라 
`jquery` 플러그인도 맨들고, `require.js` 를 사용해서 파일을 조직화해서
사용하고 있었습니다만 `$.ajax` 에 대한 callback 이 늘어남에 따라 점점
코드의 복잡도가 증가해서 해결해야 될 문제라고 생각했습니다.

[Trello][Trello] 에서
[The Trello Tech Stack](http://blog.fogcreek.com/the-trello-tech-stack/)
[backbone.js][backbone.js]를 쓴다는 글을 보고 사용방법을 좀
알아보았습니다.

![Trello Early Architecture Drawing](http://blog.fogcreek.com/wp-content/uploads/2012/01/trello-freehand.jpg)

## 자세한 설명은 (몰라서)생략 ##

왜 쓰는지 뭔가 좋은지 그런거 모릅니다. 여기서는 어떻게 사용하는지만
다룹니다. 검색 ㄱㄱ

![자세한 설명은 생략한다](https://lh5.googleusercontent.com/-i82V0GanZr4/T0IfDIDWq6I/AAAAAAAAABY/VnAii_7ds5E/no_more_details.jpg)

## 맨든사람 ##

[Jeremy Ashkenas - Twitter](https://twitter.com/#!/jashkenas),
[Jeremy Ashkenas - Github](https://github.com/jashkenas/)

요즘 가장 인기있는 코딩왕이 아닐까 합니다.

- [CoffeeScript][CoffeeScript]
- [underscore.js](http://documentcloud.github.com/underscore/)
- [backbone.js][backbone.js]

위에 세개다 저 사람이 맨들었답니다.

## 돈워리 ##

http://documentcloud.github.com/backbone/#examples

- FourSuare
- LinkedIn
- 37 Signals
- Trello
- Silex(끼워 넣어 봅니다)
- ...

등에서 이미 쓰이고 있다고..

## 연습삼아 맨든 프로젝트 ##

일단 [CoffeeScript][CoffeeScript]만세라서
[CoffeeScript][CoffeeScript]로 구현하였습니다.

[jquery-note plugin](http://aanoaa.github.com/jquery-note/)을 다시
한번 만들어 봤습니다.

### Server-side ###

![server-side](https://lh5.googleusercontent.com/-_aPZkJJp5LI/T0I_SRERM-I/AAAAAAAAABo/k_WLzmWNs08/s1024/res-res.example.png)

RESTful 하게 구현되어야 합니다. [backbone.js][backbone.js]의 `model`과
`collection`은 url 규약을 가지고 있습니다. 

```http://documentcloud.github.com/backbone/#Sync
The default sync handler maps CRUD to REST like so:

create → POST   /collection
read → GET   /collection[/id]
update → PUT   /collection/id
delete → DELETE   /collection/id
```

좀 간단하게 말하면 위에선 언급한 구조의 HTTP method 와 url 규약을
지키고, 요청 헤더의 `Accept`에 걸맞는 응답을 줘야 합니다.

서버와 sync 를 하기 위해 `model.save()` 또는 `collection.fetch()`를 호출
하게 되는데 이때, 모든 요청 헤더에 `Accept: application/json` 을
포함합니다. 새로운 item 을 만들기 위해 `POST` 요청을 보낼때도
마찬가집니다. 이때는 추가로 `JSON.stringify(model)` 을 POST body 에
넣고, `Content-type: application/json` 헤더를 포함합니다.

```part of request header
Accept	application/json, text/javascript, */*; q=0.01
Content-Type	application/json; charset=UTF-8
X-Requested-With	XMLHttpRequest
```

```request body
{"qid":"1","status":"reopen","comment":""}
```

server 에서 이런 데이터를 받아서 처리해야 하는(deserialize/serialize)
자잘함을 피하기 위해 저는
[Catalyst::Controller::REST][Catalyst::Controller::REST] 를
사용했습니다. 요청/응답 헤더에서 사용가능한 "Accept"를 확인하고
serialize/deserialize 를 자동으로 해주기 때문에 편리합니다.

### Client-side ###

![preview](https://lh5.googleusercontent.com/--90Nykdhcvc/T0IyuoSwCeI/AAAAAAAAABg/V0Lp1rRzJXE/preview-backbone-note.png)

- 상태 및 코멘트를 `note` 라 하겠습니다.
- 위 그림처럼 한화면에 보여지는 `note`의 집합(또는 컬렉션)을 `query`
라 하겠습니다.

`http://example.com/#/query/:id` url 을 감지하고 원하는 event 를 발생
시켜야 합니다. 그러기 위해선 `Backbone.Router` 를 확장해서 app 전용
router 를 만들어야 합니다.

```coffeescript
class MyModel extends Backbone.Model

class MyCollection extends Backbone.Collection
  model: MyModel

class MyRouter extends Backbone.Router
  routes:
    '/query/:id': 'query'
  query: (id) =>
    # do something
    coll = new MyCollection
    coll.url = "/query/#{id}"
    coll.fetch
      success: (c, res) ->
        new MyView { collection: c }
      error: (c, res) ->
        # error handling
```

`http://example.com/#/query/2` 일때, `GET /query/2` 하고 응답이
정상이면 `MyView` 를 만듭니다. view 가 초기화될때, render 를 호출하면
DOM 이 알맞게 슥슥 바뀌게 됩니다. 일반적으로 `collection.models` 를
순회하면서 html element 를 구성하고 특정 #id 에 append 합니다.

```coffeescript
class MyView extends Backbone.View
  initialize: ->
    do @render
  template:
    '''
      {{foo}} is {{bar}}
    '''
  render: =>
    _.each @collection.models, (item) =>
      # 템플릿을 사용해서 슥슥하면 편리합니다. 예를들어..
      html += Mustache.render(@template, { foo: 'hello', bar: 'world' }) # hello is world
      # 일반적으론 이렇게는 직접 데이터를 구성하지는 않겟고, item 을 사용하겟죠
      @delegateEvents() # event 처리에 대한 꼼순데 생략
      @ # render 의 마지막에 this 를 return 함으로써 jquery chain 을 사용할 수 있도록 합니다.
```

[backbone.js][backbone.js] 을 사용한 부분의 코드입니다. 조금
헷갈리수도 있는게, [Require.js][Require.js]를 사용해서 `define`,
`order!` 같은 구문이 거슬릴 수도 있습니다만, 바다와 같은 마음으로
이해해 주세요. 

{% gist 1868873 %}

## 결론 ##

- 기존꺼 바뀐거..100줄 정도 줄어듬
- js 가 웹서비스의 일부분이 되어버림
- jquery plugin 과 달리 재사용을 쉽게 못함
- 서버 구현에 있어서 sync 를 위해서 RESTful하게 설계되어야함
- rails 후렌들리

## See also ##

- [backbone.js](http://documentcloud.github.com/backbone/)
- [Backbone.js Tutorials](http://backbonetutorials.com/)
- [Hello Backbone.js Tutorial](http://arturadib.com/hello-backbonejs/)
- [An Intro to Backbone.js: Part 1](http://liquidmedia.ca/blog/2011/01/backbone-js-part-1/)
- [An Intro to Backbone.js: Part 2](http://liquidmedia.ca/blog/2011/01/an-intro-to-backbone-js-part-2-controllers-and-views/)
- [An Intro to Backbone.js: Part 3](http://liquidmedia.ca/blog/2011/02/backbone-js-part-3/)
- [Backbone.js Tutorial with Rails Part 1](http://www.jamesyu.org/2011/01/27/cloudedit-a-backbone-js-tutorial-by-example/)
- [Backbone.js Tutorial with Rails Part 2](http://www.jamesyu.org/2011/02/09/backbone.js-tutorial-with-rails-part-2/)
- https://github.com/documentcloud/backbone/wiki/Tutorials%2C-blog-posts-and-example-sites


[backbone.js]: http://documentcloud.github.com/backbone/
[Trello]: https://trello.com/
[CoffeeScript]: http://coffeescript.org/
[Catalyst::Controller::REST]: http://search.cpan.org/~bobtfish/Catalyst-Action-REST-0.96/lib/Catalyst/Controller/REST.pm
[Require.js]: http://requirejs.org/
