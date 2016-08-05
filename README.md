# Learning by Doing 

![Ubuntu version](https://img.shields.io/badge/Ubuntu-16.04%20LTS-orange.svg)
![Rails version](https://img.shields.io/badge/Rails-v5.0.0-blue.svg)
![Ruby version](https://img.shields.io/badge/Ruby-v2.3.1p112-red.svg)

Mackenzie Child's video really inspired me. So I decided to follow all of his rails video tutorial to learn how to build a web app. Through the video, I would try to build the web app by my self and record the courses step by step in text to facilitate the review.


# Project 6: How To Build A Todo App With Rails       
This time we build a pretty simple application. It's a Todo-list. We have multiple lists and multiple todo items for each list. And the ability to check an item if the item has been completed or not as well as delete the item. And we'll focus on what the application looks like(styling).


https://mackenziechild.me/12-in-12/6/  



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

To be continued...