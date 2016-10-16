### Mongoid

* Mongoid是一个用**ruby**写的针对**MongoDB**的Object-Document-Mapper
* Mongoid混合了Active Record和**MongoDB**的==schema-less and performant==
* Dynamic queries, and atomic modifier operations

---

### Mongoid - Installation

* Add Mongoid to your *Gemfile*
  * gem 'mongoid', '~> 5.0.1'

---

### Mongoid Cofiguration

* rails g mongoid:config
* 上述命令生成一个**config** file
* 位置在appname/config/mongoid.yml
* YML 文件会自动生成两个profile:
  * appname_development (开发时用)
  * appname_test（测试时用）
  * *同时可以添加一个production*（发布时使用）

---

### Mongoid.yml中的文件

1. 在development模式下：

```ruby
development:
  # Configure available database clients. (required)
  clients:
    # Defines the default client. (required)
    default:
      # Defines the name of the default database that Mongoid can connect to.
      # (required).
      database: movies_development
      # Provides the hosts the default client can connect to. Must be an array
      # of host:port pairs. (required)
      hosts:
        - localhost:27017
      options:
```

2. 在test模式下

```ruby
test:
  clients:
    default:
      database: movies_test
      hosts:
        - localhost:27017
      options:
        read:
          mode: :primary
        max_pool_size: 1
```

---

### Application.rb

这里可以看到使用**mongoid**之后默认的是使用mongoid model generator,但是你可以随意的去转换到active_record model generator

```ruby
require File.expand_path('../boot', __FILE__)

require 'rails/all'

# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(*Rails.groups)

module Movies
  class Application < Rails::Application
    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration should go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded.

    # Set Time.zone default to the specified zone and make Active Record auto-convert to this zone.
    # Run "rake -D time" for a list of tasks for finding time zone names. Default is UTC.
    # config.time_zone = 'Central Time (US & Canada)'

    # The default locale is :en and all translations from config/locales/*.rb,yml are auto loaded.
    # config.i18n.load_path += Dir[Rails.root.join('my', 'locales', '*.{rb,yml}').to_s]
    # config.i18n.default_locale = :de

    #bootstraps mongoid within applications -- like rails console
    Mongoid.load!('./config/mongoid.yml')

    #mongoid gem configures mongoid as default model generator
    #this can make it explicit or switch back to active_record default
    #config.generators {|g| g.orm :mongoid}
    #config.generators {|g| g.orm :active_record}

    # Do not swallow errors in after_commit/after_rollback callbacks.
    config.active_record.raise_in_transactional_callbacks = true
  end
end

```

