---
title: "Ruby Methods: #tap and #then"
date: 2022-08-03T00:48:03+06:00
draft: true
---

Recently, I learned about these two methods in Ruby and love using them. In this post, we will see how to use them with examples.

## tap

`tap` method helps to perform certain operations while still returning the old object.

```ruby
['hello', 'world].join(' ').tap { |message| message.upcase }

# "hello world"
```
As we can see, tap doesn't return its result. Instead the object it was called on was returned. This is useful for logging.

```ruby
def delete(user)
  user
    .update(deactivated: true)
    .tap { |user| Rails.logger.info "User with id##{user.id} is deactivated" }
end
```

## then

`then` passes the object to the block and return the result from the block

```ruby
['hello', 'world].join(' ').then { |message| message.upcase }

# "HELLO WORLD"
```

`then` returns the result form the block.

## `#tap` helps to perform Actions

tap helps to perform actions on the object and remove intermediate variables.

### without tap

```ruby
def deliver_order(order)
  updated_order = order.update(status: :delivered)
  notify_user(updated_order)
  update_inventory(updated_order)

  updated_order
end
```

### with tap

```ruby
def deliver_order(order)
  order
    .update(status: :delivered)
    .tap { |order| notify_user(order) }
    .tap { |order| update_inventory(order) }
end

order = deliver_order(order)
order.status_delivered? # true
```
Without using the tap method we would need to user intermediate variables and return the updated order at the end.

## `#then` helps in Transformation

Since then returns the result from the block, we can use it to do subsequent operations on the object.

```ruby
def result
  Product
    .then(&method(:filter_by_category))
    .then(&method(:filter_by_brand))
    .then(&method(:filter_by_price_range))
end

private

def filter_by_category(products)
  return products unless @category

  products.where(category: @category)
end

def filter_by_brand(products)
  return products unless @brand

  products.where(brand: @brand)
end

def filter_by_price_range(products)
  return products if @max_price.blank? && @min_price.blank?

  products.where("price >= ? && price <= ?", @min_price, @max_price)
end
```