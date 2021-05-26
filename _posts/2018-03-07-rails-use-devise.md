---
layout: post
title: "Ruby On Rails中gem Devise的使用"
date:       2018-03-07 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Ruby
  - Rails
  - Gem
---

[Devise](https://github.com/heartcombo/devise)是一套使用者认证(Authentication)套件，是Rails社群中最广为使用的一套。

#### Usage
> 在gemfile中加入

```ruby
  gem 'devise'
```

> 在终端输入

```shell
  $ bunlde install
  $ rails generate devise:install
```

- 注：确保登陆能正常跳转。在config/route.rb中加入root地址

> 在layouts加入提示信息

```html
  <p class="notice"><%= notice %></p>
  <p class="alert"><%= alert %></p>    
```

> 生成views页面文件

```shell
$ rails g devise:views
```

> 生成使用devise的model

```shell
$ rails g devise user 
$ rails db:migrate
```

#### 定义多个authentication_keys

> 在model加入以下代码

```ruby
attr_accessor :signin

def self.find_for_database_authentication(warden_conditions)
  conditions = warden_conditions.dup
  if signin = conditions.delete(:signin)
    where(conditions.to_h).where(["lower(username) = :value OR lower(mobile) = :value", { :value => signin.downcase }]).first
  elsif conditions.has_key?(:username)|| conditions.has_key?(:mobile)
    where(conditions.to_h).first
  end
end
```

> 在application_controller.rb中加入

```ruby
before_action :configure_permitted_parametersod_name, if: :devise_controller?
 
def configure_permitted_parametersod_name
  devise_parameter_sanitizer.permit(:sign_in) {|u| u.permit(:signin,:username, :mobile, :password, :remember_me)}
  devise_parameter_sanitizer.permit(:sign_up) {|u|
    u.permit(:signin,:username, :mobile, :password, :password_confirmation)}
end
```

> 最后更改initialize/devise.rb

```ruby
config.authentication_keys = [:signin]
```