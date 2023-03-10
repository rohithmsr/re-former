This is part of the Forms Project in The Odin Project’s Ruby on Rails Curriculum. Find it at http://www.theodinproject.com

#### Set up the Back End

You'll get good at setting up apps quickly in the coming lessons by using more or less this same series of steps (though we'll help you less and less each time):

1. Build a new rails app (called "re-former").

```
$ rails new re-former
```

2. Create a new Github repo and connect the remote to your local git repo. Check in and commit the initial stuff.
3. Modify your README file to say something you'll remember later, like "This is part of the Forms Project in The Odin Project's Ruby on Rails Curriculum.  Find it at [http://www.theodinproject.com](http://www.theodinproject.com)"
4. Create and migrate a User model with `:username`, `:email` and `:password`.
```
$ rails g model User username:string email:string password:string

$ rails db:migrate
```
5. Add validations for presence to each field in the model.

    ~~~ruby
    #/app/models/user.rb
    class User < ApplicationRecord
        validates :username, presence: true, uniqueness: true, length: { maximum: 25 }
        validates :email, presence: true, uniqueness: true
        validates :password, presence: true, length: { minimum: 6 } 
    end
    ~~~

6. Create the `:users` resource in your routes file so requests actually have somewhere to go.  Use the `only:` option to specify just the `:new` and `:create` actions.

    ~~~ruby
    #/app/config.routes.rb
    Rails.application.routes.draw do
    resources :users, only: [:new, :create]
    end
    ~~~

7. Build a new UsersController (either manually or via the `$ rails generate controller Users` generator).
```
$ rails g controller Users
```
8. Write empty methods for `#new` and `#create` in your UsersController.
    ~~~ruby
    class UsersController < ApplicationController
        def new
        end

        def create
        end
    end
    ~~~
9. Create your `#new` view in `app/views/users/new.html.erb`.
    ~~~ruby
    #/app/views/users/new.html.erb
    <h1>New User</h1>
    ~~~
10. Fire up a rails server in another tab.
```
$ rails s
```
11. Make sure everything works by visiting `http://localhost:3000/users/new` in the browser.
```
TODO: Add the image ss here!
```
#### HTML Form

The first form you build will be mostly HTML (remember that stuff at all?).  Build it in your New view at `app/views/users/new.html.erb`.  The goal is to build a form that is almost identical to what you'd get by using a Rails helper so you can see how it's done behind the scenes.

1. Build a form for creating a new user.  See the [w3 docs for forms](http://www.w3schools.com/tags/tag_form.asp) if you've totally forgotten how they work.  Specify the `method` and the `action` attributes in your `<form>` tag (use `$ rails routes` to see which HTTP method and path are being expected based on the resource you created).  Include the attribute `accept-charset="UTF-8"` as well, which Rails naturally adds to its forms to specify Unicode character encoding.
```
$ rails routes
```
```
TODO: Add routes pic
```
```
#/app/views/users/new.html.erb
<form method="POST" action="/users" accept-charset="UTF-8">

</form>
```

2. Create the proper input tags for your user's fields (email, username and password).  Use the proper password input for "password".  Be sure to specify the `name` attribute for these inputs.  Make label tags which correspond to each field.

    ~~~ruby
    #/app/views/users/new.html.erb
    <form method="POST" action="/users" accept-charset="UTF-8">
        <label for="username">Username:</label>
        <input type="text" name="username" id="username" /><br />
        <label for="email">Email:</label>
        <input type="email" name="email" id="email" /><br />
        <label for="password">Password:</label>
        <input type="password" name="password" id="password" /><br />
        <input type="submit" value="Create new user">
    </form>
    ~~~
3. Submit your form and view the server output. You will see nothing happening, no error message, nothing. If you look at the network tab in your inspector or at your server log, you can see that a request was issued, but a response of `204 No Content` is returned.
```
TODO: Add 204 ss
```
4. That's A-OK because it means that we've successfully gotten through our blank `#create` action in the controller (and didn't specify what should happen next).  Look at the server output.  It should include the parameters that were submitted, looking something like:

   ~~~bash
   Started POST "/users" for 127.0.0.1 at 2013-12-12 13:04:19 -0800
   Processing by UsersController#create as TURBO_STREAM
   Parameters: {"authenticity_token"=>"WUaJBOpLhFo3Mt2vlEmPQ93zMv53sDk6WFzZ2YJJQ0M=", "username"=>"foobar", "email"=>"foo@bar.com", "password"=>"[FILTERED]"}
   ~~~
That looks a whole lot like what you normally see when Rails does it, right?
5. Go into your UsersController and build out the `#create` action to take those parameters and create a new User from them.  If you successfully save the user, you should redirect back to the New User form (which will be blank) and if you don't, it should render the `:new` form again (but it will still have the existing information entered in it).  You should be able to use something like:

   ~~~ruby
   # app/controllers/users_controller.rb
   def create
     @user = User.new(username: params[:username], email: params[:email], password: params[:password])
     if @user.save
       redirect_to new_user_path
     else
       render :new, status: :unprocessable_entity
     end
   end
   ~~~

~~~ruby
    class UsersController < ApplicationController
        def new
            @user = User.new
        end

        def create
            @user = User.new(username: params[:username], email: params[:email], password: params[:password])

            if @user.save
                redirect_to new_user_path
            else
                render :new, status: :unprocessable_entity
            end
        end
    end
~~~

6. Test this out -- can you now create users with your form? If so, you should see an INSERT SQL command in the server log.
```
TODO: Add INSERT cmd image
```
7. We're not done just yet... that looks too long and difficult to build a user with all those `params` calls.  It'd be a whole lot easier if we could just use a hash of the user's attributes so we could just say something like `User.new(user_params)`.  Let's build it... we need our form to submit a hash of attributes that will be used to create a user, just like we would with Rails' `form_with` method.  Remember, that method submits a top level `user` field which actually points to a hash of values.  This is simple to achieve, though -- just change the `name` attribute slightly.  Nest your three User fields inside the variable attribute using brackets in their names, e.g. `name="user[email]"`.

~~~ruby
    #/app/views/users/new.html.erb
    <form method="POST" action="/users" accept-charset="UTF-8">
        <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

        <label for="username">Username:</label>
        <input type="text" name="user[username]" id="user[username]" /><br />
        <label for="email">Email:</label>
        <input type="email" name="user[email]" id="user[email]" /><br />
        <label for="password">Password:</label>
        <input type="password" name="user[password]" id="user[password]" /><br />
        <input type="submit" value="Create new user">
    </form>
~~~

8. Resubmit.  Now your user parameters should be nested under the `"user"` key like:

   ~~~bash
   Parameters: {"authenticity_token" => "WUaJBOpLhFo3Mt2vlEmPQ93zMv53sDk6WFzZ2YJJQ0M=", "user" =>{ "username" => "foobar", "email" => "foo@bar.com", "password" => "[FILTERED]" } }
   ~~~

4. You'll get some errors because now your controller will need to change.  But recall that we're no longer allowed to just directly call `params[:user]` because that would return a hash and Rails' security features prevent us from doing that without first validating it.
5. Go into your controller and comment out the line in your `#create` action where you instantiated a `::new` User (we'll use it later).
6. Implement a private method at the bottom called `user_params` which will `permit` and `require` the proper fields (see the [Controllers Lesson](/lessons/ruby-on-rails-controllers) for a refresher).

~~~ruby
    class UsersController < ApplicationController
        ...

        private
            def user_params
                params.require(:user).permit(:username, :email, :password)
            end
    end
~~~

7. Add a new `::new` User line which makes use of that new allow params method.

~~~ruby
    class UsersController < ApplicationController
        def new
            @user = User.new
        end

        def create
            # @user = User.new(username: params[:username], email: params[:email], password: params[:password])
            @user = User.new(user_params)

            if @user.save
                redirect_to new_user_path
            else
                render :new, status: :unprocessable_entity
            end
        end

        private
            def user_params
                params.require(:user).permit(:username, :email, :password)
            end
    end
~~~

5. Submit your form now.  It should work marvelously (once you debug your typos)!
```
🎸
```

#### Railsy Forms with `#form_tag`

Now we'll start morphing our form into a full Rails form using the `#form_tag` and `#*_tag` helpers.  There's actually very little additional help that's going on and you'll find that you're mostly just renaming HTML tags into Rails tags.

1. Comment out your entire HTML form.  It may be helpful to save it for later on if you get stuck.
2. Convert your `<form>` tag to use a `#form_tag` helper and all of your inputs into the proper helper tags via `#*_tag` methods.  The good thing is that you no longer need the authentication token because Rails will insert that for you automatically. `#form_tag` is soft-deprecated as stated in the current Rails Guide. You can find the older documentation [here](https://guides.rubyonrails.org/v5.2/form_helpers.html).
3. See the [Form Tag API Documentation](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html#method-i-form_tag) for a list and usage of all the input methods you can use with `#form_tag`.

~~~erb
    <%= form_tag('/users', method: :post) do %>
        <%= label_tag :username, 'Username:' %>
        <%= text_field_tag :username %><br />
        <%= label_tag :email, 'Email:' %>
        <%= email_field_tag :email %><br />
        <%= label_tag :password, 'Password:' %>
        <%= password_field_tag :password %><br />
        <%= submit_tag "Create new user" %>
    <% end %>
    <!--
    <form method="POST" action="/users" accept-charset="UTF-8">
        <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

        <label for="username">Username:</label>
        <input type="text" name="user[username]" id="user[username]" /><br />
        <label for="email">Email:</label>
        <input type="email" name="user[email]" id="user[email]" /><br />
        <label for="password">Password:</label>
        <input type="password" name="user[password]" id="user[password]" /><br />
        <input type="submit" value="Create new user">
    </form>
    -->
~~~

4. Test out your form.  You'll need to change your `#create` method in the controller to once again accept normal top level User attributes, so uncomment the old `User.new` line and comment out the newer one.

~~~ruby
    class UsersController < ApplicationController
        ...

        def create
            @user = User.new(username: params[:username], email: params[:email], password: params[:password])
            # @user = User.new(user_params)

            ...
        end
        
        ...
    end
~~~
5. You've just finished the first step.
```

```

#### Railsy-er Forms with `#form_with`

`#form_tag` probably didn't feel that useful -- it's about the same amount of work as using `<form>`, though it does take care of the authenticity token stuff for you.  Now we'll convert that into `#form_with`, which will make use of our model objects to build the form.

1. Modify your `#new` action in the controller to instantiate a blank User object and store it in an instance variable called `@user`.
2. Comment out your `#form_tag` form in the `app/views/users/new.html.erb` view (so now you should have TWO commented out form examples).
3. Rebuild the form using `#form_with` and the `@user` from your controller.  You'll need to switch your controller's `#create` method again to accept the nested `:user` hash from `params`.
4. Play with the `#input` method options -- add a default placeholder (like "example@example.com" for the email field), make it generate a different label than the default one (like "Your user name here"), and try starting with a value already populated.  Some of these things you may need to Google for, but check out the [`#form_with` Rails API docs](https://api.rubyonrails.org/v6.1.1/classes/ActionView/Helpers/FormHelper.html#method-i-form_with)
5. Test it out.


~~~erb
    <%= form_with model: @user, method: :post do |form| %>
        <%= form.label :username, "Your username here!" %>
        <%= form.text_field :username, default: "Prepopulated Value", size: 20 %><br />
        <%= form.label :email %>
        <%= form.email_field :email, placeholder: "example@example.com" %><br />
        <%= form.label :password %>
        <%= form.password_field :password %><br />
        <%= form.submit %>
    <% end %>

    <!--
    # DEFAULT FORM_WITH COMPONENTS
    <%= form_with model: @user, method: :post do |form| %>
        <%= form.label :username %>
        <%= form.text_field :username %><br />
        <%= form.label :email %>
        <%= form.email_field :email %><br />
        <%= form.label :password %>
        <%= form.password_field :password %><br />
        <%= form.submit %>
    <% end %>
    -->

    <!--
    <%= form_tag('/users', method: :post) do %>
        <%= label_tag 'username', 'Username:' %>
        <%= text_field_tag 'username' %><br />
        <%= label_tag 'email', 'Email:' %>
        <%= email_field_tag 'email' %><br />
        <%= label_tag 'password', 'Password:' %>
        <%= password_field_tag 'password' %><br />
        <%= submit_tag "Create new user" %>
    <% end %>
    -->

    <!--
    <form method="POST" action="/users" accept-charset="UTF-8">
        <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

        <label for="username">Username:</label>
        <input type="text" name="user[username]" id="user[username]" /><br />
        <label for="email">Email:</label>
        <input type="email" name="user[email]" id="user[email]" /><br />
        <label for="password">Password:</label>
        <input type="password" name="user[password]" id="user[password]" /><br />
        <input type="submit" value="Create new user">
    </form>
    -->
~~~

#### Editing

1. Update your routes and controller to handle editing an existing user.  You'll need your controller to find a user based on the submitted `params` ID.


~~~ruby
    #/app/config.routes.rb
    Rails.application.routes.draw do
    resources :users, only: [:new, :create, :edit, :update]
    end
~~~


~~~ruby
    class UsersController < ApplicationController
        def new
            @user = User.new
        end

        def create
            # @user = User.new(username: params[:username], email: params[:email], password: params[:password])
            @user = User.new(user_params)

            if @user.save
                redirect_to new_user_path
            else
                render :new, status: :unprocessable_entity
            end
        end

        def edit
            @user = User.find(params[:id])
        end

        def update
            @user = User.find(params[:id])

            if @user.update(user_params)
                redirect_to new_user_path(@user)
            else
                render :edit, status: :unprocessable_entity
            end
        end

        private
            def user_params
                params.require(:user).permit(:username, :email, :password)
            end
    end
~~~

2. Create the Edit view at `app/views/users/edit.html.erb` and copy/paste your form from the New view.  Your HTML and `#form_tag` forms (which should still be commented out) will not work -- they will submit the form as a POST request when you need it to be a PATCH (PUT) request (remember your `$ rails routes`?).  It's an easy fix, which you should be able to see if you attempt to edit a user with the `#form_with` form (which is smart enough to know if you're trying to edit a user or creating a new one).

~~~erb
    # I have removed the HTTP method in the form. form_with auto-detects whether it is a POST or PATCH request!

    <%= form_with model: @user do |form| %>
        <ul>
            <% @user.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
            <% end %>
        </ul>
        
        <%= form.label :username, "Your username here!" %>
        <%= form.text_field :username, default: "Prepopulated Value", size: 20 %><br />
        <%= form.label :email %>
        <%= form.email_field :email, placeholder: "example@example.com" %><br />
        <%= form.label :password %>
        <%= form.password_field :password %><br />
        <%= form.submit %>
    <% end %>

    <!--
    # DEFAULT FORM_WITH COMPONENTS
    <%= form_with model: @user, method: :post do |form| %>
        <%= form.label :username %>
        <%= form.text_field :username %><br />
        <%= form.label :email %>
        <%= form.email_field :email %><br />
        <%= form.label :password %>
        <%= form.password_field :password %><br />
        <%= form.submit %>
    <% end %>
    -->

    <!--
    <%= form_tag('/users', method: :post) do %>
        <%= label_tag 'username', 'Username:' %>
        <%= text_field_tag 'username' %><br />
        <%= label_tag 'email', 'Email:' %>
        <%= email_field_tag 'email' %><br />
        <%= label_tag 'password', 'Password:' %>
        <%= password_field_tag 'password' %><br />
        <%= submit_tag "Create new user" %>
    <% end %>
    -->

    <!--
    <form method="POST" action="/users" accept-charset="UTF-8">
        <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

        <label for="username">Username:</label>
        <input type="text" name="user[username]" id="user[username]" /><br />
        <label for="email">Email:</label>
        <input type="email" name="user[email]" id="user[email]" /><br />
        <label for="password">Password:</label>
        <input type="password" name="user[password]" id="user[password]" /><br />
        <input type="submit" value="Create new user">
    </form>
    -->
~~~

3. Do a "view source" on the form generated by `#form_with` in your Edit view, paying particular attention to the hidden fields at the top nested inside the `<div>`.  See it?
4. Modify the top of your form view to display a list of the error messages that are attached to the failed model object when it fails validations. Recall the `#errors` and `#full_messages` methods.

~~~erb
    <%= form_with model: @user, method: :post do |form| %>
        <ul>
            <% @user.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
            <% end %>
        </ul>
        
        <%= form.label :username, "Your username here!" %>
        <%= form.text_field :username, default: "Prepopulated Value", size: 20 %><br />
        <%= form.label :email %>
        <%= form.email_field :email, placeholder: "example@example.com" %><br />
        <%= form.label :password %>
        <%= form.password_field :password %><br />
        <%= form.submit %>
    <% end %>
~~~

5. Save this project to Git and upload to GitHub.
```
🍗🍗🍗🍗🍗🍗🍗🍗 DINNER
```