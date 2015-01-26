---
layout: post
title: "A Guide to The Rails Command Line"
date: 2012-05-13 21:09
comments: true
categories: [Rails]
---

rails
-----
## 1.new and server
With just three commands we whipped up a Rails server listening on port 3000

    rails new commandsapp
    cd commandsapp
    rails server(rails s)

> rails server -e production -p 4000

> rails server --help

## 2.generate

    rails generate controller NAME [action action] [options]
    eg: rails generate controller Say hello goodbye

    rails generate model NAME [field:type field:type] [options]
    eg: rails generate model post title:string body:text published:boolean

    rails generate scaffold HighScore user:references game:string score:integer
    rails generate migration add_quantity_to_production quantity:integer
> the common data definition tasks
[detail](http://guides.rubyonrails.org/migrations.html)

    add_column
    add_index
    change_column
    change_table
    create_table
    drop_table
    remove_column
    remove_index
    rename_column

## 3.other

    rails console(rails c)
    rails dbconsole(rails db)
    rails plugin

    rails runner(rails r)
    eg: rails runner "Model.long_running_method"

    rails destroy(rails d)
    eg: rails generate model Oops
        rails destroy model Oops

Rake
------

    rake --tasks(rake -T)  #show a list of Rake tasks

### 1.about

    rake about    #About your application's environment

### 2.assets

    rake assets:clean          # Remove compiled assets
    rake assets:precompile     # Compile all the assets named in config.assets.precompile

### 3.db

    rake db:create             # Create the database from config/database.yml for the current Rails.env
    rake db:create:all         # Create all dbs

    rake db:drop               # Drops the database for the current Rails.env (use db:drop:all to drop all databases)
    rake db:fixtures:load      # Load fixtures into the current environment's database.

    rake db:migrate            # Migrate the database (options: VERSION=x, VERBOSE=false).
    rake db:migrate:status     # Display status of migrations
    rake db:migrate:down VERSION=..........
    rake db:migrate redo

    rake db:perf:all           # recreate table and test data
    rake db:perf:perf          # populate test data
    rake db:perf:prepare       # recreate table structure for populating data

    rake db:populate           # recreate table and test data
    rake db:populate:all       # recreate table and test data
    rake db:populate:populate  # populate test data
    rake db:populate:prepare   # recreate table structure for populating data

    rake db:rollback           # Rolls the schema back to the previous version (specify steps w/ STEP=n) eg:rake db:rollback STEP=3.
    rake db:schema:dump        # Create a db/schema.rb file that can be portably used against any DB supported by AR
    rake db:schema:load        # Load a schema.rb file into the database

    rake db:seed               # Load the seed data from db/seeds.rb
    rake db:setup              # Create the database, load the schema, and initialize with the seed data
    rake db:reset
    rake db:structure:dump     # Dump the database structure to db/structure.sql. Specify another file with DB_STRUCTURE=db/my_structure.sql
    rake db:version            # Retrieves the current schema version number

### 4.routes

    rake routes                # list all of your defined routes

### 5.tmp

    rake tmp:cache:clear       # clears tmp/cache.
    rake tmp:sessions:clear    # clears tmp/sessions.
    rake tmp:sockets:clear     # clears tmp/sockets.
    rake tmp:clear             # clears all the three: cache, sessions and sockets.

### 6.spec

    rake spec                  # Run all specs in spec directory (excluding plugin specs)
    rake spec:controllers      # Run the code examples in spec/controllers
    rake spec:helpers          # Run the code examples in spec/helpers
    rake spec:lib              # Run the code examples in spec/lib
    rake spec:mailers          # Run the code examples in spec/mailers
    rake spec:models           # Run the code examples in spec/models
    rake spec:rcov             # Run all specs with rcov
    rake spec:requests         # Run the code examples in spec/requests
    rake spec:routing          # Run the code examples in spec/routing
    rake spec:views            # Run the code examples in spec/views

### 7.log

    rake log:clear             # Truncates all *.log files in log/ to zero bytes

### 8.stats

    rake stats                 # show the detail of code(lines,classes,methods...)

[detail](http://guides.rubyonrails.org/command_line.html)

gem
---

    gem -v                                                # gem版本
    gem update                                            # 更新所有包
    gem update --system                                   # 更新RubyGems软件

    gem install rake                                      # 安装rake,从本地或远程服务器
    gem install rake --remote                             # 安装rake,从远程服务器
    gem install watir -v(或者--version) 1.6.2              # 指定安装版本的
    gem uninstall rake                                    # 卸载rake包

    gem list                                              # 列出所有安装的gem
    gem list d                                            # 列出本地以d打头的包
    gem query -n '[0-9]' --local                          # 查找本地含有数字的包
    gem search log --both                                 # 从本地和远程服务器上查找含有log字符串的包
    gem search log --remoter                              # 只从远程服务器上查找含有log字符串的包
    gem search -r log                                     # 只从远程服务器上查找含有log字符串的包

    gem help                                              # 提醒式的帮助
    gem help install                                      # 列出install命令 帮助
    gem help examples                                     # 列出gem命令使用一些例子

    gem build rake.gemspec                                # 把rake.gemspec编译成rake.gem
    gem check -v pkg/rake-0.4.0.gem                       # 检测rake是否有效
    gem cleanup                                           # 清除所有包旧版本，保留最新版本
    gem contents rake                                     # 显示rake包中所包含的文件
    gem dependency rails -v 0.10.1                        # 列出与rails相互依赖的包
    gem environment                                       # 查看gem的环境
    gem list | cut -d" " -f1 | xargs gem uninstall -aIx   # 删除所有安装的gem
