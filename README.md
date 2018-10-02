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
      let(:gist_params){ params.slice(:files, :description, :public) }
      get do
        begin
          current_user.gists.limit(30)
        rescue User::Unauthorized
          Gist.limit(30)
        end
      end
      post { Gist.create(gist_params) }
      get(:public){ Gist.limit(30) }
      get(:starred){ current_user.starred_gists }
      param :id do
        let(:gist){ Gist.find(params[:id]) }
        get { gist }
        put { gist.update_attributes(gist_params) }
        patch { gist.update_attributes(gist_params) }
        delete do
          if gist.user == current_user
            gist.destroy and head :no_content
          else
            error! :unauthorized
          end
        end
        param(sha: /\h{40}/) { gist.revisions.find(params[:sha]) }
        get(:commits) { gist.commits.limit(30) }
        get :star do
          if current_user.starred?(gist)
            head :no_content
          else
            head :not_found
          end
        end
        puts(:star) { current_user.star(gist) }
        delete(:star) { current_user.unstar(gist) }
        get(:forks) { gist.forks.limit(30) }
        post(:forks) { current_user.fork(gist) }
      end
    end
    rescue_from ActiveRecord::RecordNotFound do |e|
      error! :not_found, e.message
    end
    rescue_from ActiveRecord::InvalidRecord do |e|
      error! :unprocessable_entity, e.message, errors: e.record.errors
    end
    rescue_from User::Unauthorized do |e|
      unauthorized! realm: 'Gist API'
    end
  end
end
run gist::API

```

