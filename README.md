# Learning by Doing 

![Ubuntu version](https://img.shields.io/badge/Ubuntu-16.04%20LTS-orange.svg)
![Rails version](https://img.shields.io/badge/Rails-v5.0.0-blue.svg)
![Ruby version](https://img.shields.io/badge/Ruby-v2.3.1p112-red.svg)

Mackenzie Child's video really inspired me. So I decided to follow all of his rails video tutorial to learn how to build a web app. Through the video, I would try to build the web app by my self and record the courses step by step in text to facilitate the review.


# Project 6: How To Build A Todo App With Rails       
This time we build a pretty simple application. It's a Todo-list. We have multiple lists and multiple todo items for each list. And the ability to check an item if the item has been completed or not as well as delete the item. And we'll focus on what the application looks like(styling).


https://mackenziechild.me/12-in-12/6/  

![image](https://github.com/TimingJL/Todo-App/blob/master/pic/index_styling.jpeg)
![image](https://github.com/TimingJL/Todo-App/blob/master/pic/show_styling.jpeg)
![image](https://github.com/TimingJL/Todo-App/blob/master/pic/todo_list_new_styling.jpeg)
![image](https://github.com/TimingJL/Todo-App/blob/master/pic/todo_item_edit_styling.jpeg)


### Highlights of this course
1. Todo Lists
2. Todo Items
3. Mark As Complete
4. Custom Styling
5. HAML


# Create A Todo-App
```console
$ rails new todo
```


Chage directory to the pin_board. Under `Gemfile`, add `gem 'therubyracer'`, save and run `bundle install`.      

Note: 
Because there is no Javascript interpreter for Rails on Ubuntu Operation System, we have to install `Node.js` or `therubyracer` to get the Javascript interpreter.

```console
$ bundle install
```

Then run the `rails server` and go to `http://localhost:3000` to make sure everything is correct.

# Create The Ability To Add Todo-List
To start, we're going to quickly create the ability to add todo list by generating a todo-list scaffold.
```console
$ rails g scaffold todo_list title:string description:text
$ rake db:migrate
```

Let's confirm that is worked.          
We'll go to `http://localhost:3000/todo_lists`. And now, we have the ability to create todo-lists.        


### Set the Root of the Application
Then we set the root of the application.       
Let's open up `config/routes.rb`
```ruby
Rails.application.routes.draw do
  resources :todo_lists
  root "todo_lists#index"
end
```

Now, if we just go to `http://localhost:3000/`, it'll list out all our todos. 
![image](https://github.com/TimingJL/Todo-App/blob/master/pic/todo_scaffold.jpeg)


### Generate a Model for Todo Items
Next things we want to do is generate a model for our todo items.
```console
$ rails g model todo_item content:string todo_list:references
```

If you want to take a look at what that create for, you should go to `db/migrate/xxxxxxxxxxxxxx_create_todo_items.rb`
```ruby
class CreateTodoItems < ActiveRecord::Migration[5.0]
  def change
    create_table :todo_items do |t|
      t.string :content
      t.references :todo_list, foreign_key: true

      t.timestamps
    end
  end
end
```

And if you go under `app/models/todo_item.rb`, you can see it `belongs _to :todo_list`.
```ruby
class TodoItem < ApplicationRecord
  belongs_to :todo_list
end
```

So let's do our db migrate to migrate our database:
```console
$ rake db:migrate
```

From here, we want to add association between the todo-item and the todo-list.     
Under `app/models/todo_list.rb`
```ruby
class TodoList < ApplicationRecord
	has_many :todo_items
end
```

Then, we also want to create some routes for our todo-items.       
So in `config/routes.rb`
```ruby
Rails.application.routes.draw do
  resources :todo_lists do
  	resources :todo_items
  end
  
  root "todo_lists#index"
end
```
And you can see the routes by `$ rake routes`
```
                  Prefix Verb   URI Pattern                                             Controller#Action
    todo_list_todo_items GET    /todo_lists/:todo_list_id/todo_items(.:format)          todo_items#index
                         POST   /todo_lists/:todo_list_id/todo_items(.:format)          todo_items#create
 new_todo_list_todo_item GET    /todo_lists/:todo_list_id/todo_items/new(.:format)      todo_items#new
edit_todo_list_todo_item GET    /todo_lists/:todo_list_id/todo_items/:id/edit(.:format) todo_items#edit
     todo_list_todo_item GET    /todo_lists/:todo_list_id/todo_items/:id(.:format)      todo_items#show
                         PATCH  /todo_lists/:todo_list_id/todo_items/:id(.:format)      todo_items#update
                         PUT    /todo_lists/:todo_list_id/todo_items/:id(.:format)      todo_items#update
                         DELETE /todo_lists/:todo_list_id/todo_items/:id(.:format)      todo_items#destroy
              todo_lists GET    /todo_lists(.:format)                                   todo_lists#index
                         POST   /todo_lists(.:format)                                   todo_lists#create
           new_todo_list GET    /todo_lists/new(.:format)                               todo_lists#new
          edit_todo_list GET    /todo_lists/:id/edit(.:format)                          todo_lists#edit
               todo_list GET    /todo_lists/:id(.:format)                               todo_lists#show
                         PATCH  /todo_lists/:id(.:format)                               todo_lists#update
                         PUT    /todo_lists/:id(.:format)                               todo_lists#update
                         DELETE /todo_lists/:id(.:format)                               todo_lists#destroy
                    root GET    /                                                       todo_lists#index
```                    
If you look all the `todo_items`, they are nested under the `todo_lists`.


Next thing we want do to is generate a controller for our todo-items.
```console
$ rails g controller todo_items
```

And we want the ability to create our todo-items.        
In `app/controllers/todo_items_controller.rb`
```ruby
class TodoItemsController < ApplicationController
	before_action :set_todo_list

	def create
		@todo_item = @todo_list.todo_items.create(todo_item_params)

		redirect_to @todo_list
	end

	private

	def set_todo_list
		@todo_list = TodoList.find(params[:todo_list_id])
	end

	def todo_item_params
		params[:todo_item].permit(:content)
	end
end
```

Then, we want to create two partial under `app/views/items` (`_todo_item.html.erb` and `_form.html.erb`).       
So in the `app/views/items/_form.html.erb`, what we want to do is create a `form_for`

```html
<%= form_for([@todo_list, @todo_list.todo_items.build]) do |f| %>
	<%= f.text_field :content, placeholder: "New Todo"%>
	<%= f.submit %>
<% end %>
```

Then in the `app/views/items/_todo_item.html.erb`
```html

 <p><%= todo_item.content %></p>
```

Next, what we want to do is show the form as well as the todo-items under the todo-list show page.       
In `app/views/todo_lists/show.html.erb`
```html

	<p id="notice"><%= notice %></p>

	<p>
	  <strong>Title:</strong>
	  <%= @todo_list.title %>
	</p>

	<p>
	  <strong>Description:</strong>
	  <%= @todo_list.description %>
	</p>

	<div id="todo_items_wrapper">
		<%= render @todo_list.todo_items %>
		<div id="form">
			<%= render "todo_items/form" %>
		</div>	
	</div>

	<%= link_to 'Edit', edit_todo_list_path(@todo_list) %> |
	<%= link_to 'Back', todo_lists_path %>
```

Now, you can see we have the title, the description and a `New Todo` form.
![image](https://github.com/TimingJL/Todo-App/blob/master/pic/new_todo_form.jpeg)


# Delete A Todo-Item
I also want the ability to delete a todo-item.         
In `app/views/todo_items/_todo_Item.html.erb`
```html

	<p><%= todo_item.content %></p>
	<p><%= link_to  "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %></p>
```

And in `app/controllers/todo_items_controller.rb`, we add a delete action:
```ruby
def destroy
	@todo_item = @todo_list.todo_items.find(params[:id])
	if @todo_item.destroy
		flash[:success] = "Todo List item was deleted."
	else
		flash[:error] = "Todo List item could not be deleted."
	end
	redirect_to @todo_list
end
```

# Mark A Item As Complete
Next, we want the ability to mark a item as complete.         
So, what we gonna do next is we want to add a migration. To add a completed date time to our todo-item's table.         
```console
$ rails g migration add_completed_at_to_todo_items completed_at:datetime
$ rake db:migrate
```

Next, we need to add some routes for the ability to mark as complete.        
In `config/routes.rb`
```ruby
Rails.application.routes.draw do
  resources :todo_lists do
  	resources :todo_items do
  		member do
  			patch :complete
  		end
  	end
  end

  root "todo_lists#index"
end
```


In `app/views/todo_items/_todo_item.html.erb`
```html

	<div class="row clearfix">
		<% if todo_item.completed? %>
			<div class="complete">
				<%= link_to "Mark as Complete", complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch %>
			</div>
			<div class="todo_item">
				<p style="opacity: 0.4;"><strike><%= todo_item.content %></strike></p>
			</div>
			<div class="trash">
				<%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>
			</div>
		<% else %>
			<div class="complete">
				<%= link_to "Mark as Complete", complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch %>
			</div>
			<div class="todo_item">
				<p><%= todo_item.content %></p>
			</div>
			<div class="trash">
				<%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>
			</div>
		<% end %>
	</div>
```

Next, we need to add complete action in our `app/controllers/todo_items_controller.rb`
```ruby
class TodoItemsController < ApplicationController
	before_action :set_todo_list
	before_action :set_todo_item, except: [:create]

	def create
		@todo_item = @todo_list.todo_items.create(todo_item_params)
		redirect_to @todo_list
	end

	def destroy
		if @todo_item.destroy
			flash[:success] = "Todo List item was deleted."
		else
			flash[:error] = "Todo List item could not be deleted."
		end
		redirect_to @todo_list
	end

	def complete
		@todo_item.update_attribute(:completed_at, Time.now)
		redirect_to @todo_list, notice: "Todo item complete"
	end

	private

	def set_todo_list
		@todo_list = TodoList.find(params[:todo_list_id])
	end

	def set_todo_item
		@todo_item = @todo_list.todo_items.find(params[:id])
	end

	def todo_item_params
		params[:todo_item].permit(:content)
	end
end
```


And under our model `app/models/todo_item.rb`, if it completed, it should not be blank.         
```ruby
class TodoItem < ApplicationRecord
  belongs_to :todo_list

  def completed?
  	!completed_at.blank?
  end
end
```
![image](https://github.com/TimingJL/Todo-App/blob/master/pic/mark_as_complete.jpeg)


# Styling
The next thing I want to do is make it look quite a bit nicer.      
What I'm going to to first is rename `application.css` as `application.css.scss` in `app/assets/stylesheets/`.        
And delete the `app/assets/stylesheets/scaffolds.scss`, `app/assets/stylesheets/todo_items.scss`, and `app/assets/stylesheets/todo_lists.scss`.


And paste this to `app/assets/stylesheets/application.scss`.
```scss
/*
 * This is a manifest file that'll be compiled into application.css, which will include all the files
 * listed below.
 *
 * Any CSS and SCSS file within this directory, lib/assets/stylesheets, vendor/assets/stylesheets,
 * or vendor/assets/stylesheets of plugins, if any, can be referenced here using a relative path.
 *
 * You're free to add application-wide styles to this file and they'll appear at the bottom of the
 * compiled file so the styles you add here take precedence over styles defined in any styles
 * defined in the other CSS/SCSS files in this directory. It is generally better to create a new
 * file per style scope.
 *
 *= require_tree .
 *= require_self
 */

 $white_opaque: rgba(250, 250, 250, .3);
 $dark: #1F7972;

* {
	box-sizing: border-box;
}
html {
  height: 100%;
}

body {
 	height: 100%;
  background: -webkit-linear-gradient(40deg, #4CB8C4 10%, #EAECC6 100%);
  background:    -moz-linear-gradient(40deg, #4CB8C4 10%, #EAECC6 100%);
  background:     -ms-linear-gradient(40deg, #4CB8C4 10%, #EAECC6 100%);
  background:      -o-linear-gradient(40deg, #4CB8C4 10%, #EAECC6 100%);
  background:         linear-gradient(40deg, #4CB8C4 10%, #EAECC6 100%);
  font-family: 'Lato', sans-serif;
}

.clearfix:before,
.clearfix:after {
    content: " ";
    display: table;
}

.clearfix:after {
    clear: both;
}

#notice {
	text-align: center;
	font-size: 0.6em;
	color: $dark;
	font-weight: 100;
}

.container {
	width: 50%;
	max-width: 750px;
	background: $white_opaque;
	margin: 3em auto 0 auto;
	border-radius: 7px;
	-webkit-box-shadow: 0 0 4px 3px rgba(0,0,0,.3);
	box-shadow: 0 0 4px 4px rgba(0,0,0,.03);
	padding: 1em 0;
}

.todo_list_title {
	text-align: center;
	font-weight: 700;
	font-size: 2.5rem;
	text-transform: uppercase;
	color: white;
	margin: 0;
	a {
		text-decoration: none;
		color: white;
		transition: all .4s ease-in-out;
		&:hover {
			opacity: 0.4;
		}
	}
}

.todo_list_sub_title {
	margin: 0 0 3em 0;
	text-align: center;
	font-size: 0.9em;
	color: $dark;
	font-weight: 100;
}

.index_row {
	padding: 2em;
	border-bottom: 1px solid rgba(250,250,250, .3);
	.todo_list_sub_title {
		margin-bottom: 0;
	}
}

#todo_items_wrapper {
	.row {
		width: 100%;
		border-top: 1px solid rgba(250,250,250, .3);
		border-bottom: 1px solid rgba(250,250,250, .3);
		.complete {
			width: 10%;
			float: left;
			text-align: center;
			border-right: 1px solid rgba(250,250,250, .3);
			padding: 1em 0;
		}
		.todo_item {
			width: 80%;
			float: left;
			p {
				margin: 0;
				padding: 1em;
				color: $dark;
				font-weight: 100;
			}
		}
		.trash {
			width: 10%;
			float: left;
			text-align: center;
			border-left: 1px solid rgba(250,250,250, .3);
			padding: 1em 0;
		}
		i {
			color: white;
			transition: all .4s ease-in-out;
			&:hover {
				color: $dark;
			}
		}
	}
	#form {
		margin-top: 2em;
		padding: 0 5%;
		input[type="text"] {
			width: 72%;
			margin-right: 2%;
			display: inline-block;
			outline: none;
			background: rgba(250,250,250,.4);
			border: none;
			height: 40px;
			border-radius: 4px;
			padding: 1em 2em;
		}
		input[type="submit"] {
			background: rgba(250,250,250,.4);
			outline: none;
			border: none;
			height: 40px;
			border-radius: 4px;
			width: 25%;
			display: inline-block;
			transition: all .4s ease-in-out;
			cursor: pointer;
			&:hover {
				background: $dark;
			}
		}
		::-webkit-input-placeholder { color: $dark; }
	}
}

.links {
	display: block;
	margin: 1.5em auto 0 auto;
	text-align: center;
	font-size: 0.8em;
	color: white;
	a {
		color: white;
	}
}

.forms {
	padding: 0 5%;
}
label {
	color: $dark;
}
input[type="text"], textarea {
	width: 100%;
	margin: .5em 2% 1em 0;
	display: inline-block;
	outline: none;
	background: rgba(250,250,250,.4);
	border: none;
	height: 40px;
	border-radius: 4px;
	padding: 1em 2em;
}
textarea {
	height: 200px;
}
input[type="submit"] {
	background: white;
	outline: none;
	border: none;
	height: 40px;
	border-radius: 4px;
	width: 25%;
	display: inline-block;
	transition: all .4s ease-in-out;
	cursor: pointer;
	color: $dark;
	&:hover {
		background: $dark;
		color: white;
	}
}
::-webkit-input-placeholder { color: $dark; }
```



And in our views `app/views/layouts/application.html.erb`
```ruby

	<!DOCTYPE html>
	<html>
	<head>
	  <title>Todo</title>
	  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
	  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
	  <%= csrf_meta_tags %>
	</head>
	<body>

	<div class="container">
		<%= yield %>
	</div>

	</body>
	</html>
```


In `app/views/todo_lists/index.html.erb`
```html

	<% @todo_lists.each do |todo_list| %>
	  <div class="index_row clearfix">
	    <h2 class="todo_list_title"><%= link_to todo_list.title, todo_list %></h2>
	    <p class="todo_list_sub_title"><%= todo_list.description %></p>
	  </div>
	<% end %>

	<div class="links">
	  <%= link_to "New Todo List", new_todo_list_path %>
	</div>
```
![image](https://github.com/TimingJL/Todo-App/blob/master/pic/index_styling.jpeg)

In `app/views/layouts/application.html.erb`, there is two things we need to add. One, I use a google font, as well as a awesome font for the icon.      
Google font:       
https://www.google.com/fonts/specimen/Lato             

Font Awesome:
http://fontawesome.io/get-started/       
```html

	<!DOCTYPE html>
	<html>
	<head>
	  <title>Todo</title>
	  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
	  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
	  <%= csrf_meta_tags %>
	  <link href='http://fonts.googleapis.com/css?family=Lato:300,400,700' rel='stylesheet' type='text/css'>
	  <link href="//maxcdn.bootstrapcdn.com/font-awesome/4.2.0/css/font-awesome.min.css" rel="stylesheet">
	</head>
	<body>

	<div class="container">
		<%= yield %>
	</div>

	</body>
	</html>
```


In `app/views/todo_items/_todo_item.html.erb`
```html

	<div class="row clearfix">
		<% if todo_item.completed? %>
			<div class="complete">
				<%= link_to complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch do %>
					<i style="opacity: 0.4;" class="fa fa-check"></i>
				<% end %>
			</div>
			<div class="todo_item">
				<p style="opacity: 0.4;"><strike><%= todo_item.content %></strike></p>
			</div>
			<div class="trash">
				<%= link_to todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } do %>
					<i class="fa fa-trash"></i>
				<% end %>
			</div>
		<% else %>
			<div class="complete">
				<%= link_to complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch do %>
					<i class="fa fa-check"></i>
				<% end %>
			</div>
			<div class="todo_item">
				<p><%= todo_item.content %></p>
			</div>
			<div class="trash">
				<%= link_to todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } do %>
					<i class="fa fa-trash"></i>
				<% end %>
			</div>
		<% end %>
	</div>
```


In `app/views/todo_lists/show.html.erb`
```html

	<p id="notice"><%= notice %></p>

	<h2 class="todo_list_title"><%= @todo_list.title %></h2>
	<p class="todo_list_sub_title"><%= @todo_list.description %></p>

	<div id="todo_items_wrapper">
		<%= render @todo_list.todo_items %>
		<div id="form">
			<%= render "todo_items/form" %>
		</div>
	</div>

	<div class="links">
		<%= link_to 'Edit', edit_todo_list_path(@todo_list) %> |
		<%= link_to 'Delete', todo_list_path(@todo_list), method: :delete, data: { confirm: "Are you sure?" }
	 %> |
	  <%= link_to 'Back', todo_lists_path %>
	</div>
```

And in `app/controllers/todo_lists_controller.rb`, we change the
```
format.html { redirect_to todo_lists_url, notice: 'Todo list was successfully destroyed.' }
``` 
to
```
format.html { redirect_to root_url, notice: 'Todo list was successfully destroyed.' }
```
![image](https://github.com/TimingJL/Todo-App/blob/master/pic/show_styling.jpeg)

Next thing we want to do is the forms.        
Let's go under our `app/views/todo_lists/new.html.erb`
```html

	<h1 class="todo_list_title">New Todo List</h1>

	<div class="forms">
		<%= render 'form' %>
	</div>

	<div class="links">
		<%= link_to 'Back', todo_lists_path %>
	</div>
```

In `app/views/todo_lists/_form.html.erb`
```

	<%= form_for(@todo_list) do |f| %>
	  <% if @todo_list.errors.any? %>
	    <div id="error_explanation">
	      <h2><%= pluralize(@todo_list.errors.count, "error") %> prohibited this todo_list from being saved:</h2>

	      <ul>
	      <% @todo_list.errors.full_messages.each do |message| %>
	        <li><%= message %></li>
	      <% end %>
	      </ul>
	    </div>
	  <% end %>

	  <div class="field">
	    <%= f.label :title %><br>
	    <%= f.text_field :title %>
	  </div>
	  <div class="field">
	    <%= f.label :description %><br>
	    <%= f.text_area :description %>
	  </div>
	  <div class="actions">
	    <%= f.submit %>
	  </div>
	<% end %>
```
![image](https://github.com/TimingJL/Todo-App/blob/master/pic/todo_list_new_styling.jpeg)


In `app/views/todo_lists/edit.html.erb`
```ruby

	<h1 class="todo_list_title">Edit Todo List</h1>

	<div class="forms">
		<%= render 'form' %>
	</div>

	<div class="links">
		<%= link_to 'Cancel', todo_lists_path %>
	</div>
```

![image](https://github.com/TimingJL/Todo-App/blob/master/pic/todo_item_edit_styling.jpeg)


Complete!