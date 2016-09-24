## Table of Contents

* [Polymorphic associations](#polymorphic-associations)
* [Recursive associations](#recursive-associations) 

## Polymorphic associations

Active Record's `has_many` and `belongs_to` associations work really well when the two sides of the relationship have fixed classes: 

- An `Author`can have many `Books`. 
- A `Library` can have many `Books`, etc. 

**But sometimes you may want to use one table and model to represent something that can be associated with many types of entities.**

For example let's say that I want to register addresses for People & Companies, how should I model that? The first idea that came into my mind is create two different models one for People (`AddressPerson`) and another one for Companies (`AddressCompany`), but that's way to repetitive! This is kind of scenario when Polymorphic associations come in handy.

The name may be daunting (Polymorphic uhhhhhh :ghost:), but there's really nothing to fear. Let's work through the previous example: 


First we'll need to create migrations for Person, Company & Address model: 

```ruby
class CreatePeople < ActiveRecord::Migration[5.0]
  def change
    create_table :people do |t|
      t.string :name
      t.timestamps
    end
  end
end

class CreateCompanies < ActiveRecord::Migration[5.0]
  def change
    create_table :companies do |t|
      t.string :name
      t.timestamps
  end
end

class CreateAddresses < ActiveRecord::Migration[5.0]
  def change
    create_table :addresses do |t|
      t.string :street
      t.references :addressable, polymorphic: true, index: true
      t.timestamps
  end
end
```

I guess you have notice some things unusual about the `CreateAddresses` migration, why `addressable_id` and `addressable_type` show up? What are those? Well in order to declare a polymorphic association we need to departure from the usual ActiveRecord convention: 

- First the name of the foreign key is neither `people_id` nor `company_id`, it's called `addressable_id` instead. 
- Second, we need to add a column called `addressable_type`, you'll see in a moment how we're going to use these columns.

Now let's generate models for Person, Company and Address. 

```ruby
class Person < ActiveRecord::Base
  has_many :addresses, as: :addressable
end

class Company < ActiveRecord::Base
  has_many :addresses, as: :addressable
end
```

As you can see the `has_many` calls in the two models are identical, but what about the `as: :addresable` part? This is the option that makes the new polymorphic association work, it tells ActiveRecord that the current model's roles in this association act as `addressable`, as opposed to say a person has many addresses or a company has many addresses.

Next we need to modify the `Address` model to say that it belongs_to addressable things:

```ruby
class Address
  belongs_to :addressable, polymorphic: true
end
```

If we had omitted the `:polymorphic` option to `belongs_to`, ActiveRecord would have assumed that `Address` belonged to object of class `Addressable` and would have managed the foreign keys and lookups in the usual way. However, since we've included the `:polymorphic` option, Active Record knows how to perform lookups based on both the foreign key and the type. 

The best way to understand what's going on here is to see it in action, let's load the rails console and give our news model a spin

```ruby
> person = Person.create(name: "Cosme")
> => #<Person id: 1, name: "Cosme", ...>
> address = Address.new(street: "Av. Siempre viva",...)
> address.addressable = person
> address.addressable_id 
> => 1
> address.addresable_type
> => "Person"
> address.save
``` 

Aha! Associating a Person with an Address populates both the  `addressable_id` and `addressable_type` field. Naturally associating a `Company` with an `Address` will have the same effect 

```ruby
> company = Company.create(name: "Planta Nuclear")
> => #<Company id: 1, name: "Planta Nuclear",...>
> address = Address.new(street: "Av. Siempre viva 2",...)
> address.addressable = company
> address.addressable_id
> => 1
> address.addressable_type
> => "Company"
> address.save
```

A call to Company.find(1).addresses will execute the following SQL:


```sql
SELECT *FROM addressesWHERE (addresses.addressable_id = 1 ANDaddresses.addressable_type = 'Company')
```

## Recursive associations

When using rails relationships you'll sometimes find a model that should have a relation to itself. For example, let's imagine you want to model an Employee - Subordinates relationship, and a Employee - Manager relationship. 
Should we do three different models? `Employee`, `Subordinate` and `Manager`? Well we could, but that'll be hard to maintain. 

Lets think about it for a minute, a `Manager` and a `Subordinate` are also `Employees`right? Why don't we use the same model? Luckily for us, with rails relationships we can store all employees in a single database model, and also be able to trace relationships:


```ruby
class Employee < ApplicationRecord
  has_many :subordinates, class_name: "Employee", foreign_key: "manager_id"

  belongs_to :manager, class_name: "Employee", optional: true
end
```

With this setup we can retrieve subordinates from an employee with `@employee.subordinates` and a employee manager with: `@employee.manager`.

Notice that in order for this to work, we'll need to add a column that references to the model itself when creating the employees table

```ruby
class CreateEmployees < ActiveRecord::Migration[5.0]
  def change
    create_table :employees do |t|
      t.integer :manager
      t.timestamps null: false
    end
  end
end
```
