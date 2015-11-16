# ActiveRecord Validations

## Objectives

Students will be able to:

- Apply a validation requirement using validates macro
- Identify when validation occurs in the lifespan of an object
- Introspect on the ActiveRecord::Errors collection object on an AR instance
  - use valid?
  - use errors
- Generate full_messages for errors
- Check an attribute for validation errors
- common validation macros
  - scope on uniqueness
  - validation configuration options
- Add a custom validation error to an AR model


## Notes

Introduce the concept of validation, our models protect our DB and can inforce requirements for the attributes.

The validates macro.

Validates presence :name

When do validations run?

Looking at a model with a validation error

Trying to save invalid models.

Looking at instance.errors

instance.errors[:field] and instance.errros.on? (if that's still a thing)

full_messages

error on a specific field, the error parts (field and message)

common validations, presence, minimum, maximum, etc, message configurations, etc.

adding a custom validation via def validate or our own validation macro

validates :custom_requirement

Add resources to rails guides on validation and validations api and errors api.
