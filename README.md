### Crepe
---

https://github.com/crepe/crepe

```
gem 'crepe', github: 'crepe/crepe'
gem install creperie --pre
crepe new my_api
```

```ruby
# config.ru
require 'bundler/setup'
require 'crepe'

module Gist
  class API < Crepe::API
    let(:current_user){ User.authorize!(request.headers['Authorization']) }
    namespace :gists do
      let(){}
      get do
        begin
        rescue User::Unauthorized
        end
      end
      post {}
      get(){}
      get(){}
      param :id do
        let(){}
        get {}
        put {}
        patch {}
        delete do
          if gist.user == current_user
          else
          end
        end
      end
    end
  end
end

```

