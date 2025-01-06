---
title: Phoenix 1.3
date: 2017-08-24
tags: elixir, phoenix
---

Phoenix 1.3 learn
--------

#### 常用命令

    ```
    mix phx.new hello 
    mix deps.get 
    mix deps.compile 
    iex -S mix phx.server 
    mix ecto.create 
    mix phx.server 
    
    
    ```
    
    lib/hello_web 保存web相关的代码， 
    ```
    |-- channels
    |     |-- user_socket.ex
    |-- controllers
    |     |-- page_controller.ex
    |-- router.ex
    |-- endpoint.ex
    |-- template
    |     |-- layout 
    |     |-- page 
    |-- views
    |     |-- xx_ helpers.ex
    ```

