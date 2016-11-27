---
layout: post
title: "encrypt and decrypt attributes"
date: 2014-10-27 20:04
categories: [Rails]
tags: [Rails]
---

加密保存到数据库中的字段，但不影响读写
增加一个字段encrypt_key用来保存key

{% highlight ruby %}
before_save :encrypt_note
def encrypt_note
  salt  = SecureRandom.random_bytes(64)
  key   = encrypt_key || ActiveSupport::KeyGenerator.new(ENV["SECRET_KEY"]).generate_key(salt)
  crypt = ActiveSupport::MessageEncryptor.new(key)

  %w(client_note food medicines hydration md_appointments activity mood positive_thinking positive_action listener_note).each_with_index do |att|
    val = if !new_record? and send("#{att}_changed?")
      read_attribute(att.to_sym)
    else
      send(att) || ""
    end
    send("#{att}=", crypt.encrypt_and_sign(val))
  end

  self.encrypt_key = key
end

%w(client_note food medicines hydration md_appointments activity mood positive_thinking positive_action listener_note).each do |attr|
  define_method(attr) do
    if encrypt_key.present?
      crypt = ActiveSupport::MessageEncryptor.new(encrypt_key)
      crypt.decrypt_and_verify(read_attribute(attr.to_sym))
    else
      read_attribute(attr.to_sym)
    end
  end
end
{% endhighlight %}

read_attribute(:xxx)更加底层，能忽略model中定义的与attribute同名的方法，取得数据库中该attribute真实的值

但是再这样使用过程中要注意一点，form_for时它取到的值是数据库中的值，而不是通过module中重定义后调用到的值

或者也可以尝试下[attr_encrypted](https://github.com/attr-encrypted/attr_encrypted)

Overwriting default accessors
All column values are automatically available through basic accessors on the Active Record object, but sometimes you want to specialize this behavior. This can be done by overwriting the default accessors (using the same name as the attribute) and calling read_attribute(attr_name) and write_attribute(attr_name, value) to actually change things.

{% highlight ruby %}
class Song < ActiveRecord::Base
  def length=(minutes)
    write_attribute(:length, minutes.to_i * 60)
  end
  
  def length
    read_attribute(:length) / 60
  end
end
{% endhighlight %}

You can alternatively use self[:attribute]=(value) and self[:attribute] instead of write_attribute(:attribute, value) and read_attribute(:attribute)



#### references
* https://stackoverflow.com/questions/373731/override-activerecord-attribute-methods
* http://api.rubyonrails.org/classes/ActiveSupport/MessageEncryptor.html
* http://api.rubyonrails.org/classes/ActiveRecord/Base.html
* http://www.davidverhasselt.com/set-attributes-in-activerecord/
* http://api.rubyonrails.org/files/activerecord/README_rdoc.html
* http://api.rubyonrails.org/classes/ActiveRecord/Base.html
