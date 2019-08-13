---
layout: post
title:      "User Accounts and Authentication using OAuth"
date:       2019-08-13 04:47:50 +0000
permalink:  user_accounts_and_authentication_using_oauth
---


I just created my first Ruby on Rails application that requires user accounts for access. Users may choose to create an account through the application or signup using a third-party account.

I’ll be going over how to allow your application to create an account for users that signup through a third-party and how to login users with application accounts or third-party accounts. 

In this example, I am using the [OmniAuth](“https://github.com/omniauth/omniauth”) gem as the OAuth 2.0 method and Facebook as the strategy (OmniAuth refers to providers as strategies). For the full list of OmniAuth strategies, take your pick [here](“https://github.com/omniauth/omniauth/wiki/List-of-Strategies”). 

## Gems

Let’s get started by adding the gems you’ll need in your `Gemfile`. 

```
gem 'omniauth'
gem 'omniauth-facebook'
gem 'dotenv-rails'
```

Run `bundle install`.

The second gem will depend on your strategy choice. The dotenv gem will allow you to load the variables from a .env file into ENV when the environment is bootstrapped.

## Files

Next, create a new file `config/initializers/omniauth.rb`. This is where you’ll place your app’s OAuth credentials. 

```
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :facebook, ENV['FACEBOOK_KEY'], ENV['FACEBOOK_SECRET']
end
```

Create a new file `.env` at the root of your application. **Add `.env` to your `.gitignore` file to make sure your credentials are not accidentally commited.**

Get your strategy credentials and place them in `.env`.

```
FACEBOOK_KEY=pasteyourstrategykeyhere
FACEBOOK_SECRET=pasteyourstrategysecrethere
```

## Route

To allow your user to login or signup using the third-party account, you’ll need to redirect them to `/auth/:provider`, where `:provider` is the strategy name. Provide this link on your signup and login page. 

Assuming you have your user model and a SessionsController set up, add the following to `config/routes.rb`.

```
get '/auth/:provider/callback', to: 'sessions#create'
```

This route is where the strategy will redirect users after they have been successfully authenticated.

## Custom User Class Method

After the user is successfully authenticated through the strategy, the user’s information will be available in the Authentication Hash, which can be accessed through `request.env['omniauth.auth']`. You can place it in a private method in the SessionsController like so. 

```
# sessions_controller.rb

private

def auth_hash
        request.env['omniauth.auth']
end
```

Next, create a User class method to use the information in the `auth_hash`. We’ll use the user’s information to find their account (if they have one) or create a new account for them (if they don’t have one). 

```
# user.rb

def self.find_or_create_by_omniauth(auth_hash)
        self.find_or_create_by(uid: auth['uid']) do |u|
            u.name = auth['info']['name']
u.email = auth['info']['email']
            u.password = SecureRandom.hex
        end
end
```

More user information provided by the Authentication Hash, with Facebook as the strategy, can be viewed [here](“https://github.com/mkdynamic/omniauth-facebook#auth-hash”).

In the example user model above, we’re using `SecureRandom.hex` to generate a secure and random password for the user. If you’re using the BCrypt gem and the [has_secure_password](“https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password”) method in your user model, a user must have a password in order to be persisted to the database. `SecureRandom.hex` solves that problem for us. 

**Tip:** If you’re using BCrypt and `has_secure_password`, don’t forget to use `:password_digest` in your migration, instead of `:password`.

## SessionsController

Now, we can set up the `create` action in our SessionsController to handle both normal user account login and third-party account login or signup. 

```
# sessions_controller.rb

def create 
        if auth_hash
            @user = User.find_or_create_by_omniauth(auth_hash)
            session[:user_id] = @user.id
            render 'users/show'
        else
            @user = User.find_by_email(params[:user][:email])
            if @user && @user.authenticate(params[:user][:password])
                session[:user_id] = @user.id
                render 'users/show'
            else
                flash[:message] = "Incorrect login information."
                redirect_to '/login'
            end
        end
    end

private

    def auth_hash
        request.env['omniauth.auth']
    end
```

If a user uses a third-party account, an Authentication Hash will be available. `if auth_hash` will be true and our User class method `find_or_create_by_omniauth` will be triggered. The user’s information will then be available in `@user` and logged in.

If `auth_hash` is false, that means we have a normal user logging in through our application. We will attempt to find the user by their email address. If the user exists and the password submitted is authenticated (using `authenticate` provided to us through the `has_secure_password` method), the user will be logged in. 

There you have it! You can now login normal users and third-party authenticated users. I hope this has helped you get OmniAuth up and running for your application! 
