[![Build Status](https://secure.travis-ci.org/uploadcare/uploadcare-rails.png?branch=master)](http://travis-ci.org/uploadcare/uploadcare-rails)

[Changelog](http://changelog.uploadcare.com/tagged/rails)

**Warning!** This plugin is of alpha quality. Some things may not work or be unexpectedly slow.
You are welcome to try it with your projects and report any issues with the latest release.

# Installation

This is a Rails gem for using the [Uploadcare.com](https://uploadcare.com) service. To install it, add it in your Gemfile: `gem 'uploadcare-rails'`, then run:

    bundle install

# Usage

This gem provides ability to attach Uploadcare files to your models. To do that, you need a string field in your model, which is going to hold an Uploadcare file's CDN URL.

## Configuration

The first thing is to configure stuff for Uploadcare. Create a file `config/initializers/uploadcare.rb`:

```ruby
Uploadcare::Rails::Engine.configure do
  config.uploadcare.public_key = 'demopublickey'
  config.uploadcare.private_key = 'demoprivatekey'
  config.uploadcare.widget_version = '0.8'


  # Set the following to true if you don't want to propagate exceptions
  # that are happening while storing a file (e.g. storing a file
  # which no longer exists). They all are written to logger.error anyway.

  config.uploadcare.silence_save_errors = false

end
```

(These are our *demo* keys. You can use them to try things out. Don't forget to change them to your own keys before building anything serious).

## Models

Add an attribute to your model. First, create and run a migration:

    rails generate migration AddFileToPost file:string
    rake db:migrate

Edit your model to indicate that this field is used for Uploadcare files:

```ruby
class Post < ActiveRecord::Base
  attr_accessible :content, :name, :title, :file  # don't forget to make this attribute accessible
  is_uploadcare_file :file                        # this is the line you want to add
end
```

## Forms

To display nice widgets for file upload, include the script for desired widget version (here we use 0.8) in your layout.

```erb
<%= uploadcare_include_tag version: '0.8' %>
```

This will include the widget script from Uploadcare CDN via a tag, which is the preferred way to do it. If you omit the version argument (which is fine), the value from `config/initializers/uploadcare.rb` will be used.

In some circumstances, however, you'd want to run the script code through the asset pipeline, or work offline. To do this, instead of using `uploadcare_include_tag` just add it to your assets:

```javascript
// = require uploadcare
```

This will add the widget script to the pipeline, whichever version of it ships with the version of uploadcare-rails you're using.

Now we can use the Uploadcare widget in our forms:

```erb
<%= f.uploadcare_uploader_field :file %>

# or, if you use `simple_form`
<%= form.input :file, as: :uploadcare_uploader %>
```

## Displaying files

Uploadcare-rails gem takes care of storing files when saving a model, so as soon as you add a field to your form, everything should work.
You'll be able to show an image with `cdn_url` (or `public_url` alias) just like that:

```erb
<%= image_tag @post.file.cdn_url("scale_crop/500x200", "effect/grayscale/") %>
```

If you need the unique identifier that Uploadcare uses to represent a file,
it is available like so:

```erb
UUID: <%= @post.file.uuid %>
UUID (alias): <%= @post.file.file_id %>
```

[Other information](https://uploadcare.com/documentation/rest/#file) about the file is accessed through the `api` method that makes a single HTTP request to Uploadcare servers **for each file**. Use cautiously or cache the returned information somewhere.

```erb
File size: <%= @post.file.api.size %> byte(s)
```
