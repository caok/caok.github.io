---
layout: post
title: "Optimistic Locking"
date: 2013-12-12 00:22
comments: true
categories: [Rails]
---

业务逻辑的实现过程中，往往需要保证数据访问的排他性。此时，我们就需要通过一些机制来保证这些数据在某个操作过程中不会被外界修改，这样的机制，在这里，也就是所谓的“ 锁 ”，即给我们选定的目标数据上锁，使其无法被其他程序修改。

#### Optimistic Locking(乐观锁)的两种实现方式:
* 使用自增长的整数表示数据版本号。更新时检查版本号是否一致，比如数据库中数据版本为6，更新提交时version=6+1,使用该version值(=7)与数据库version+1(=7)作比较，如果相等，则可以更新，如果不等则有可能其他程序已更新该记录，所以返回错误。
* 使用时间戳来实现.

<!-- more -->

当两个人在同一时间修改同一条记录，其中一个人的修改可能就会被另外一个人所覆盖。解决这种问题的一种方法就是[Optimistic Locking](http://railscasts.com/episodes/59-optimistic-locking-revised)。

[Optimistic locking](http://api.rubyonrails.org/classes/ActiveRecord/Locking/Optimistic.html)允许多个用户编辑同一条记录并假定一个最小的数据冲突。当某条记录被打开时，它通过检查是否有另外一个进程在对这条记录进行修改，如果这种情况发生了，将会抛出一个ActiveRecord::StaleObjectError的异常，并且update操作将会被忽略。

### 1.lock_version(自增长的数据版本号)
rails通过lock_version来激活optimistic locking.
(Active Records support optimistic locking if the field lock_version is present.)

所以我们需要增加一个lock_version的字段
```ruby
add_column :products, :lock_version, :integer, :default => 0, :null => false
```
```ruby _form.html
<%= f.hidden_field :lock_version %>
```
记录更新时增加对ActiveRecord::StaleObjectError异常的捕捉
```ruby products_controller.rb
def update
  @product = Product.find(params[:id])
  if @product.update_with_conflict_validation(params[:product])
    redirect_to @product, notice: "Updated product."
  else
    render :edit
  end 
end 
```
```ruby product.rb
class Product < ActiveRecord::Base
  belongs_to :category
  attr_accessible :name, :price, :released_on, :category_id, :lock_version
  
  def update_with_conflict_validation(*args)
    update_attributes(*args)
  rescue ActiveRecord::StaleObjectError
    self.lock_version = lock_version_was
    errors.add :base, "This record changed while you were editing."
    changes.except("updated_at").each do |name, values|
      errors.add name, "was #{values.first}"
    end
    false
  end
end
```
如果A和B同时在修改product(lock_version为0)，A修改好后保存product，此时lock_version就从0变为1(而B那还未保存，所以还是为0)。上面的代码中当捕捉到异常时，self.lock_version为0,而lock_version_was为1。(而正常情况下lock_version和lock_version_was是一致的)

### 2.时间戳
```ruby _form.html
<%= f.hidden_field :original_updated_at %>
```
```ruby product.rb
class Product < ActiveRecord::Base
  belongs_to :category
  attr_accessible :name, :price, :released_on, :category_id, :original_updated_at
  validate :handle_conflict, only: :update

  def original_updated_at
    @original_updated_at || updated_at.to_f
  end
  attr_writer :original_updated_at

  def handle_conflict
    if @conflict || updated_at.to_f > original_updated_at.to_f
      @conflict = true
      @original_updated_at = nil
      errors.add :base, "This record changed while you were editing. Take these changes into account and submit it again."
      changes.each do |attribute, values|
        errors.add attribute, "was #{values.first}"
      end
    end
  end
end
```

### 参考:
* http://railscasts.com/episodes/59-optimistic-locking-revised
* http://railscasts.com/episodes/59-optimistic-locking
* http://api.rubyonrails.org/classes/ActiveRecord/Locking/Optimistic.html
* http://www.myexception.cn/database/511804.html
