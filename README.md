# ActiveRecord Validations

ActiveRecord can validate our models for us before they even touch the database. This means it's harder to end up with bad data, which can cause problems later even if our code is technically bug-free.

We can use `ActiveRecord::Base` helper methods like `#validates` to set
things up.


## Objectives

After this lesson, you should be able to:

- Identify when validation occurs in the lifespan of an object
- Introspect on the `ActiveRecord::Errors` collection object on an AR instance
  - use `#valid?`
  - use `#errors`
- Generate `full_message`s for errors
- Check an attribute for validation errors
- Add a custom validation error to an AR model


## Context: Databases and Data Validity

What is a "validation"?

In the context of Rails, validations are special method calls that go at the top of model class definitions and prevent them from being saved to the database if their data doesn't look right.

In general, "validations" are any code that perform the job of protecting the database from invalid data.

### AR Validations Are Not Database Constraints (Validations)

Many relational databases, such as SQLite and PostgreSQL, have data validation features that check things like length and data type. These validations are typically added via migrations, and depending on the specific validation, they may or may not be reflected in the schema.rb file.

Database constraints and model validations are also functionally different. Database constraints will ALWAYS be checked when adding or updating data in the database, while AR validations will only be checked when adding or updating data through Ruby/Rails (e.g. if we use SQL code in the command line to modify the database, AR validations are not run).

Some developers use database constraints and AR validations, while others rely on AR validations alone. Ultimately, it depends on how the developer plans to add and update data in the database. In this lesson, we'll be focusing on AR validations.

### What is "invalid data"?

Suppose you get a new phone and you ask all of your friends for their phone number again. One of them tells you, "555-868-902". If you're paying attention, you'll probably wrinkle your nose and think, "Wait a minute. That doesn't sound like a real phone number."

