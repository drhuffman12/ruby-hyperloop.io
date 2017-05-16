---
title: Advanced configuration with Ruby On Rails
---


# Advanced configuration with Ruby On Rails


## Initializer

Hyperloop generator adds an initializer file:

```ruby
#config/initializers/hyperloop.rb

Hyperloop.configuration do |config|
  config.transport = :simple_poller
end
```

The parameter `config.transport = :simple_poller` allows us to configure the way push notifications are configured.

Hyperloop implements push notifications (via a number of possible technologies) so changes to records on the server are dynamically pushed to all authorized clients.

More detail in the documentation: [{ Configuring the Transport }](/docs/models/configuring-transport)


## Policies

Hyperloop generator adds a policy default file:

```ruby
#app/policies/application_policy.rb

class Hyperloop::ApplicationPolicy
  # Allow any session to connect:
  always_allow_connection
  # Send all attributes from all public models
  regulate_all_broadcasts { |policy| policy.send_all }
  # Allow all changes to public models
  allow_change(to: :all, on: [:create, :update, :destroy]) { true }
end unless Rails.env.production?
```

Hyperloop uses Policies to regulate what connections are opened between clients and the server and what data is distributed over those connections.

More detail in the documentation: [{ Configuring the policies }](/docs/policies/authorization)


## Hyperloop architectures

By default the hyperloop install generator creates the hyperloop structure inside the `/app` directory :

```
/app/hyperloop/
/app/hyperloop/components
/app/hyperloop/models
/app/hyperloop/operations
/app/hyperloop/stores
```

If for any reasons you want to change those directories places you can do that using the config parameter `config.hyperloop.auto_config = false`.

You can find the complete source code of a Helloworld sample Hyperloop app using the advanced configuration parameters here: [{ Hyperloop with Advanced configuration }](https://github.com/ruby-hyperloop/hyperloop-rails-helloworld-advancedconfig)

For example, let's say you want this architecture:

+ Components in the `app/views/components` directory
+ Models in `app/models/public` directory
+ Operations in `myhyperloop/operations` directory
+ Stores in `myhyperloop/stores` directory

#### First you have to modify some configurations files

```ruby
#config/application.rb

config.hyperloop.auto_config = false

config.eager_load_paths += %W(#{config.root}/app/models/public)
config.autoload_paths += %W(#{config.root}/app/models/public)
config.eager_load_paths += %W(#{config.root}/app/myhyperloop/operations)
config.autoload_paths += %W(#{config.root}/app/myhyperloop/operations)

config.assets.paths << ::Rails.root.join('app', 'myhyperloop').to_s
config.assets.paths << ::Rails.root.join('app', 'models').to_s
```

```
#app/assets/javascripts/application.js

//= require 'react_ujs'
//= require 'jquery'
//= require 'jquery_ujs'
//= require 'turbolinks'
//= require_tree .

//= require 'components'
//= require 'myhyperloop'
Opal.load('components');
Opal.load('myhyperloop');
```

```
#app/views/components.rb

require 'opal'

require 'react/react-source-browser'
require 'react/react-source-server'

require 'hyper-component'

if React::IsomorphicHelpers.on_opal_client?
  require 'opal-jquery'
  require 'browser'
  require 'browser/interval'
  require 'browser/delay'
end

require 'hyper-model'
require 'hyper-store'
require 'hyper-operation'
require 'models'

require_tree './components'

```

```
#app/models/models.rb

require_tree './public' if RUBY_ENGINE == 'opal'
```


#### Your Components


```
#app/views/components/home/helloworld.rb

module Components
  module Home
  	class Helloworld < Hyperloop::Component
  		...
  	end
   end
end
```

#### Your models

```
#app/models/public/helloworld.rb

class Helloworld < ApplicationRecord
end

```

```
#app/models/public/application_record.rb

class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
end
```


#### Your operations and Stores

```
app/myhyperloop
app/myhyperloop/operations
app/nyhyperloop/stores
```

```
#app/myhyperloop/myhyperloop.rb

require_tree './operations'
require_tree './stores'
```


