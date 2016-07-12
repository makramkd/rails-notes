# Rails Notes
## Rails Basics
Rails is a web development framework built using the Ruby programming language.

It follows the Model-View-Controller paradigm, or MVC, which seeks to separate
business logic or domain logic from input and presentation logic associated with
the user interface (UI). Domain logic typically consists of data models for
things like users, articles, products, and the UI is just a web page in the
browser.

Copied verbatim from RailsTutorial.org:

> When interacting with a Rails application, a browser sends a request, which is received by a web server and passed on to a Rails controller, which is in charge of what to do next. In some cases, the controller will immediately render a view, which is a template that gets converted to HTML and sent back to the browser. More commonly for dynamic sites, the controller interacts with a model, which is a Ruby object that represents an element of the site (such as a user) and is in charge of communicating with the database. After invoking the model, the controller then renders the view and returns the complete web page to the browser as HTML.

> Michael Hartl, RailsTutorial.org (Chapter 1)

## Controller Actions and Routes
Controller actions are methods inside the body of a Rails controller class.
A Rails controller class is a class that inherits from `ActionController::Base`. The simplest controller class looks like this:

```ruby
class AppController < ActionController::Base
    protect_from_forgery with: :exception
    
end
```

which is a controller class with no actions at all (since it has no methods
of its own, other than those it inherited from it's base class). Say we 
add a method called `index`:

```ruby
class AppController < ActionController::Base
    protect_from_forgery with: :exception
    
    def index
        render text: "Hello, world!"
    end
end
```

We can then configure the rails routes in `config/routes.rb` to use this action
as the root action of our application:

```ruby
Rails.application.routes.draw do
    root 'app#index'
end
```

You can then check the routes of the application using the `rake routes` command. Since we set the root route to be `'app#index'`, if we run the 
Rails server (via `rails server`) on our local machine, we just need to 
visit `localhost:3000` by default (port 3000 on `localhost`) and we'll see
`Hello, world!` rendered on the page.

## Models
Models are typically represented as relational database tables: one example of a data model is a `user` data model, which can be represented
(in a very simplified manner) by the following table:

<center>
    <table>
      <tr>
        <th colspan="2">user</th>
      </tr>
      <tr>
        <td>_id</td>
        <td>integer</td>
      </tr>
      <tr>
        <td>username</td>
        <td>string</td>
      </tr>
      <tr>
        <td>email</td>
        <td>string</td>
      </tr>
    </table>
</center>

Since `SQL` databases are commonly used (most likely either MySQL, SQLite3, or
PostgreSQL), the `_id` part of the row in the table is generated automatically
by the database management system.

To generate a scaffolded model of `user` above, we can use the Rails
`generate` command, as follows:

```
$ rails generate scaffold User username:string email:string
```

Note that we don't have to specify the `_id` column because that's inserted
by Rails automatically. In order to actually update the database schema 
with this new table definition, we need to do a *database migration*. This is
done by the following Rake command wrapped in a `bundle exec`:

```
$ bundle exec rake db:migrate
```

This updates the databse schema with the new data model.

### Data Validation
Rails gives us the ability to do data validation outside of the database
with ActiveRecord's `validates` method. Say we have a `Tweet` data model that
contains two pieces of data: the tweet content (abbreviated to `content`)
and the ID of the user that tweeted it, we can add a validation rule to 
make the tweet at most 140 characters long like this:

```ruby
class Tweet < ActiveRecord::Base
    validates :content, length: { maximum: 140 }
end
```

### Associations between Data Models
When designing relational database schemas, we used terms such as
"one-to-many relationship" and the like. Rails allows us to make use of such
terminology via *associations* between data models. As an example, if we
have a `User` model and a `Tweet` model, since a user can have many tweets
(i.e it is a one-to-many relationship), we can write the following in the
`User` class:

```ruby
class User < ActiveRecord::Base
    has_many :tweets
end
```

Another example is the following: one tweet has exactly one user as an author,
so we can say that it *belongs to* a user:

```ruby
class Tweet < ActiveRecord::Base
    belongs_to :user
    ...
end
```

## REST
REST is an acronym for REpresentational State Transfer. In the context of
Rails applications, this means that most application components (such as 
data models) are modeled as resources that can be created, read, updated,
and destroyed, also known as CRUD. These operations also correspond to the
four fundamental HTTP requests: POST, GET, PATCH, DELETE.

For Rails application development, this means that we should structure our
application using resources that will get CRUDed. 

## Adding Bootstrap
Adding Bootstrap to Rails involves including the `bootstrap-sass` gem in the
Gemfile and then run `bundle install`:

```ruby
source 'https://rubygems.org'

gem 'rails'
gem 'bootstrap-sass'
...
```

## Migrations
The result of the `rails g model ...` command is a new ruby file that is
called a *migration*. A migration provides a way to alter the structure of 
the database incrementally so that the data model can adapt to changing
requirements. 

In order to apply the migrations to the database you need to run `bundle exec rake db:migrate`. 

Any kind of migration can be done manually using the following `rails` command:

```
rails g migration migration_name
```

You can then fill up the migration Ruby file manually with any changes you 
have to make. A common change is to add an index to a table column. For 
example, if you want to add a unique index to the e-mail column in the `user` table,
you can do the following migration:

```
rails g migration add_index_to_users_email
```

And in the migration file:

```ruby
class AddIndexToUsersEmail < ActiveRecord::Migration
  def change
    add_index :users, :email, unique: true
  end
end
```

## Setting Up Heroku
Heroku uses PostgreSQL as a database rather than 
SQLite or something else so you have to add the following code to the
`Gemfile` in order to get the right database to work in production:

```ruby
group :production do
    gem 'pg'
    gem 'rails_12factor'
end
```
Once this is done, commit it to the local git repository. We then have to run
the following command:

```
$ bundle install --without production
```

once we're in the main project directory. This updates the `Gemfile.lock` file,
which Heroku uses in order to install any gems for production. This also
refrains from installing any production gems on the development machine.

Once this book-keeping is done, we can do the following steps with Heroku:

```shell
$ heroku login
$ heroku keys:add
$ heroku create          # adds heroku remote and creates domain
$ git push heroku master # installs rails app on remote
```
Note that if you did any database migrations on your local machine then you
need to run the migration on Heroku:

```
$ heroku run rake db:migrate
```