"555-868-902" is an example of **[invalid data](https://en.wikipedia.org/wiki/Fictitious_telephone_number)**... for a phone number. It's probably a valid account number for some internet service provider in Alaska, but there's no way to figure out what your friend's phone number is from those nine numbers. It's a showstopper, and even worse, it kind of looks like valid data if you're not looking closely.


### Validations Protect the Database

Invalid data is the bogeyman of web applications: it hides in your database until the worst possible moment, then jumps out and ruins everything by causing confusing errors.

Imagine the phone number above being saved to the database in an application that makes automatic calls using the Twilio API. When your system tries to call this number, there will be an error because no such phone number exists, which means you need to have an entire branch of code dedicated to handling *just* that edge case.

It would be much easier if you never have bad data in the first place, so you can focus on handling edge cases that are truly unpredictable.

That's where validations come in.


## Basic Usage

For more examples of basic validation usage, see the Rails Guide for [Active Record Validations][ar_validations]. Take a few minutes to browse the helpers listed in Section 2.

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true
end

Person.create(name: "John Doe").valid? # => true
Person.create(name: nil).valid? # => false
```

[ar_validations]: http://guides.rubyonrails.org/active_record_validations.html

`#validates` is our Swiss Army knife for validations. It takes two arguments: the first is the name of the attribute we want to validate, and the second is a hash of options that will include the details of how to validate it.

In this example, the options hash is `{ presence: true }`, which implements the most basic form of validation, preventing the object from being saved if its `name` attribute is empty.

## Lifecycle Timing

Before proceeding, keep the answer to this question in mind:

**What is the difference between `#new` and `#create`?**

If you've forgotten, `#new` instantiates a new ActiveRecord model *without* saving it to the database, whereas `#create` immediately attempts to save it, as if you had called `#new` and then `#save`.

**Database activity triggers validation**. An ActiveRecord model instantiated with `#new` will not be validated, because no attempt to write to the database was made. Validations won't run unless you call a method that actually hits the DB, like `#save`.

The only way to trigger validation without touching the database is to call the `#valid?` method.

For a full list of methods that trigger validation, see [Section 4][ar_callbacks_4] of the Rails Guide for Active Record Callbacks. Don't worry about the rest of the information in that guide just yet; we'll go into callbacks later!

[ar_callbacks_4]: http://guides.rubyonrails.org/active_record_callbacks.html#running-callbacks

## Validation Failure

Here it is, the moment of truth. What can we do when a record fails validation?

### How can you tell when a record fails validation?

**Pay attention to return values!**

By default, ActiveRecord does not raise an exception when validation fails.  DB operation methods (such as `#save`) will simply return `false` and decline to update the database.

Every database method has a sister method with a `!` at the end which will raise an exception (`#create!` instead of `#create` and so on).

And of course, you can always check manually with `#valid?`.

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true
end

person = Person.new
person.valid? #=> false
person.save #=> false
person.save! #=> EXCEPTION
```

### Finding out why validations failed

To find out what went wrong, you can look at the model's `#errors` object.

You can check all errors at once by examining `errors.messages`.

```ruby
person = Person.new
person.errors.messages #=> empty
person.valid? #=> false
person.errors.messages #=> name: can't be blank
```

You can check one attribute at a time by passing the name to `errors` as a key,
like so:

```ruby
person.errors[:name]
```


## Displaying Validation Errors in Views

See [Section 8][ar_validations_8] of the Rails Guide for an example of how to use the `ActiveModel::Errors#full_messages` helper, reproduced here for convenience:

[ar_validations_8]: http://guides.rubyonrails.org/active_record_validations.html#displaying-validation-errors-in-views

```erb
<% if @article.errors.any? %>
  <div id="error_explanation">
    <h2>
      <%= pluralize(@article.errors.count, "error") %>
      prohibited this article from being saved:
    </h2>

    <ul>
    <% @article.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```

This constructs more complete-looking sentences from the more terse messages available in `errors.messages`.


## Other Built-in Validators

Rails has a host of built-in helpers.

### Length

`length` is one of the most versatile:

```ruby
class Person < ActiveRecord::Base
  validates :name, length: { minimum: 2 }
  validates :bio, length: { maximum: 500 }
  validates :password, length: { in: 6..20 }
  validates :registration_number, length: { is: 6 }
end
```

The `in` argument makes use of a [Range][ruby-core-range].

[ruby-core-range]: http://ruby-doc.org/core/Range.html

Remember that there's no syntactical magic happening with any of these method calls. If we weren't using Ruby's "poetry mode" (which is considered standard for Rails), the above code would look like this:

```ruby
class Person < ActiveRecord::Base
  validates(:name, { :length => { :minimum => 2 } })
  validates(:bio, { :length => { :maximum => 500 } })
  validates(:password, { :length => { :in => 6..20 } })
  validates(:registration_number, { :length => { :is => 6 } })
end
```

Phew!

### Uniqueness

Another common built-in validator is `uniqueness`:

```ruby
class Account < ActiveRecord::Base
  validates :email, uniqueness: true
end
```

This will prevent any account from being created with the same email as another already-existing account.

### Custom Messages

This isn't a validator in its own right, but a handy convenience option for specifying your own error messages:

```ruby
class Person < ActiveRecord::Base
  validates :not_a_robot, acceptance: true, message: "Humans only!"
end
```

### Custom Validators

There are three ways to implement custom validators, with examples in [Section
6][ar_validators_6] of the Rails Guide.

Of the three, `#validate` is the simplest. If your validation needs become more
complex, consult the documentation. For _most_ validations, though, the
following method should be good enough.

1. Create a new directory in `app` called `validators`. Because most Rails
   developers don't need to write custom validation, this directory is **not**
   created by default like `models` or `controllers`.
2. Identify the ActiveRecord attribute you want to validate. Is it the `email`
   or the `last_name` on the `Person` class, for example?
3. Create a new file in the `app/validators` directory of the form attribute
   (from the previous step) + `_validator.rb`. So in the case of validating an
   attribute called `email`, create a file `app/validators/email_validator.rb`
4. Inside the new file, define the class. The class name should match the file
   name of the file, but "Camel-Cased." So `email_validator` should be class
  `EmailValidator`. The class should inherit from `ActiveModel::Validator`
5. The validator class must have one instance method, `#validate`. This method
   will receive one argument typically called `record`.
6. Inside of `#validate`, you'll be able to get properties from `record` and
   determine ***whether it is invalid***. If the record is **invalid**, push (`<<`)
   to `record.errors[:attribute]` e.g. `record.errors[:email]` a `String` which
   is a message that you want to display that describes why the message is not
   valid.
7. Lastly, in the implementation of the class being validated e.g. `Person`,
   add:
   1. An `include` of ActiveModel::Validations
   2. The helper call: `validates_with (className)`. In our example we'd put, `validates_with EmailValidator` (see step 4, above)

The result of these steps should be the following:

```ruby
class EmailValidator < ActiveModel::Validator
  def validate(record)
    unless record.email.match?(/flatironschool.com/)
      record.errors[:email] << "We're only allowed to have people who work for the company in the database!"
    end
  end
end
```

```ruby
class Person
  include ActiveModel::Validations
  validates_with EmailValidator
end
```

Here we validate that all email addresses are in the `flatironschool.com`
domain.

[ar_validators_6]: http://guides.rubyonrails.org/active_record_validations.html#performing-custom-validations
