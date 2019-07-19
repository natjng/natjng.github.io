---
layout: post
title:      "Basic Ruby Iterators and When to Use Them"
date:       2019-07-19 18:43:32 -0400
permalink:  basic_ruby_iterators_and_when_to_use_them
---


Iterators are a useful way to operate on a collection of objects. 

There are an abundance of Ruby enumerables available, but which one should you use? Well, that depends on the return value you need. Different iterators have different return values. Let’s go through some of the most common iterators and compare their return values.

## `#each`
The each method will not modify the collection and will always return the original collection. This is good for putting out information from the collection.
```
pizza_toppings = ["pepperoni", "onions", "sausage", "mushrooms", "bacon"]
pizza_toppings.each do |topping|
    puts "I want #{topping} on my pizza."
end
```

The above will print the following.
```
I want pepperoni on my pizza.
I want onions on my pizza.
I want sausage on my pizza.
I want mushrooms on my pizza.
I want bacon on my pizza.
```
And will return the following.
```
#=> ["pepperoni", "onions", "sausage", "mushrooms", "bacon"]
```
**Return value: original collection**

**Note:** Although `#each` is used in the example below, the collection will be transformed since the shovel operator is used to add “!” to the end of each element.
```
pizza_toppings = ["pepperoni", "onions", "sausage", "mushrooms", "bacon"]
pizza_toppings.each do |topping|
    topping << "!"
end
#=> ["pepperoni!", "onions!", "sausage!", "mushrooms!", "bacon!"]
```
The transformed collection will be returned. When the collection is called again, you can see that it has been modified.
```
pizza_toppings 
#=> ["pepperoni!", "onions!", "sausage!", "mushrooms!", "bacon!"] 
```

## `#map (#collect)`
The map/collect method is a good option when you want to return a modified version of the collection, without changing the original collection.

```
pizza_toppings = ["pepperoni", "onions", "sausage", "mushrooms", "bacon"]
pizza_toppings.map do |topping|
    "I want #{topping} on my pizza."
end
#=> ["I want pepperoni on my pizza.", "I want onions on my pizza.", "I want sausage on my pizza.", "I want mushrooms on my pizza.", "I want bacon on my pizza."] 
```

**Return value: new collection that has been modified based on the block**

**Note:** The original collection is not modified.

When the collection is called, you can see that it has not been changed. If you want to access the return value from map, you must save it to a variable.
```
pizza_toppings
 => ["pepperoni", "onions", "sausage", "mushrooms", "bacon"] 
```

## `#select` 
The select method is used when you need all the elements in the collection that the block evaluates to true.
```
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
numbers.select do |number|
  number.even?
end
#=> [2, 4, 6, 8, 10] 
```

**Return value: array**

**Note:** If no element makes the block evaluate to true, an empty array is returned.

```
numbers = [1, 3, 5, 7, 9]
numbers.select do |number|
  number.even?
end
=> [] 
```

## `#detect (#find)`
Detect/find will return the first element in the collection that evaluates to true.
```
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
numbers.detect do |number|
  number.even?
end
#=> 2 
```

**Return value: single object**

**Note:** If no element makes the block evaluate to true, `nil` is returned.

## `#all?`
The all method can be used with a block or passed a pattern to test whether each element can be subsumed under the pattern. All elements in the collection must evaluate to true for the method to return true. 

Used with a block.
```
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
numbers.all? do |number|
  number.even?
end
#=> false
```
Used with a pattern.
```
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
numbers.all?(Integer)
 => true 
```

**Return value: boolean**

## `#any?`
Similar to `#all?`, the `#any?` method can take a block or pattern. If any element in the collection evaluates to true, the method will return true.
```
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
numbers.any? do |number|
  number.even?
end
#=> true
```
```
data_types = ["string", false, 100]
data_types.any?(Integer)
#=> true
```

**Return value: boolean**

## `#include?`
The include method is passed an object. The method will return true if any element in the collection equals the object. 
```
pizza_toppings = ["pepperoni", "onions", "sausage", "mushrooms", "bacon"]
pizza_toppings.include?("bacon")
#=> true 
```

**Return value: boolean**

### `#any?` vs `#include?`
`#any?` is useful for evaluating the truthiness of the logic of a block.

`#include?` is useful for comparing the actual object or value.

```
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
numbers.any?{ |number| number > 8 }
#=> true 
numbers.include?(100)
#=> false
```

## TL;DR

Use the enumerable based on the return value you need. 

All the enumerables below do not alter the original collection. Please note the special case mentioned above when the original collection is altered.

### Returns the same number of elements (same length) as the original collection.

`#each` use to return the original collection

`#map (#collect)` use to return transformations

### Filters the collection based on block.

`#select` use to return all elements the block evaluates to true, **returns an array**

`#detect (#find)`  use to return the first value the block evaluates to true, **returns a single object**

### Returns a boolean value.

`#all?` use to test if all elements evaluate to true

`#any?` use to test if at least one element evaluates to true

`#include?` use to test if at least one element equals the object passed


You can view the full list of Ruby enumerables [here](https://ruby-doc.org/core-2.6.3/Enumerable.html).

Please let me know if this has been helpful or if there’s an enumerable I should add to this list!
