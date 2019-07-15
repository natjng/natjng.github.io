---
layout: post
title:      "Using Sinatra and Active Record to Build a Content Management System"
date:       2019-07-14 23:16:31 -0400
permalink:  using_sinatra_and_active_record_to_build_a_content_management_system
---


For my second coding project, I used Sinatra to build a Content Management System, Wedding Registry, to keep track of registry items. 

I used the Model-View-Controller (MVC) paradigm to organize my application and separate concerns. Active Record was used as an ORM framework to create a database, manage database schema, create associations between my models, query my database, and more. 

### Model

Users can add items to their registry and categorize those items. This was done by creating three models: User, Item, and Category. A many-to-many connection was created, making Item the join model.  

The BCrypt gem is used to store a salted, hashed version of User passwords to the database in a `password_digest` column. The `has_secure_password` method ([read more about it here](http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password)) allows us to access that column with the attribute `password`.

```
class User < ActiveRecord::Base
    has_secure_password
    has_many :items
    has_many :categories, through: :items
end

class Item < ActiveRecord::Base
    belongs_to :user
    belongs_to :category
end

class Category < ActiveRecord::Base
    has_many :items
    has_many :users, through: :items
end
```

### Controller

The application controller is separated by domains. Each domain uses RESTful routing for CRUD actions. 

The Wedding Registry app requires user accounts. Users must create an account with a unique username. `User.find_by(username: params[:username]` is used to check if the username already exists in the database. If the username exists, the account will not be created and the user will be prompted to enter new information. Users and their passwords, items, and categories may not be blank. This is ensured by redirecting the user to re-enter the information again.

### Views

Views are protected by requiring users to be logged in and by ensuring the resource belongs to the logged in user. 

When a user successfully logs in, the key `:user_id`’s value is set to the current user’s ID in the session hash. 

```
session[:user_id] = @user.id
```
A helper method was created to store the User object in an instance variable if a user is logged in.
```
helpers do
def current_user
      @current_user ||= User.find(session[:user_id]) if session[:user_id]
end
end
```
To make sure that users may only edit or delete a resource that belongs to them,  `.find_by` is used to find the object by it’s id and set it to an instance variable. If the object’s user matches the `current_user` object, the edit form is shown. If not, the user is redirected to the items index.
```
@item = Item.find_by_id(params[:id])
    if @item && @item.user == current_user
        erb :'items/edit'
    else
        redirect '/items'
    end
```

I ran into an error when my app attempted to load a resource that was deleted, so I used an if statement to redirect the user if the object does not exist. After creating the if statement, my app still broke. Initially, to find the resource, I used `@item = Item.find(params[:id])`. However, I got a different error after implementing the if statement. Error: ActiveRecord::RecordNotFound. I needed a method that would not break the app when calling `if @item` if the item is not in the database. After some research, I found that using `.find_by` will return nil if no record is found. After implementing `.find_by`, my app was able to redirect a user when trying to retrieve a deleted/non-existent resource. 

### Final Thoughts

I learned a lot from making my first application that persists and queries data, requires a valid session, and protects views. There’s definitely more functionality I’d like to add to my app. I hope to improve and expand my application as I continue to learn more. 

When creating an application with lots of moving parts, it’s important to know what each method returns and build a habit of checking the functionality of the app after each new addition. That way, you’ll know exactly what’s causing the error when your code breaks.

If you have any suggestions for improvement or if there are any errors, please feel free to let me know! What was the biggest takeaway after making your first application?
