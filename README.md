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