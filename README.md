# Larry Four - The Laravel 4 Model & Migration Generator

[![Build Status](https://travis-ci.org/XCMer/larry-four-generator.png?branch=master)](https://travis-ci.org/XCMer/larry-four-generator)

**Current Version:** 1.0.0beta2

If you are not already familiar, I had released a generator for Laravel called <a href="https://github.com/XCMer/larry-laravel-generator">Larry</a>. This version is intended to work with Laravel 4, while supporting new features like polymorphic relations.

Larry Four has been re-written from scratch for Laravel 4. We have a better syntax that allows more flexibility and room for adding in more features.

## A bird's eye view

Here's how an input to larry would look:

    User users; hm Post; mo Image imageable; btm Role;
        id increments
        timestamps
        username string 50; default "hello world"; nullable;
        password string 64
        email string 250
        type enum admin, moderator, user

    Post; mm Image imageable;
        timestamps
        title string 250
        content text
        rating decimal 5 2

    Image
        filename string

    Role
        name string
        level integer

In the above case, Larry would do the following:

- Create migration files for all the tables required, along with all the columns. Larry automatically figures out which foreign key columns to add, depending on the relations you define. Pivot table in case of belongsToMany relation is also automatically created.
- Create model files for all the models defined. These models also have relational functions automatically defined in them.


## Installation

Larry is still in beta. You can visit <a href="https://packagist.org/packages/raahul/larryfour">Packagist</a> to check the latest version of Larry Four. Currently, it is `1.0.0beta2`.

Here are the steps:

- Put the following in your composer.json: `"raahul/larryfour": "v1.0.0beta2"`
- Run `composer update`
- Add `'Raahul\LarryFour\LarryFourServiceProvider'` to the `providers` array of `app/config/app.php`

## Usage

Once you've successfully installed Larry Four, its commands should be accessible via `artisan`. You can always type just `php artisan` to see all available commands, and those available under the `larry` namespace.

Larry Four supports three commands.

    php artisan larry:generate <input_file>

The above command takes `<input_file>` as the input, and generates models and migrations based on that. You have to provide a filename that exists at the root of your Laravel 4 installation, wherein artisan itself resides.

You cannot provide absolute paths for the input file yet. If you're providing a relative path instead of a filename, then it is relative to Laravel's root directory (or basepath).

There are two other commands:

    // Generate only migrations, not models
    php artisan larry:migrations <input_file>

    // Generate only models, not migrations
    php artisan larry:models <input_file>

## Syntax reference

Create a new text file at the root of your Laravel installation, and you can name it anything you want. The input file basically defines the following things:

- All the models that should be created
- The relationship between those models
- The fields inside each of those models
- Modifiers to the fields like default values, nullable/unsigned, and indices

All the models get an auto-incrementing primary key called `id`. You can override the name of the primary key. All foreign keys created are unsigned.

Larry provides ways to override foreign key and pivot table names as well. These are optional. By default, Larry follows the same convention as Laravel for naming tables, pivot tables, and foreign keys.

Finally, Larry ignores blank lines (even if they contain whitespace). So, you're free to beautify the looks of your input.

Now, let's begin with the syntax.

### Model definition

Model definition is how you tell Larry about a new model that has to be created. Since fields need a model to be added to, a model definition will be the first logical line of your input file.

When defining a new model, the line **should not** be indented by any amount of whitespace.

The most simple model definition would look like this:

    User

All models will automatically get an `id` field of type `increments`. Apart from just defining the model, you can also define relations between models on this line.

    User users; hm Post; mo Image imageable; btm Role;

In the above case, we specify the relation that the user has with the other models. The types of the relations supported are:

    hm: hasMany
    ho: hasOne
    btm: belongToMany
    mm: morphMany
    mo: morphOne

Notice that you can't specify `belongsTo` and `morphTo` relations. They are added automatically to the concerned model when their inverses, `hasMany, hasOne, morphMany, morphOne`, are specified in a related model.

#### Semicolons

When defining related models, each definition is delimited by a semicolon. The final semicolon is optional.


#### Overriding table name

You can override the table name of the current model by simply adding it after the model name.

    User my_users; hm Post

The generated model and migration will take this into account.


#### Overriding foreign key and pivot table names

While specifying relations above, you can override the foreign key used. This can be done as:

    User; hm Post my_user_id;

In the above case, the foreign key `my_user_id` will be used instead of the conventional `user_id`. Larry takes care of the naming in the migrations, as well as overriding the default foreign key in the model's relation function.

In case of `belongsToMany`, you can override the pivot table name:

    // pivot table will be named "r_u" instead of the
    // conventional role_user
    User; btm Role r_u;

And also the foreign keys inside the pivot table:

    // foreign keys are named "u_id" & "r_id" instead of
    // the conventional "user_id" & "role_id"
    User; btm Role r_u u_id r_id;

For polymorphic relations (`morphOne` and `morphMany`), it is mandatory to specify a second paramater to the relation, indicating the name of the polymorphic function:

    User; mm Image imageable;


### Field definition

After you define a model, you need to define fields for it.

    User users; hm Post; mo Image imageable;
        id increments
        timestamps
        username string 50; default "hello world"; nullable;
        password string 64
        email string 250
        type enum admin, moderator, user

Looking above, you'll get a good idea of how fields are defined. The syntax is as follows:

    <field_name> <field_type> <field_parameters>; <field_modifier>; <field_modifier> ...

- The `<field_name>` is simply the column name.
- The `<field_type>` is any of the field types supported by Laravel.
- `<field_parameters>` are additional parameters to a field function, like length of a string.
- `<field_modifier>` includes default, nullable, unsigned , and indices. Multiple field modifiers can be specified for a field.

**Below are certain points to note:**

The `increments` field is optional, and you should have a need to specify it only if you want your auto-incrementing field to be named differently from `id`.

The `timestamps` field is special, for it has no field name. By default, timestamps are disabled in all the models, and migrations don't create columns for them. By adding `timestamps` as a field, you enable it for that model, and the migration will contain the necessary columns.

Another field that has a different syntactical nuance is the `enum` field. The parameters to the enum fields are separated by commas. They may or may not be individually enclosed in quotes, like:

    type enum "admin", "moderator", "user"
    OR
    type enum admin, moderator, user

Other types work as expected, and have syntax similar to the `string` type you see above.

The following field types are supported:

    increments
    string
    integer
    bigInteger
    smallInteger
    float
    decimal
    boolean
    date
    dateTime
    time
    timestamp
    text
    binary
    enum

And the following field modifiers are supported:

    default "value"
    nullable
    unsigned
    primary
    fulltext
    unique
    index


## Error handling

Larry Four has an improved error handling mechanism. If there are syntax errors in your input file, you will be notified about it along with the line number. The error will tell you exactly what's wrong in plain English.

Currently, Larry can detect the following errors:

- Typo in relationship types (typing `hms` instead of `hm` will yield an error)
- Insufficient parameters to relationships or field definitions
- Non-existance of a model that was specified as related in another model
- Invalid field types

## Testing

The repo contains PHPUnit tests. This should keep the bugs out and also expedite feature releases.