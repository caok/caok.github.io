---
layout: post
title: "State Machine"
date: 2015-06-07 16:00
categories: [Rails]
tags: [Rails]
---

在项目中时常会使用到状态机, 如果是在rails项目中使用的话，可以使用[aasm](https://github.com/aasm/aasm), 它是基于active-record. 如果不用rails的话，也有其他选项可以选择[workflow](https://github.com/geekq/workflow) 或 [state_machine](https://github.com/pluginaweek/state_machine).

{% highlight ruby %}
class Order < ActiveRecord::Base
  include AASM

  aasm do # default column: aasm_state(String)
    state :incomplete, initial: true
    state :open
    state :canceled

    event :purchase, before: :process_purchase do
      transitions from: :incomplete, to: :open, guard: :valid_payment?
    end

    event :cancel do
      transitions from: :open, to: :canceled
    end
  end

  def process_purchase
    # process order ...
  end

  def valid_payment?
    false
  end
end
{% endhighlight %}

这里面的guard起到一个判断条件的作用，如果false的话这个transition将会被拒绝(抛出AASM::InvalidTransition的错误）

这里的process_purchase是一个callback, 但是如果在after的callback中调用save或者update_attributes等，会影响到guard，导致guard的判断失效


{% highlight ruby %}
order = Order.new
order.may_purchase?            # => false
order.purchase                 # => raises AASM::InvalidTransition

order.canceled?                # => return true or false
order.cancel                   # => 会改变aasm_state的值，但不会保存
order.cancel!                  # => 会保存
order.may_purchase?            # => 等同于order.incomplete?
{% endhighlight %}

如果你想使用自己的字段的话，可以这么使用
{% highlight ruby %}
  enum state: {
    sleeping: 5,
    running: 99
  }

  aasm :column => :state, :enum => true do
    state :sleeping, :initial => true
    state :running
  end
{% endhighlight %}

如果你不想接收到exception, 只想接受true/false的返回值，可以这么来
{% highlight ruby %}
  aasm :whiny_transitions => false do
    ...
  end
{% endhighlight %}

### 参考
* http://railscasts.com/episodes/392-a-tour-of-state-machines?view=asciicast
* https://github.com/aasm/aasm
