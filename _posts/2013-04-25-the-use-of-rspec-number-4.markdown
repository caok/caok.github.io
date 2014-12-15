---
layout: post
title: "testing rails app #4"
date: 2013-04-25 20:32
comments: true
categories: [Rails, Rspec, Test]
---

前几篇中简单介绍了下rspec的基本用法:

* [testing rails app #1](http://caok1231.com/blog/2013/04/10/testing-rails-app/)
* [testing rails app #2](http://caok1231.com/blog/2013/04/15/the-use-of-spec-in-rails/)
* [testing rails app #3](http://caok1231.com/blog/2013/04/17/the-use-of-rspec-number-3/)

现在介绍下rspec在rails的controller中一些常见的测试例子

<!-- more -->
事先的准备(这里假定对user的firstname做了限定，不允许为空)：
```ruby
FactoryGirl.define do
  factory :user do
    sequence(:firstname) { |n| "user#{n}" }
    lastname ['jack', 'lucy', 'dave', 'lily', 'john', 'beth'].sample
 
    factory :invalid_user do
      firstname nil 
    end 
  end
end
```
### 1.Testing GET methods
```ruby
let(:user) { create :user }

describe "GET #index" do
  it "populates an array of users" do
    user = create :user
    get :index
    assigns(:users).should eq([user])
  end
  
  it "renders the :index view" do
    get :index
    response.should render_template :index
  end
end

describe "GET #show" do
  it "assigns the requested user to user" do
    get :show, id: user
    assigns(:user).should eq(user)
  end
  
  it "renders the #show view" do
    get :show, id: user.id
    response.should render_template :show
  end
end
```
attributes_for:用来生成一个包含了指定对象的值的hash

id: user 等同与 id: user.id

### 2.Testing POST methods
```ruby
describe "POST create" do
  context "with valid attributes" do
    let(:valid_attributes) { attributes_for(:user) }
    it "creates a new user" do
      expect{
        post :create, user: valid_attributes
      }.to change(User,:count).by(1)
    end
    
    it "redirects to the new user" do
      post :create, user: valid_attributes
      response.should redirect_to User.last
    end
  end
  
  context "with invalid attributes" do
    let(:invalid_user) { attributes_for(:invalid_user) }
    it "does not save the new user" do
      expect{
        post :create, user: invalid_user 
      }.to_not change(User,:count)
    end
    
    it "re-renders the new method" do
      post :create, user: invalid_user
      response.should render_template :new
    end
  end 
end
```

### 3.Testing PUT methods
```ruby
describe 'PUT update' do
  let(:user) { create :user, firstname: "Lawrence", lastname: "Smith" }
  
  context "valid attributes" do
    let(:valid_attributes) { attributes_for(:user) }

    it "located the requested user" do
      put :update, id: user, user: valid_attributes
      assigns(:user).should eq(user)      
    end
  
    it "changes user's attributes" do
      put :update, id: user, 
        user: Factory.attributes_for(:user, firstname: "Larry", lastname: "Smith")
      user.reload
      user.firstname.should eq("Larry")
      user.lastname.should eq("Smith")
    end
  
    it "redirects to the updated user" do
      put :update, id: user, user: valid_attributes
      response.should redirect_to user
    end
  end
  
  context "invalid attributes" do
    let(:invalid_attributes) { attributes_for(:invalid_user) }

    it "locates the requested user" do
      put :update, id: user, user: invalid_attributes
      assigns(:user).should eq(user)      
    end
    
    it "does not change user's attributes" do
      put :update, id: user, 
        user: Factory.attributes_for(:user, firstname: nil, lastname: "Jack")
      user.reload
      user.firstname.should eq("Lawrence")
      user.lastname.should_not eq("Jack")
    end
    
    it "re-renders the edit method" do
      put :update, id: user, user: invalid_attributes
      response.should render_template :edit
    end
  end
end
```

### 4.Testing DELETE methods
```ruby
describe 'DELETE destroy' do
  let(:user) { create :user }

  it "deletes the user" do
    expect{
      delete :destroy, id: user        
    }.to change(User,:count).by(-1)
  end
    
  it "redirects to users#index" do
    delete :destroy, id: user
    response.should redirect_to users_url
  end
end
```

#### 参考：
* http://rubydoc.info/gems/rspec-rails/frames
* http://railscasts.com/episodes/71-testing-controllers-with-rspec
* http://railscasts.com/episodes/157-rspec-matchers-macros
* http://everydayrails.com/2012/04/07/testing-series-rspec-controllers.html
