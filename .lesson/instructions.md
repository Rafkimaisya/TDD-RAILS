# TDD for Rails Model

In the previous session, we have learned how to write unit tests and do test-driven development with Ruby. In this session, we are going to learn about test-driven development for Rails. As this track is focused on backend only, we are only going to learn TDD for Rails Models and Controllers.

## Setup

Before we begin, we need to setup a few things. First, once you create a new Rails project in Replit, immediately execute `git commit -m "Initial commit"` to save the current state.

### Gems

Add the following to `Gemfile`:

```
group :development, :test do
  gem "rspec-rails"
  gem "factory_bot_rails"
  gem "faker"
end
```

amd then execute `bundle install`. Like before, commit this change with the following command: `git commit -m "Add rspec-rails, factory_bot_rails, and faker to Gemfile"`. From now on, we will do much more granular comits in our project.

### RSpec

Next, we need to generate RSpec in our Rails. For this, execute `rails generate rspec:install`. The result should look similar to this:

```
create  .rspec
create  spec
create  spec/spec_helper.rb
create  spec/rails_helper.rb
```

As usual, don't forget to commit, `git commit -m "Generate rspec:install"`.

### Factory Bot

Create folder `spec/support`, add file `spec/support/factory_bot.rb`, fill with:

```ruby
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
```

Edit `spec/rails_helper.rb`, add the following line below the line that contains comment `# Add additional requires below this line. Rails is not loaded until this point!`:

```ruby
require_relative 'support/factory_bot'
```

Execute `git commit -m "Add factory_bot config to rspec"`.

### Generate Our First Model

Generate our Food model with name, description, and price with command `rails generate model Food name:string description:text price:float`. The result should look like this:

```
invoke  active_record
create    db/migrate/20220407015925_create_foods.rb
create    app/models/food.rb
invoke    rspec
create      spec/models/food_spec.rb
invoke      factory_bot
create        spec/factories/foods.rb
```

Execute `rails db:migrate`, the result should look like:

```
== 20220407015925 CreateFoods: migrating ======================================
-- create_table(:foods)
   -> 0.0034s
== 20220407015925 CreateFoods: migrated (0.0035s) =============================
```

To confirm that our setup works as expected, run `rspec -fd`. You should see a message similar to:

```
Food
  add some examples to (or delete) /food-delivery/spec/models/food_spec.rb (PENDING: Not yet implemented)

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) Food add some examples to (or delete) /home/iqbal/Workspace/Office/generasi-gigih/food-delivery/spec/models/food_spec.rb
     # Not yet implemented
     # ./spec/models/food_spec.rb:4


Finished in 0.00149 seconds (files took 2.31 seconds to load)
1 example, 0 failures, 1 pending
```

Again, don't forget to commit, `git commit -m "Generate model for Food"`.

## Validation Spec

After the steps above, you might be wondering, what behaviors do we specify in our model? It seems like all the behaviors of our `Food` class is defined by Rails `ActiveRecord` already. Do we need to write specs for behaviors like `Food.all` or `Food.where(name: 'Nasi Uduk')`? What do you think?

In our opinion, we should only write specs for behaviors that we have control over. Since we don't really write the implementation of `Food.all`, for instance, we don't need to write specs for those behaviors. However, as a model is the place where we put the business logic of our application, we need to specify those behaviors. We'll start with validations.

### Validating Presence

One business logic that we can have is to ensure that only data that satisfies certain conditions that are saved to our system. Here, we can validate the presence of certain attributes before we store a data to our database. In this example, we validate the presence of `name` and `description` for our Food model.

Add the following line in `spec/models/food_spec.rb`:

```ruby
require 'rails_helper'

RSpec.describe Food, type: :model do
  it 'is valid with a name and a description' do
    food = Food.new(
      name: 'Nasi Uduk',
      description: 'Betawi style steamed rice cooked in coconut milk. Delicious!',
      price: 15000.0
    )

    expect(food).to be_valid
  end
end
```

Execute `rspec -fd`:

```
Food
  is valid with a name and a description

Finished in 0.02506 seconds (files took 1.14 seconds to load)
1 example, 0 failures
```

You might be wondering, "hey, I have not written any additional code, but the spec is green. Should I just skip this spec?" While it is a valid question, we deem it's necessary to keep this spec as it is a contract of our model: all Food should have a name and a description. While this does not follow the red-green-refactor pattern, we deem it's necessary to keep this spec. As usual, don't forget to commit our progress `git commit -m "Add customary spec for Food model"`.

Next, let's update our `food_spec` to ensure that it's invalid without a name:

```ruby
require 'rails_helper'

RSpec.describe Food, type: :model do
  it 'is valid with a name and a description' do
    food = Food.new(
      name: 'Nasi Uduk',
      description: 'Betawi style steamed rice cooked in coconut milk. Delicious!',
      price: 15000.0
    )

    expect(food).to be_valid
  end

  it 'is invalid without a name' do
    food = Food.new(
      name: nil,
      description: 'Betawi style steamed rice cooked in coconut milk. Delicious!',
      price: 15000.0
    )

    food.valid?

    expect(food.errors[:name]).to include("can't be blank")
  end
end
```

Execute `rspec -fd`:

```
Food
  is valid with a name and a description
  is invalid without a name (FAILED - 1)

Failures:

  1) Food is invalid without a name
     Failure/Error: expect(food.errors[:name]).to include("can't be blank")
       expected [] to include "can't be blank"
     # ./spec/models/food_spec.rb:23:in `block (2 levels) in <top (required)>'

