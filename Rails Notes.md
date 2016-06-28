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

