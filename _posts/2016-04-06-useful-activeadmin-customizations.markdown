---
layout: post
title: "Useful ActiveAdmin Customizations"
date: 2016-04-06 13:00
---

在日常的应用的经常会遇到需要一个简单的管理后台, 而[ActiveAdmin](http://activeadmin.info/)常会成为我们的首选。这里我们搜集些可能会用到的配置

{% highlight ruby %}
  menu :priority => 8

  actions :all, except: [:new, :create, :edit, :update]
  config.sort_order = "id_desc"

  scope 'Clicks' do
    TuneLog.where(log_type: TuneLog.log_types[:click])
  end
{% endhighlight %}

### Dynamic Site Title
{% highlight ruby %}
# config/initializers/active_admin.rb
config.site_title = proc { "#{current_site.name} CMS" }
{% endhighlight %}




首先我们要在Gemfile加入两个gem做准备
{% highlight ruby %}
gem 'stripe'
gem 'figaro'
{% endhighlight %}
这里的[figaro](https://github.com/laserlemon/figaro)主要是为了STRIPE_PUBLISHABLE_KEY和STRIPE_SECRET_KEY
{% highlight ruby %}
development:
  STRIPE_SECRET_KEY: '******'
  STRIPE_PUBLISHABLE_KEY: '******'
{% endhighlight %}

{% highlight ruby %}
def create
  token = params[:stripeToken]

  begin
    Stripe.api_key = ENV["STRIPE_SECRET_KEY"]
    charge = Stripe::Charge.create(
      amount:      params[:amount],
      currency:    "usd",
      card:        token,
      description: "Rails Stripe customer"
    )

    # do something to record this transaction in local

    redirect_to xxx_path, notice: "Pay successful."
  rescue Stripe::CardError => e
    # The card has been declined or some other error has occurred
    redirect_to xxx_path, alert: e
  end
end
{% endhighlight %}
这里有一些注意点:

* 1.amount是以美分为单位的，所以要除以100后才是美元
* 2.stripe支持的最小金额是50美分，低于这个值是无法付款成功的
* 3.这里的token和amount都尽量从view传递回来，方便创建自己的本地交易记录

{% highlight html %}
<%= form_tag charges_path, id: "pay_charge", method: :post do %>
  <%= hidden_field_tag 'stripeToken', '' %>
  <%= hidden_field_tag 'stripeEmail', '' %>
  <%= hidden_field_tag 'amount', 1000 %>
  <%= submit_tag 'Pay', class: 'button', id: "check-out" %>
<% end %>

<script src="https://checkout.stripe.com/checkout.js"></script>
<script>
  var handler = StripeCheckout.configure({
    key: "<%= ENV['STRIPE_PUBLISHABLE_KEY'] %>",
    //image: 'logo.png',
    token: function(token) {                 #这里可以理解为一个回调
      $("#stripeToken").val(token.id);
      $("#stripeEmail").val(token.email);
      $("form#pay_charge").submit();
    }
  });

  $('#check-out').on('click', function(e) {
    handler.open({
      name: "clark's blog",
      description: "demo site",
      amount: 1000          #注意是美分 
    });
    e.preventDefault();
  });

  // Close Checkout on page navigation
  $(window).on('popstate', function() {
    handler.close();
  });
</script>
{% endhighlight %}

当用户输入完信用卡信息并检查通过后，"回调"就会获取到[token](https://stripe.com/docs/api#tokens)中的一些信息, 比如后台里需要的stripeToken

{% highlight ruby %}
The callback to invoke when the Checkout process is complete. 
function(token) 
token is the token object created. 
token.id can be used to create a charge or customer. 
token.email contains the email address entered by the user.
{% endhighlight %}

记住那怕view中handler.open等更多的是执行了一个信用卡信息的检查，不会设计到扣款，真正的扣款是controller中的“Stripe::Charge.create”

### 备注:
* https://www.viget.com/articles/8-insanely-useful-activeadmin-customizations
