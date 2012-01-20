---
author: 홍형석
layout: post
title: "jquery note plugin"
date: 2012-01-20 19:23
comments: true
categories: jquery
---

![jquery note preview](https://lh5.googleusercontent.com/-0TusrT1pxqA/Txk9pS0yFcI/AAAAAAAAABQ/AuhNXWl4ZrA/jquey.note.png)

[demo](http://aanoaa.github.com/jquery-note/)

# jquery.note #

## what the.. ##

메모장입니다.
상태를 저장할 수 있고요.
상태에 따른 hook 이 많아서 유연하게 사용할 수 있고요.
ajax 로 데이터를 주고 받아서 서버에 기록을 남길 수도 있습니다.
회사에서 필요해서 맨들었습니다.
여기저기 찾아봤는데 꼭 바라는게 없더라고요.

## 사용법 ##

html 안에서 css, 랑 js 를 include 해주고요
`$(document).ready` 안에 슥슥해주면 됩니다.

{% codeblock [jquery.note] [lang:html] %}
<link rel="stylesheet" href="css/jquery.note.css" type="text/css" media="screen" />
<script type="text/javascript" src="script/jquery-1.6.2.js"></script>
<script type="text/javascript" src="script/jquery.note.js"></script>
<script type="text/javascript">
  $(document).ready(function() {
    $("a[title=note]").each(function() {
      $(this).note();
    });
  });
</script>
{% endcodeblock %}

[coffeescript](http://coffeescript.org/) 로 작성했고요, css 는
[scss](http://sass-lang.com/) 로 작성했습니다.

## Hooks ##

{% codeblock [jquery.note hook example] [lang:javascript] %}
$(document).bind('afterReveal.note', function(e, data) {
  if (data.opts.notes.length === 0) {
    return $.ajax({
      type: 'GET',
      dataType: 'json',
      url: '/path/to',
      success: function(res, textStatus, jqXHR) {
        data.opts.notes = res;
        // res 는 아래처럼 구성됩니다.
        // [{"who":"hshong","date":"2012-01-20 19:23","status":"open"},{"who":"aanoaa","date":"2012-01-20 19:25","note":"다시하렴 처음부터"}]
        if (data.opts.notes.length !== 0) return $(data.owner).click();
      }
    });
  }
});
{% endcodeblock %}

notes가 보여질때 갯수를 확인한다음에, 0개라면 서버로 질의해서 노트와,
상태정보를 받아와서 업데이트 해주고 reloading 합니다.

옵션에 따라 달리 동작할 수 있는데, 실제로 쓸거 아니면 알 필요
없습니다.

사용가능한 hook 은 아래와 같습니다.

- `init.note` 초기화할때
- `afterClose.note` 닫히자마자
- `beforeSend.note` ajax 보내기전에
- `afterSuccess.note` ajax 성공하자마자
- `beforeReveal.note` 보여지기바로전에
- `afterReveal.note` 보여지자마자
- `changeStatus.note` open에서 close 되는것처럼 상태가 변경되고 나면

# See also #

- [jquery-note on github](https://github.com/aanoaa/jquery-note)

# 마무리 #

jquery plugin 을 만들고자 하는 분이라면 한번 스윽 보셔도 괜찮을 것
같습니다.

javascript 잘하시는 어떤분이 제 코드를 보고 좀 까줬으면 하는 바람도
있습니다. 제가 영어가 딸려서 미국사람한테는 그런부탁 못하거든요.

정말 글 못쓰네요.. 어렵고요 아아아아.. 쓰다보면 나아지겟죠.
