---
title: rails-time-zone
date: 2019-05-21
tags: pratice
---

# Rails Timezone

TimeZone 是 rails 为ruby扩展的一个 有用工具。

application.rb
config.time_zone = "Beijing"
可以指定整个应用的timezone。 然而， 使用timezone依然需要一个特定的方法调用，才能够使用rails 提供的扩展。

相关代码如下:

activesupport/lib/active_support/core_ext/time/zones.rb

为 ruby 的TIme class 提供了， zone, zone=, use_zone（在临时设定timezone，并yield之后，重置 timezone）, fine_zone！， in_time_zone(转到换 一个timezone的时间表示) 扩展，


zone 为一个ActiveSupport::TimeZone instance, 提供now, today, at, utc_to_local 等， 配置， timezones简单别名映射
ActiveSupport::TimeWithZone, 则是简单的一个包含timezone， utc time， 的封装， 包含了一些对界面的映射， to_s, to_yml, httpdate, strftime （其中包含了 ::Timezone 表示） 等
