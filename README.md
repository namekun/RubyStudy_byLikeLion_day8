# 20180618 Rails with LikeLion day8



※ rails g model board contents ip_address는 String 속성까지 만들어냄

※ rails g model board contents:text ip_address index:integer는 타입까지 지정

※ rails g controller tweet index show new edit #컨트롤러 생성시 index, show,new, edit 함수를 같이 생성함

* 모든 코드는 day7의 twitter코드를 고치는 과정입니다.

### form-helper, view helper

* `<html>`을 사용하는데 있어 좀더 쉽게 작성할 수 있는 방법은 `Ruby`의 `Form Helper`을 사용하는 것입니다.  `Form Helper`는 다음과 같이 사용할 수 있습니다.

- http://guides.rubyonrails.org/form_helpers.html 참고
- form이나 view를 함수처럼 사용해서 사용하기 쉽게 만들어낸 것입니다.
- form_tag는 요청도 자바스크립트로, 응답도 자바스크립트로 받는 방법임
- a태그는 link_to로 바꿔줄 수 있음. 즉,  <%= link_to board.contents, "/tweet/<%= board.id %>">로 변경가능

*new.html.erb*

```erb
<!--<body>-->
   <!-- <form class="form-signin container" action='twitter/create' method="POST">-->
<!--form_tag를 통해 한줄로 훨씬 편하게 쓸 수 있다.-->
<%= form_tag('/twitter/create', class:"container") do %> 
      <div class="form-label-group">
      <!--  <input type="text" name="username" class="form-control" placeholder="닉네임을 입력하세요!" required="" autofocus="" maxlength="10"> -->          
           <%= text_area_tag :username, nil, class: "form-control",  placeholder:"닉네임을 입력해주세요!", maxlength:10 %>
        <br/>
      </div>
  

      <!--<div class="form-label-group">-->
      <!--  <input type="contents" name="board" class="form-control" placeholder="내용을 입력해주세요!" required="" maxlength="50">-->
      <!--  <br/>-->
      <!--</div>-->
    <%= text_area_tag :board, nil, class: "form-control",  placeholder:"내용을 입력해주세요!", maxlength:50 %>
    <br/>
   
    

    <!--<div>-->
    <!--  <button class="btn btn-lg btn-primary btn-block" type="submit">트윗하기!</button>-->
     <%= submit_tag "tweet!", class: "btn btn-lg btn-primary btn-block" %>
    <!--  <p class="mt-5 mb-3 text-muted text-center">© 2018</p>-->
    <!--</div>-->
    <!--</form>-->
  <%end%>

<!--</body>-->
```



*edit.html.erb*

```erb
<body>
    <!--<form class="form-signin container" action= "/twitter/<%=@tweet.id%>/update" method ="POST">-->
    <%= form_tag("/twitter/#{@tweet.id}/update", class:"form-signin container") do %>    
        
    <!--<input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">-->
      <div class="text-center mb-4">
        <img class="mb-4" src="https://getbootstrap.com/assets/brand/bootstrap-solid.svg" alt="" width="72" height="72">
        <h1 class="h3 mb-3 font-weight-normal">수정할 내용이 있나요?</h1>
        <p>그렇다면 얼른 바꿔주세요!</p>
      </div>

      <!--<div class="form-label-group">-->
      <!-- <br/>-->
      <!--</div>-->
       <%= text_area_tag :board, @tweet.board, class: "form-control", maxlength:50 %>
    <br/>
      

      <!--<button class="btn btn-lg btn-primary btn-block" type="submit">수정하기!</button>-->
    <%= submit_tag "수정!", class: "btn btn-lg btn-primary btn-block" %>
      <p class="mt-5 mb-3 text-muted text-center">©2018</p>
 
    
    <!--</form>-->
  <% end %>

</body>
```

### flash



- flash는 한 번 보여지고 더 이상 보여주지않는 것을 말함. 
- flash를 사용하면, 어떤 이벤트가 발생했을 때, 일정시간동안 alert를 발생시킴

*twitter_controller.rb*

```ruby
  class TwitterController < ApplicationController

def index 
    @tweets = Twitter.all
    p session
    cookies[:username] = "김남혁"
end

def new
    
end 

def create
    tweet = Twitter.new
    tweet.board = params[:board]
    tweet.username = params[:username]
    tweet.ip_address = request.ip
    tweet.save
    flash[:success] = "트윗이 잘 날라갔습니다!"
    redirect_to '/twitter'
end

def edit
    @tweet = Twitter.find(params[:id])
end

def update
    tweet = Twitter.find(params[:id])
    tweet.board = params[:board]
    tweet.save
    flash[:success] = "트윗에 업데이트 되었습니다!"
    redirect_to "/twitter"
end

def destroy
    tweet=Twitter.find(params[:id])
    tweet.destroy
    flash[:success] = "트윗이 없어졌습니다!"
    redirect_to '/twitter'
end


def show
    @tweet = Twitter.find(params[:id])    
end


end
```


