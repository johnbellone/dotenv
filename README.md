# dotenv [![Build Status](https://secure.travis-ci.org/bkeepers/dotenv.png)](https://travis-ci.org/bkeepers/dotenv)

Loads environment variables from `.env` into `ENV`, automagically.

Read more about the [motivation for dotenv at opensoul.org](http://opensoul.org/blog/archives/2012/07/24/dotenv/).

## Installation

### Rails

Add this line to your application's Gemfile:

```ruby
gem 'dotenv-rails', :groups => [:development, :test]
```

And then execute:

    $ bundle

### Sinatra or Plain ol' Ruby

Install the gem:

    $ gem install dotenv

As early as possible in your application bootstrap process, load `.env`:

```ruby
require 'dotenv'
Dotenv.load
```

To ensure `.env` is loaded in rake, load the tasks:

```ruby
require 'dotenv/tasks'

task :mytask => :dotenv do
    # things that require .env
end
```

## Usage

Add your application configuration to your `.env` file in the root of
your project:

```shell
S3_BUCKET=YOURS3BUCKET
SECRET_KEY=YOURSECRETKEYGOESHERE
```

You can also create files per environment, such as `.env.test`:

```shell
S3_BUCKET=tests3bucket
SECRET_KEY=testsecretkey
```

An alternate yaml-like syntax is supported:

```yaml
S3_BUCKET: yamlstyleforyours3bucket
SECRET_KEY: thisisalsoanokaysecret
```

Whenever your application loads, these variables will be available in `ENV` and a `Doten.env` helper object:

```ruby
s3 = AWS::S3.new({
  :access_key_id     => ENV['S3_ACCESS_KEY'],
  :secret_access_key => ENV['S3_SECRET_ACCESS_KEY']
})

# or

s3 = AWS::S3.new({
  :access_key_id     => Dotenv.env.s3_access_key,
  :secret_access_key => Dotenv.env.s3_secret_access_key
})
```

### Rails

In Rails, `#env` is available on the `Rails.configuration` object, models, mailers, controllers…

```ruby
class ApplicationController < ActionController::Base
  before_filter :redirect_unknown_hosts

  def redirect_unknown_hosts
    if request.host != env.app_host
      redirect_to "#{env.app_host}#{request.path}"
    end
  end
end
```

…and views.

```erb
<!-- app/views/layouts/application.html.erb -->
<script>
  // ...
  _gaq.push(['_setAccount', '<%= env.google_analytics_id %>']);
  _gaq.push(['_setDomainName', '.<%= env.app_host %>']);
  // ...
</script>
```

## Customization

Logic-less configuration is ideal, but unfortunately we don't live in an ideal world. You can add logic to your configuration by extending it with your own module:

```
# lib/my_app/config.rb
module MyApp
  module Config
    def redis_url
      self['REDISTOGO_URL'] || self['GH_REDIS_URL'] || "redis://localhost:6379"
    end
  end
end

# config/application.rb
module MyApp
  class Application < Rails::Application
    # ...

    require 'my_app/config'
    config.env.extend MyApp::Config
  end
end

# config/initializers/resque.rb
Resque.redis = Redis.connect :url => Rails.configuration.env.redis_url
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
