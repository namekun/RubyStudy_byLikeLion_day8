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