*application.html.erb의 flash부분*

```erb
 ....
    <div clas="container">
      <% flash.each do |k, v| %>
        <script>
          toastr.<%= k %>("<%= v %>");
        </script>
      <% end %>
    </div>
    <%= yield %>
  </body>
</html>
```

*gemfile.rb*

```ruby
source 'https://rubygems.org'

git_source(:github) do |repo_name|
  repo_name = "#{repo_name}/#{repo_name}" unless repo_name.include?("/")
  "https://github.com/#{repo_name}.git"
end

....

gem 'bootstrap', '~> 4.1.1'

gem 'toastr_rails'


```

*twitter_app > app > assets > javascripts > application.js*

```ruby
...
//= require popper
//= require bootstrap
//= require toastr_rails
...

toastr.options = {
  "closeButton": true,
  "debug": false,
  "progressBar": true,
  "positionClass": "toast-top-right",
  "showDuration": "300",
  "hideDuration": "1000",
  "timeOut": "5000",
  "extendedTimeOut": "1000",
  "showEasing": "swing",
  "hideEasing": "linear",
  "showMethod": "fadeIn",
  "hideMethod": "fadeOut"
};
```

*twitter_app > app > assets > stylesheets > application.scss*

```ruby
@import 'bootstrap';
@import 'toastr_rails';

#toast-container{
  top: 70px;
}
```

### Cookie와 Session

