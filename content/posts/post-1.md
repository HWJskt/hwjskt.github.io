+++
title="테마 2가지 사용하기"
date=2023-04-20

[extra]
toc = true

[taxonomies]
categories = ["making Blog"]
tags = ["zola", "themes"]
+++

zola 의 Deep-Thought theme 을 이용해서 만들던 중, Book theme 을 함께 쓰려고 조금씩 수정을 했습니다.  
그 과정에서 겪은 시행착오를 메모합니다.  

<!-- more -->

## 원하는 글꼴로 통일
Typora 의 newsprint css 를 다운받아 /sass 폴더에 복사합니다.  
확장자를 css 에서 scss로 바꿔줍니다.  
뒤에 `{% block user_custom_stylesheet %}` 블럭을 이용해서 적용해줍니다.

<br/>

## css 를 복사해오기
zola 는 sass 파일로 작성한 다음, build 할 때 css 파일로 만듭니다.  
Book theme 를 사용하려면 테마의 sass 파일들을 /sass 폴더로 복사해서 가져와야 합니다.  
파일이름이 구별되도록 book.sass 파일을 제외한 나머지 파일에 _book 이라는 prefix를 붙여줍니다.  
(앞에 _ 를 붙이지 않으면 book.sass에서 import 가 되지 않습니다.)  
book.sass 파일안에 import 도 파일이름에 맞게 고쳐줍니다.  

<br/>

## 각 메뉴에 맞게 css 적용하기
기존 Deep-Thought 테마를 사용하는 곳에는 글꼴 선택 정도의 테마를 적용합니다.
/templates/base.html 에 아래 스타일을 추가합니다.  
```jinja2
{% block user_custom_stylesheet %}
    {# 전체 글꼴 적용 #}
    <link rel="stylesheet" href="{{ get_url(path='newsprint.css') | safe }}">
{% endblock %}
```
  
Book 테마를 사용하는 곳에는 book에서 가져온 css 만 적용하는데, 나머지는 원래 css를 적용해서 통일감을 유지합니다.
/templates/book_index.html 에 아래 스타일을 추가합니다.  
```jinja2
{% block user_custom_stylesheet %}
    {# book 의 css를 불러온 후, deep-thought의 css를 덮어썼다. #}
    <link rel="stylesheet" href="{{ get_url(path='book.css') | safe }}">
    <link rel="stylesheet" href="{{ get_url(path='deep-thought.css') | safe }}">
    {# 전체 글꼴 적용 #}
    <link rel="stylesheet" href="{{ get_url(path='newsprint.css') | safe }}">
{% endblock %}
```
  
약간 차이가 남는데 어쩔수 없습니다. css를 다 뜯어고치기엔 너무 귀찮다.......

<br/>

## 전용 template 를 만들기
book theme 의 /templates 폴더에서 파일을 복사해옵니다.  
prefix 를 붙여서 book_index.html, book_page.html, book_sectinn.html 파일로 이름을 변경합니다.
아래와 같이 조금씩 수정해줍니다.

### - book_index.html
base.html을 복사해서 book_index.html 을 만듭니다.  
`{% extends "DeepThought/templates/base.html" %}` 를 `{% extends "base.html" %}` 로 수정합니다.

css 를 불러오기 위해 아래 코드를 추가합니다.  
```jinja2
{% block user_custom_stylesheet %}
    {# book 의 css를 불러온 후, deep-thought의 css를 덮어썼다. #}
    <link rel="stylesheet" href="{{ get_url(path='book.css') | safe }}">
    <link rel="stylesheet" href="{{ get_url(path='deep-thought.css') }}">
    {# 본문형식을 맞춰본다 #}
    <link rel="stylesheet" href="{{ get_url(path='newsprint.css') | safe }}">
{% endblock %}
```
  
{% block content %} 에 book theme 의 /templates 폴더애 있는 index.html 의 <body> 를 복사해붙여넣습니다.  

navigation 이 nav bar 와 겹치지 않게 위치를 내려줍니다.
```jinja2
            {% block before_menu %}
            {# lt.side menu 가 상단 navbar에 가리지 않게 내린다  #}
            <div><br><br><br></div>
            {% endblock before_menu %}
            <nav role="navigation">
```
  
block content 가 겹치면 오류가 나기 때문에 이름을 book_content 로 바꿔어줍니다.
```jinja2
               <div class="book-content">
                    {% block book_content %}
                    {% endblock book_content %}
                </div>
```
  

### - book_section.html, book_page.html 수정
`{% extends "book_index.html" %}` 로 수정하고, 아래 내용도 적당히 수정해줍니다.  
pre, post 화살표가 적절히 작동하려면 `{% set index = get_section(path="_index.md") %}` 에서 `_index.md` 대신 `rust/_index.md` 식으로 바꿔주어야 합니다.  


