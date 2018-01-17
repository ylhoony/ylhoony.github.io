---
layout: post
title:      "Sinatra Project"
date:       2018-01-17 06:31:32 +0000
permalink:  sinatra_project
---


For Sinatra project, I created a app to manage the company addresses. 
I wanted to test certain data structure between users and companies.
Below is the ERD that I planned out to set up models. This became more complicated than I expected to handle the association and data. Well, it is all about the master account having multiple addresses and contacts, nothing more than that. But, user should have visibility to the companies created under their account. 


![ERD](https://i.pinimg.com/564x/b5/d1/90/b5d190523c68880cf1b35458ac1648c2.jpg)


Below is the summary of the association:
* User has many companies through User_Companies join table.
* Company has many users through User_Companies join table. Company has many addresses and contacts.
* User_companies from join table belongs to User and Company.
* Country has many companies, addresses, and contacts.
* Address belongs to company and contact.
* Contact belongs to company and has one address.

Based on the association, I created 5 views/models, and 6 controllers including ApplicationController.
User_company from join table was not counted to create individual model.



When working on the migration, I had to pay attention to setting up the boolean.
Default value was set to the column with boolean value in the migration as code below.


```
class CreateUserCompanies < ActiveRecord::Migration[5.1]
  def change
    create_table :user_companies do |t|
      t.integer :user_id
      t.integer :company_id
      t.boolean :is_owner, default: false
    end
  end
end
```

In the form for the field with boolean value, I used 'checkbox' to receive true or false. 
In order to include the value when it was not checked, the hidden value was set above the input checkbox line.
Without the hidden values, form did not include the field in the params when the form was submitted. 

```
  <input type="hidden" name="address[is_billing_address]" value="false">
  <label for="billing_address">Billing Address: </label><input type="checkbox" name="address[is_billing_address]" value="true" id="billing_address"><br>
  <input type="hidden" name="address[is_shipping_address]" value="false">
  <label for="shipping_address">Shipping Address: </label><input type="checkbox" name="address[is_shipping_address]" value="true" id="shipping_address">
	```


Since I created User_Companies join table, I added one more condition to make sure the user can access to the companies that they have access to. 

```
  get '/companies/:id/contacts' do
    if logged_in?
      if current_user.company_ids.include?(params[:id].to_i)
        @company = Company.find(params[:id])
        erb :"company_contacts/index"
      else
        redirect "/"
      end
    else
      redirect "/login"
    end
  end

```


Below is just note. Eventually, I did not need this, but at the beginning, I wanted to check all the data if there is any record that matches to each parameter. It took some time to fugure out converting string to symbol properly, anyhow I was able to check all database with the code below. 

```
    def find_by_params(params) 
      params.find do |key, value|
        Country.find_by("#{key.to_s}": value)  #=> Country.find_by(key.to_sym => value)  this worked same.
      end
    end
```

It was great to practice to set up the entry point, server, and routes with views!






