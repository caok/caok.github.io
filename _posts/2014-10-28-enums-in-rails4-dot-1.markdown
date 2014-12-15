---
layout: post
title: "enums in rails4.1"
date: 2014-10-28 08:54
comments: true
categories: [Rails]
---

{% highlight ruby %}
class Conversation < ActiveRecord::Base
  enum status: [ :active, :archived ]
end
{% endhighlight %}

also you can define like this
{% highlight ruby %}
class Conversation < ActiveRecord::Base
  enum status: { active: 0, archived: 1 }
end
{% endhighlight %}

{% highlight ruby %}
# conversation.update! status: 0
conversation.active!
conversation.active? # => true
conversation.status  # => "active"

# conversation.update! status: 1
conversation.archived!
conversation.archived? # => true
conversation.status    # => "archived"

# conversation.update! status: 1
conversation.status = "archived"

# conversation.update! status: nil
conversation.status = nil
conversation.status.nil? # => true
conversation.status      # => nil

Conversation.statuses # => { "active" => 0, "archived" => 1 }
{% endhighlight %}

### Default Values for Enums
{% highlight ruby %}
create_table :conversations do |t|
  t.column :status, :integer, default: 0, null: false
end
{% endhighlight %}

### query
{% highlight ruby %}
Conversation.active
Conversation.archived
Conversation.send(my_conversation.status)

Conversation.where.not(status: 0)
Conversation.where("status <> ?", Conversation.statuses[:archived])
Conversation.where.not(status: my_conversation.status)
{% endhighlight %}

not work
{% highlight ruby %}
Conversation.where(status: my_conversation.status)
Conversation.where(status: :archived)
Conversation.where(status: "archived")
{% endhighlight %}

### Pluck vs Map with Enums
{% highlight ruby %}
statuses_with_map = Conversation.select(:status).where.not(status: nil).distinct.map(&:status)               ===>  ["active", "archived"]
statuses_with_pluck = Conversation.distinct.where.not(status: nil).pluck(:status)                            ===>  [0, 1]
{% endhighlight %}

### Enum Source
{% highlight ruby %}
conversation.status = "archived"
conversation.status = 1
# both of these work
puts conversation.status # prints "archived"
{% endhighlight %}

You can look at the [Rails source code for ActiveRecord::Enum](https://github.com/rails/rails/blob/877ea784e4cd0d539bdfbd15839ae3d28169b156/activerecord/lib/active_record/enum.rb#L82).
{% highlight ruby %}
# def status=(value) self[:status] = statuses[value] end
define_method("#{name}=") { |value|
  if enum_values.has_key?(value) || value.blank?
    # set the db value to the integer value for the enum
    self[name] = enum_values[value]
  elsif enum_values.has_value?(value) # values contains the integer
    self[name] = value
  else
    # enum_values did not have the key or value passed
    raise ArgumentError, "'#{value}' is not a valid #{name}"
  end
}
{% endhighlight %}

### References:
* http://edgeapi.rubyonrails.org/classes/ActiveRecord/Enum.html
* https://hackhands.com/ruby-on-enums-queries-and-rails-4-1/?utm_source=rubyweekly&utm_medium=email
