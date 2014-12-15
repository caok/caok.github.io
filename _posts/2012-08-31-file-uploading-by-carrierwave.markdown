---
layout: post
title: "file uploading by carrierwave"
date: 2012-08-31 18:13
comments: true
categories: [Rails]
---

#### 1.安装
在gemfile中
    # Attachment
    gem 'carrierwave'
    gem 'mini_magick'
    gem 'mime-types'

<!-- more -->

> bundle install

#### 2.生成attachment
    rails g model attachment file_name:string content_type:string file_size:string attachmentable_type:string attachmentable_id:integer attachment:string

> rake db:migrate

#### 3.生成一个uploader
    rails g uploader Attachment

#### 4.配置attachment_uploader
```
# attachment_uploader.rb
# encoding: utf-8
class AttachmentUploader < CarrierWave::Uploader::Base

  include CarrierWave::MiniMagick
  include CarrierWave::MimeTypes

  # Include the Sprockets helpers for Rails 3.1+ asset pipeline compatibility:
  include Sprockets::Helpers::RailsHelper
  include Sprockets::Helpers::IsolatedHelper

  # Choose what kind of storage to use for this uploader:
  storage :file
  # storage :fog

  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  process :set_content_type

  version :thumb, :if => :image? do
    process :resize_to_fit => [90, 90]
  end

  protected
  def image?(new_file)
    new_file.content_type.include? 'image'
  end
end
```

#### 4.修改model
```
# attachment.rb
class Attachment < ActiveRecord::Base
  attr_accessible :attachemtable_id, :attachment, :attachmentable_type, :content_type, :file_name, :file_size, :attachmentable

  mount_uploader :attachment, AttachmentUploader

  belongs_to :attachmentable, :polymorphic => true

  validates :attachmentable, :presence => true
  validates :attachment, :presence => true

  before_save :set_attachment_attributes

  protected
  def set_attachment_attributes
    if attachment.present? && attachment_changed?
      self.content_type = attachment.file.content_type
      self.file_size = attachment.file.size
      self.file_name = attachment.file.original_filename
    end
  end
end
```

在相关联的model中增加（company）
```
#company.rb
attr_accessor :file
has_one :attachment, :as => :attachmentable
```

#### 5.页面附件上载的支持
view中
```
# _form.html.erb
<%= form_for @company, :html => { :multipart => true } do |f| %>
  <div class="field">
    <%= file_field_tag :file %>
  </div>
  <%= f.submit %>
<% end %>
```

如果使用haml格式
```
#_form.html.haml
= semantic_form_for(resource, :html => {:multipart => true}) do |f|
  = render "shared/error_messages", :target => resource
  = f.inputs do
    = f.input :file, :as => :file, :label => "附件"
  = f.actions
```

相应的需要在在create或update的action中增加
    Attachment.create(:attachment => params[:company][:file], :attachmenttable => @company) if params[:company][:file]
这样在创建或是更新的时候就能将图片增加进去

#### 6.我这里将新增和删除图片的操作给独立开来
```
resources :companies do
  delete :remove_attachment, :on => :member
  get :attachment, :on => :member
  put :add_attachment, :on => :member
end
```

```
#company_controller.rb
def attachment
  @company = Company.find(params[:id])
end

def add_attachment
  @company = Company.find(params[:id])
  Attachment.create(:attachment => params[:file], :attachmentable => @company) if params[:file]

  respond_to do |format|
    format.html { redirect_to attachment_company_path(@company) }
  end
end

def remove_attachment
  @company = Company.find(params[:id])
  attachment = @company.attachments.find(params[:attachment])
  attachment.destroy

  respond_to do |format|
    format.html { redirect_to attachment_company_path(@company) }
  end
end
```

view的attachment.html.erb中
图片的显示和删除
```
<% if @company.attachments.size > 0 -%>
  <% @company.attachments.each do |attachment| -%>
    <%= link_to image_tag(attachment.attachment.url(:thumb)), attachment.attachment.url %>
    <%= button_to "删除附件",
                remove_attachment_company_path(@company, :attachment => attachment),
                :method => :delete,
                :data => { :confirm => t('.confirm', :default => t("helpers.links.confirm", :default => 'Are you sure?')) },
                :class => 'btn btn-mini btn-danger' %>
  <% end-%>
<% end -%>
```

图片的添加
```
<%= form_for @company, :url => add_attachment_company_path(@company), :html => { :multipart => true } do |f| %>
    <%= f.label :file %>
    <%= file_field_tag :file %>
  <%= f.submit %>
<% end %>
```
