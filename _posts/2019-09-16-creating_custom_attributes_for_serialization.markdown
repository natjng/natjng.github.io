---
layout: post
title:      "Creating Custom Attributes for Serialization"
date:       2019-09-16 20:51:43 -0400
permalink:  creating_custom_attributes_for_serialization
---


In my latest project, I created a Single Page JavaScript Application utilizing a Rails API backend. On the backend, the action must convert the data to JSON when passing information to be rendered. The following example uses the [Fast JSON API](https://github.com/Netflix/fast_jsonapi) serializer for Ruby Objects. 

When using the Fast JSON API serializer, we must specify the attributes to be included. Let’s use the two models below as an example. 

```
class User < ApplicationRecord
    attr_accessor :username
    has_many :games
end

class Game < ApplicationRecord
    attr_accessor :score, :user_id
    belongs_to :user
end
```

In each model’s serializer, simply add the attribute names to the `attributes` method to make them available to the view. Here, I’m including the username attribute from my user model. 

```
class UserSerializer
  include FastJsonapi::ObjectSerializer

  attributes :username
end
```

However, I’d also like to render more information about each user instance. For example, I want to show each user’s high score, but that is not an attribute on my user model. This is where custom attributes come in handy. To create a custom attribute with Fast JSON API, we can do so using Ruby block syntax in the serializer class.

```
class UserSerializer
  include FastJsonapi::ObjectSerializer

  attributes :username

  attributes :high_score do |object|
    "#{object.games.map(&:score).max}"
  end
end
```

To get a user’s high score, I first need all the game instances that belong to the user. After getting all the user’s games, I want to put the game scores in a collection and find the highest value, using `max`. 

By using the above block syntax, that is set to a custom attribute name, I can get the user’s high score in my serialized data for rendering. Now, here’s what the output JSON looks like: 

```
{
  "data": {
    "id": "1",
    "type": "user",
    "attributes": {
      "username": "natalie",
      "high_score": "7"
    },
    ...
}
```

There are also ways to override an object’s property or an attribute’s name when using Fast JSON API. More information on this can be found in the [documentation](https://github.com/Netflix/fast_jsonapi#attributes). 

I hope this has helped or inspired you to create a custom attribute for serialization!