Finished in 0.04116 seconds (files took 1.29 seconds to load)
2 examples, 1 failure

Failed examples:

rspec ./spec/models/food_spec.rb:14 # Food is invalid without a name
```

To make the spec green, update `app/models/food.rb`:

```ruby
class Food < ApplicationRecord
  validates :name, presence: true
end
```

Execute `rspec -fd`:

```
Food
  is valid with a name and a description
  is invalid without a name

Finished in 0.1432 seconds (files took 1.66 seconds to load)
2 examples, 0 failures
```

Now that the spec is green again, let's commit our progress, `git commit -m "Add validation for Food, invalid without name"`. Write a spec to validate the presence of `description` on your own!

### Validating Uniqueness

We can have a lot of [other kinds of validation](https://guides.rubyonrails.org/active_record_validations.html). Below, we want to validate the uniqueness of a certain attribute, to be exact, we want to ensure no two rows in our Food table have the same name:

```ruby
RSpec.describe Food, type: :model do
  # some lines of code is not shown

  it "is invalid with a duplicate name" do
    food1 = Food.create(
      name: "Nasi Uduk",
      description: "Betawi style steamed rice cooked in coconut milk. Delicious!",
      price: 10000.0
    )
    
    food2 = Food.new(
      name: "Nasi Uduk",
      description: "Just with a different description.",
      price: 10000.0
    )

    food2.valid?
    
    expect(food2.errors[:name]).to include("has already been taken")
  end
end
```

Don't forget to execute `rspec -fd` again to ensure this goes red.

```
Food
  is valid with a name and a description
  is invalid without a name
  is invalid with a duplicate name (FAILED - 1)

Failures:

  1) Food is invalid with a duplicate name
     Failure/Error: expect(food2.errors[:name]).to include("has already been taken")
       expected [] to include "has already been taken"
     # ./spec/models/food_spec.rb:41:in `block (2 levels) in <top (required)>'

Finished in 0.1475 seconds (files took 1.19 seconds to load)
3 examples, 1 failure

Failed examples:

rspec ./spec/models/food_spec.rb:26 # Food is invalid with a duplicate name
```

Write code to make it pass:

```ruby
class Food < ApplicationRecord
  validates :name, presence: true, uniqueness: true
end
```

Now it should go green:

```
Food
  is valid with a name and a description
  is invalid without a name
  is invalid with a duplicate name

Finished in 0.19022 seconds (files took 1.34 seconds to load)
3 examples, 0 failures
```

As usual, don't forget to commit `git commit -m "Add validation for Food, invalid with duplicate name"`.

In real projects, what kind of validations we should write for our models depend on the business logic of the app. This is why as a software engineer, you should be able to work and communicate with the business people to understand the problems they're trying to solve and the business domain in which you are writing your solution. Don't be a feature factory!

## Class Methods

In addition to validations, business logic also can be implemented with class methods. For instance, suppose you are going to run a promo which only involves all foods that contain the 'N' letter in its name, we can do the following:

```ruby
require 'rails_helper'

RSpec.describe Food, type: :model do
  # -cut-
  
  describe 'self#by_letter' do
    it "should return a sorted array of results that match" do
      food1 = Food.create(
        name: "Nasi Uduk",
        description: "Betawi style steamed rice cooked in coconut milk. Delicious!",
        price: 10000.0
      )

      food2 = Food.create(
        name: "Kerak Telor",
        description: "Betawi traditional spicy omelette made from glutinous rice cooked with egg and served with serundeng.",
        price: 8000.0
      )

      food3 = Food.create(
        name: "Nasi Semur Jengkol",
        description: "Based on dongfruit, this menu promises a unique and delicious taste with a small hint of bitterness.",
        price: 8000.0
      )

      expect(Food.by_letter("N")).to eq([food3, food1])
    end
  end
end
```

Red:

```
Food
  is valid with a name and a description
  is invalid without a name
  is invalid with a duplicate name
  self#by_letter
    should return a sorted array of results that match (FAILED - 1)

Failures:

  1) Food self#by_letter returns a sorted array of results that match
     Failure/Error: expect(Food.by_letter("N")).to eq([food3, food1])

     NoMethodError:
       undefined method `by_letter' for Food:Class
     # ./spec/models/food_spec.rb:64:in `block (3 levels) in <top (required)>'

Finished in 0.16871 seconds (files took 1.22 seconds to load)
4 examples, 1 failure

Failed examples:

rspec ./spec/models/food_spec.rb:45 # Food self#by_letter returns a sorted array of results that match
```

Make it pass:

```ruby
class Food < ApplicationRecord
  validates :name, presence: true, uniqueness: true

  def self.by_letter(letter)
    where("name LIKE ?", "#{letter}%").order(:name)
  end
end
```

Result:

```
Food
  is valid with a name and a description
  is invalid without a name
  is invalid with a duplicate name
  self#by_letter
    should return a sorted array of results that match

Finished in 0.19866 seconds (files took 1.48 seconds to load)
4 examples, 0 failures
```

# Homework

For your homework, write specs and necessary code to make the following scenario pass:
- Food model does not accept non numeric values for "price" field
- Food model does not accept "price" less than 0.01

Bonus point:
- Category model with "name" as its only field, write the necessary specs
- Change Food model so it now has to have one category
