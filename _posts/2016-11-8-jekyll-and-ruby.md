---
layout: post
title: "Jekyll and ruby"
description: "Jekyll and ruby"
catagory: tech
tags: ruby, jekyll
---


# 安装指定版本ruby

ref link:  
http://ruby-china.org/wiki/install_ruby_guide  
http://ruby-china.org/wiki/rvm-guide  

```bash:
rvm install 2.3.0
rvm list known
```

# 安装jekyll

```bash:
sudo apt-get install ruby-dev
gem install minima -v '2.0.0'
gem install rdiscount
gem install Jekyll 
```

rdiscount 支持markdown插件

## Q&A

如果出现下面错误  
/usr/lib/ruby/1.9.1/rubygems/custom_require.rb:36:in `require': cannot load such file -- mkmf (LoadError)

解决方法：sudo apt-get install ruby-dev

## check

```bash:
gem env
root@onos:~# gem env 
RubyGems Environment:
  - RUBYGEMS VERSION: 1.8.23
  - RUBY VERSION: 1.9.3 (2013-11-22 patchlevel 484) [x86_64-linux]
  - INSTALLATION DIRECTORY: /var/lib/gems/1.9.1
  - RUBY EXECUTABLE: /usr/bin/ruby1.9.1
  - EXECUTABLE DIRECTORY: /usr/local/bin
  - RUBYGEMS PLATFORMS:
    - ruby
    - x86_64-linux
  - GEM PATHS:
     - /var/lib/gems/1.9.1
     - /root/.gem/ruby/1.9.1
  - GEM CONFIGURATION:
     - :update_sources => true
     - :verbose => true
     - :benchmark => false
     - :backtrace => false
     - :bulk_threshold => 1000
     - :sources => ["https://rubygems.org/"]
  - REMOTE SOURCES:
     - https://rubygems.org/
```

## 清除非rvm安装的ruby 

```bash:
rm -rf /usr/local/lib/ruby
rm -rf /usr/lib/ruby
rm -f /usr/local/bin/ruby
rm -f /usr/bin/ruby
rm -f /usr/local/bin/irb
rm -f /usr/bin/irb
rm -f /usr/local/bin/gem
rm -f /usr/bin/gem
```

