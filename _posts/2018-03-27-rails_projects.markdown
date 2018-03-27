---
layout: post
title:      "Rails Projects"
date:       2018-03-27 09:28:15 +0000
permalink:  rails_projects
---


Ok, this rails project actually took a lot longer than I expected.
Well, I just made this big to see if I can do the modeling in the way I wanted to do. 

I tactically created very basic order management system. 
Not eveything went as I expected since I probably need javascript later to achieve data display on condition. 

Anyway, I am writing some interesting parts I went through for my project. 

1. Table name for boolean

This has gotton quite interesting.
I needed set up a column to see if the data is active or inactive.
Then the simple question to myself was 'should I add question mark at the end of the field name for the table?'.
I just went ahead creating a table with question mark, of course, it did not work on the table, and with the form data. 

When I checked the convention, preferred way would be avoiding `is_` style for model because Ruby prefers the suffix for "is"-style method names. 
Other interesting point was that I can use question mark with boolean field name when I get a value. Ruby on Rails defines a "predicate" version (one that ends in a ?) of your boolean attribute and returns true/false as expected. As you can see below, `active` field is boolean, and I can use `active?` for clarity as Rails convention


```
FreightTerm id: 3, name: "DDP", company_id: 1, active: false, created_at: "2018-03-25 02:24:02", updated_at: "2018-03-25 02:24:02">

FreightTerm.last.active?

FreightTerm Load (0.2ms)  SELECT  "freight_terms".* FROM "freight_terms" ORDER BY "freight_terms"."id" DESC LIMIT ?  [["LIMIT", 1]]

=> false
```


2. Single Table Inheritance

The other part that I focused on in this project was using one model for multiple cases. 
I created `Account` model for `Supplier` and `Customer` since they have the same structure. 
`Account` is the actual model with database table, and `Supplier` and `Customer` are the subclass of `Account`.

When I created migration for `Account`, I set the field named `type`, which records the if the record is `Supplier` or `Customer` instance. This allowed me to use one model - `Account`, but when I was work with data, I was able to call it with the correct type. 

Below is the model setup from my project - 

```
class Account < ApplicationRecord
  scope :customers, -> { where(type: "Customer") }
  scope :suppliers, -> { where(type: "Supplier") }

  belongs_to :company
  belongs_to :currency
  belongs_to :payment_term
  belongs_to :warehouse
  has_many :account_addresses

  validates :name, presence: true
end
```

```
class Customer < Account
  has_many :sales_orders, foreign_key: "account_id"
end
```

```
class Supplier < Account
  has_many :purchase_orders, foreign_key: "account_id"
end
```

You can notice each `Customer` and `Supplier` have association with `SalesOrder` and `PurchaseOrder`.
In fact, `SalesOrder` and `PurchaseOrder` also share the same model named `AccountOrder` that belongs to `Account`. 
Since I used the Single Table Inheritance on `AccountOrder` model as well - `SalesOrder` and `PurchaseOrder` are subclass of `AccountOrder`, I configured `foreign_key` as `account_id` to map properly. 

By doing this, I only created 2 models - `Account` and `AccountOrder`.
However, I had an effect like I have 4 models - `Customer`, `Suppleir`, `SalesOrder`, `PurcahseOrder`

It was quite tricky to configure at the beginning, but it was worth try to set it up for testing.
Below is the link that I referenced to: 

https://www.driftingruby.com/episodes/single-table-inheritance