- _twitter_app_session이 로그인한 유저의 정보를 갖고 있음. 즉, 브라우저가 들고있다는 말임. 서버가 가지고있는게 아님. 즉, client와 server가 정보를 주고 받을 때, request와 response가 독립적으로 움직이는데, 쿠키는 브라우저에 정보를 주고받은 것을 저장하고, 세션은 해쉬로 암호와하여 비밀리에 주고받는다.
- flash도 하나의 쿠키임.. 휘발성
- https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies
- 쿠키는 상태가 없는([stateless](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview#HTTP_is_stateless_but_not_sessionless)) HTTP 프로토콜에서 상태 기반 정보를 기억합니다.


# 강사님 ver

### Model

- 기존에 Model을 만들 때 모델명만 넣었다면 이번에는 뒤에 여러개의 파라미터를 붙여 컬럼이 함께 선언될 수 있도록 해보자.

```command
$ rails g model board contents:string ip_address:string
$ rake db:migrate
```

- : 뒤에 데이터 타입을 붙여주면 된다

*db/migrate/create_boards.rb*

```ruby
class CreateBoards < ActiveRecord::Migration[5.0]
  def change
    create_table :boards do |t|
      t.string :contents
      t.string :ip_address

      t.timestamps
    end
  end
end

```



### Routes

- RESTful 한 규칙에 맞춰 라우팅을 추가한다.

*config/routes.rb*

```ruby
Rails.application.routes.draw do
  root 'tweet#index'
  
  get '/tweets' => 'tweet#index'
  get '/tweet/new' => 'tweet#new'
  get '/tweet/:id' => 'tweet#show'
  post '/tweet/create' => 'tweet#create'
  get '/tweet/:id/edit' => 'tweet#edit'
  post '/tweet/:id/update' =>'tweet#update'
  get '/tweet/:id/destroy' => 'tweet#destroy'

end
```

- 추후에 `DELETE`나 `PUT`, `PATCH` 메소드로 수정할 예정이다.
- 주의해야할 점은 `get '/tweet/new'` 가 `get '/tweet/:id'` 보다 먼저 선언되야 한다는 점이다.



### Controller

- 모델과 마찬가지로 Controller를 만들 때 파라미터를 추가하면 액션과 뷰 파일을 함께 만들 수 있다.

```command
$ rails g controller tweet index show new edit
```

- 뷰 파일이 필요한 부분만 골라 파라미터를 추가했다.

*app/controllers/tweet_controller.rb*

```ruby
class TweetController < ApplicationController
  def index
    @boards = Board.all
  end

  def show
    @board = Board.find(params[:id])
  end

  def new
  end
  
  def create 
    board = Board.new
    board.contents = params[:contents]
    board.ip_address = request.ip
    board.save
    redirect_to "/tweet/#{board.id}"
  end

  def edit
    @board = Board.find(params[:id])
  end
  
  def update
    board = Board.find(params[:id])
    board.contents = params[:contents]
    board.ip_address = request.ip
    board.save
    redirect_to "/tweet/#{board.id}"
  end
  
  def destroy
    board = Board.find(params[:id])
    board.destroy
    redirect_to '/tweets'
  end
end
```

- 기본적인 CRUD 코드는 안보고도 쓸 수 있을 정도로 반복해서 숙달할 수 있도록 한다.



### View

- 컨트롤러를 생성하면서 뷰파일을 함께 생성했기 때문에 따로 파일을 만들어 줄 필요는 없다.

*app/views/tweet/index.html.erb*

```erb
<div class="text-center">
    <a class="btn btn-primary" href="/tweet/new">새글 등록하기</a>
</div>
<div class="list-group">
<% @boards.each do |board| %>
    <a href="/tweet/<%= board.id %>" class="list-group-item list-group-item-action"><%= board.contents %></a>
<% end %>
</div>
```

*app/views/tweet/show.html.erb*

```erb
<h2><%= @board.contents %></h2>
<p><%= @board.ip_address %></p>
<a href="/tweets" class="btn btn-primary">목록으로</a>
<a href="/tweet/<%= @board.id %>/edit" class="btn btn-warning text-white">수정</a>
<a href="/tweet/<%= @board.id %>/destroy" class="btn btn-danger">삭제</a>
```

*app/views/tweet/new.html.erb*

```erb
<form action="/tweet/create" method="post">
    <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token">
    <textarea name="contents" class="form-control"></textarea>
    <input type="submit" class="btn btn-primary" value="작성하기">
</form>
```

*app/views/tweet/edit.html.erb*

```erb
<form action="/tweet/<%= @board.id %>/update" method="post">
    <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token">
    <textarea name="contents" class="form-control"><%= @board.contents %></textarea>
    <input type="submit" class="btn btn-primary" value="수정하기">
</form>
```

- 정상적으로 동작하는지 잘 확인하자.



### View Helper

#### form_tag

- 기존의 html `form` 태그를 사용하면 항상 `authenticity_token`을 함께 넘겨줘야 정상적으로 동작한다. Rails의 보안 때문인데 이 토큰이 없으면 폼이 정상적으로 동작하지 않는다. 또한 `form` 태그에 action과 method 속성을 함께 해야한다.(form 의 default method는 get이다.) 이러한 복잡성을 해결하기 위해 view helper 중에서 form helper를 사용한다. 또한 form의 여러 input 태그들을 루비코드로 전환하여 사용할 수 있다. 원래의 코드를 다음과 같이 고칠 수 있다.

*app/views/tweet/new.html.erb*

```erb
<%= form_tag('/tweet/create') do %>
    <%= text_area_tag :contents, nil, class: "form-control" %>
    <%= submit_tag "작성하기", class: "btn btn-primary" %>
<% end %>
```

- 위 코드로 만들어지는 뷰는 다음과 같다.

```html
<form action="/tweet/create" accept-charset="UTF-8" method="post">
    <input name="utf8" type="hidden" value="✓">
    <input type="hidden" name="authenticity_token" value="G1DrVLJyphi00sromq8W8EEalFT84wGakgqddBC0hQ4WLLOH2+QR9IjxzdtPMlvphg4Od72DosqBCGLcLgUO7w==">
    <textarea name="contents" id="contents" class="form-control"></textarea>
    <input type="submit" name="commit" value="작성하기" class="btn btn-primary" data-disable-with="작성하기">
</form>
```

- `authenticity_token` 등 복잡한 코드를 자동으로 생성해준다.



#### link_to

- `a` 태그에 대해서도 view helper가 존재하는데 `link_to "text", path`의 형태로 사용할 수 있다.

*app/views/tweet/index.html.erb*

```erb
<div class="text-center">
    <a class="btn btn-primary" href="/tweet/new">새글 등록하기</a>
</div>
<div class="list-group">
<% @boards.each do |board| %>
    <%= link_to truncate(board.contents, length: 10), "/tweet/#{board.id}", class: "list-group-item list-group-item-action" %>
<% end %>
</div>
```

- 추가적인 속성을 사용하기 위해서는 파라미터로 계속해서 붙여나가면 된다.
- 형태를 잘 모를 경우에는 google 검색 중 **API-DOC**을 통해 형태를 확인할 수 있도록 한다.
- 대표적으로 사용하는 **view helper**는 위 두가지이다. 추가적으로 사용할 경우에는 설명을 추가하도록 하겠다.



### cookies, session, flash

- 모든 http 요청은 각각 독립적이다. 이전의 요청을 기억하거나 현재 사용자가 누구인지 기억하지 못한다는 의미이다. 이러한 한계점을 해결하기 위해서 `cookie`를 활용한다.

> MDN에서 제공하는 쿠키의 정의
>
> HTTP 쿠키(웹 쿠키, 브라우저 쿠키)는 서버가 사용자의 웹 브라우저에 전송하는 작은 데이터 조각입니다. 브라우저는 그 데이터 조각들을 저장할 수 있고 동일한 서버로 다음 요청 시 함께 전송할 것입니다. 일반적으로, 동일한 브라우저에서 두 요청이 들어오는 것을 말합니다 — 예를 들자면, 유저 로그인 상태 유지. 쿠키는 상태가 없는([stateless](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview#HTTP_is_stateless_but_not_sessionless)) HTTP 프로토콜에서 상태 기반 정보를 기억합니다. 

![1](./day8_1.jpg)

- 해당 내용은 개발자 도구 *application* 탭에서 확인할 수 있다.
- 쿠키에 정보를 등록하려면 controller에 다음 코드를 추가하면 된다.

*app/controllers/tweet_controller.rb*

```ruby
...
    def index
        @boards = Board.all
        cookies[:hello] = "world"
    end
...
```

![2](./day8_2.jpg)

- 위 그림과 같이 우리가 등록한 쿠키를 확인할 수 있다. 하지만 위와 같은 방법으로 등록할 경우 우리가 설정한 쿠키의 key와 value를 모든 사용자가 확인할 수 있다. 이러한 경우에 사용자 위조와 같은 불상사가 발생할 수 있다.
- 이러한 경우를 해결하기 위해 *session*을 사용하게 된다. *session*은 이러한 정보를 서버에 저장하고 session-id를 브라우저에 저장하여 사용자 요청마다 해당 session-id를 받아 정보를 유지시키는 방식이다.

![3](./day8_3.jpg)

>1. session이란? 서버가 해당 서버(웹)로 접근(request)한 클라이언트(사용자)를 식별하는 방법
>
>2. 서버(웹)는 접근한 클라이언트(사용자)에게 response-header field인 **set-cookie** 값으로 클라이언트 식별자인 **session-id(임의의 긴 문자열)를 발행**(응답)한다.
>
>3. 서버로부터 발행(응답)된 **session-id는 해당 서버(웹)와** **클라이언트(브라우저)** **메모리에 저장**된다. 
>
>이때 클라이언트 메모리에 사용되는 cookie 타입은 세션 종료 시 같이 소멸되는 **"Memory cookie"**가 사용된다.
>
>4. 서버로부터 발행된 session(데이터)을 통해 개인화(사용자)를 위한 데이터(userInfo 등..)로 활용할 수 있다.

- session 에 대해 위와 같이 설명하고 있다.

```ruby
...
    def index
        @boards = Board.all
        session[:hello] = "world"
    end
...
```

- 레일즈에서 세션은 쿠키와 동일한 방식으로 설정할 수 있다.

#### flash

- 지속적으로 유지되는 쿠키나 세션과는 달리 다음 요청에서 한번만 동작하는 임시 쿠키와 같은 역할 하는 것이 있다. 바로 `flash`인데, `flash`는 다음 페이지에 한번 메시지를 보여주고 리디렉션 시 바로 사라진다.
- 글이 등록되거나 삭제, 수정됐을 때 완료 메시지를 띄울 때 주로 사용한다.

```ruby
...
  def create 
    board = Board.new
    board.contents = params[:contents]
    board.ip_address = request.ip
    board.save
    flash[:success] = "새 글이 등록되었습니다."
    redirect_to "/tweet/#{board.id}"
  end
...
```

- 위와같은 방식으로 사용할 수 있다. 실제 뷰에서 보기 위해서는 다음과 같은 코드를 레이아웃에 등록한다.

*app/views/layouts/application.html.erb*

```erb
...
  <body>
    <div class="d-flex flex-column flex-md-row align-items-center p-3 px-md-4 mb-3 bg-white border-bottom box-shadow">
      <h5 class="my-0 mr-md-auto font-weight-normal">My Twitter</h5>
      <nav class="my-2 my-md-0 mr-md-3">
        <a class="p-2 text-dark" href="/">Home</a>
        <a class="p-2 text-dark" href="/tweet/new">New</a>
      </nav>
    </div>
    <div class="container">
    <% flash.each do |k,v| %>
        <div class="alert alert-<%= k %>"><%= v %></div>
    <% end %>
    <%= yield %>
    </div>
  </body>
</html>
```

- `alert` 클래스는 bootstrap에 있는 class를 활용하여 더 보기좋은 뷰를 완성해준다.
