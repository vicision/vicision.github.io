---
layout: post
title:  Caboodall, my Sinatra CMS app
date:   2017-05-16 16:41:04 -0400
---

![caboodall_logo](http://i.imgur.com/16HBbaX.png)

Time for another project blog post! This time around I'm writing about a Content Management System (CMS) web app I built on the Sinatra framework. It's an app for keeping track of your collections, from records to books or comics to movies--whatever you want, really. I'm calling it [Caboodall, and you can try it out for yourself here.](http://https://github.com/vicision/caboodall)

My initial plan when I began this project was to further build out [NYC Today](https://github.com/vicision/nyc-today-cli-app), my CLI project. I thought I might take that app's event listing capabilities, add user accounts, and allow users to save events to their own collections (or calendars), along with adding the ability for users to enter and edit their own events, all in a web-based graphical interface. I quickly realized that this would be a much more intricate and involved undertaking than was necessary at this time--though I absolutely plan to revisit it at a later day, . 

So I began to think about other options. Scanning my bedroom for inspiration, I took one look at the shelves of books and records that are overflowing along almost every wall and knew what to do. After all, I really ought to have some way to keep track of all of that junk anyway! 

I named the app Caboodall in reference to the common saying "the whole kit and caboodle" (of course), with the spelling modified to reflect its purpose of keeping track of "all" your stuff. 

I've kept the app pretty simple for the time being, with just enough features to be viable. Users can create an account, whereupon they're automatically logged in and shown their home screen:

![Caboodall Home](http://i.imgur.com/Lm0xbuH.png)

Clicking the "Add an item" button will bring up a form to enter the item's name, creator, and item type (for which a user can either enter a new type or choose from previously types in a dropdown menu. 

![Add Item Form](http://i.imgur.com/P84kU8D.png)

After clicking the "Add Item" button on the form, a message will appear letting the user know that their item was successfully added to its respective caboodall. Each new item type that's entered becomes a new "caboodall" to which more items can then be added. 

![Message](http://i.imgur.com/YIwAoKO.png)

The item name and caboodall name in the image above are links to the item and caboodall, respectively. Items can then be deleted from the "my caboodalls" page that displays each caboodall and the first few items in it, from each caboodall's own page, or from the item's own page. 

![Caboodalls Page](http://i.imgur.com/QY3XT3c.png)

And that's the basics of Caboodall's functionality. Now for the technical part! To begin, I should note that I didn't implement any update forms and their corresponding patch actions and routes, only because as basic as the app is, it's just as easy to delete an item for which a mistake has been made and add it again as it would be to update it with the correct info. Updating is one of many possible options I've considered adding when I build out the app with more features in the future. 

I should also give credit where it's due to Learn alumnus Brian Emory and his fantastic [Corneal gem,](https://github.com/thebrianemory/corneal) which makes generating a Sinatra app skeleton/scaffold a real breeze. I used it to jumpstart this app and I definitely recommend it to anyone else who's one. Thanks, Brian!

As the app stands, I have three models: `User`, `Item` and `Type`. `User` has many `types` and `items`, `Type` has many `items` and belongs to `user`, and `Item` belongs to `user` and to `type`. `User` includes `has_secure_password` in order to make use of the Bcrypt gem's password encryption. Each model also has a `slug` and `.find_by_slug` method to modify names into URL-friendly forms and then find by this form. You can see the ActiveRecord schema that created the tables for each model below:

![ActiveRecord Schema](http://i.imgur.com/cWVP4Sx.png)

There are four controllers, `ApplicationController` which inherits from `Sinatra::Base`, and then the Users, Items, and Types controllers which in turn inherit from `ApplicationController`. 

ApplicationController has the configuration for sessions, view folders and passwords along with helper methods `logged_in?` and `current_user` that prevent people who aren't logged in from viewing caboodalls and prevents users from accessing content belonging to other users when called in the various Controller routes. It also contains the `get` action for the root route `"/"`. 

`UsersController` has all of the actions for signing up and logging in and out of the application by rendering the views pages (`signup.erb` and `login.erb`) with the respective forms needed for these actions.

`ItemsController` has the `get` actions for the `/items` and `/new` routes which display the user's items (`show.erb`) and the form for adding a new item (`new.erb`), respectively. It also contains the `post` action that creates a new item in the database from the params in the `items/new.erb` view and the `get` action for the `/items/:slug` route that displays an individual item's view page. Lastly, it contains the `delete` action for the items.


`TypesController` simply has the actions for rendering the `index.erb` and `show.erb` views, which display a user's caboodalls of each type and the items in that caboodall type, respectively. 

Now that I've put you to sleep, I'll wrap this up by saying that this was once again a rewarding experience and I look forward to actually using this system to keep track of all of my books and records. Hopefully it can be of use to others in the future as well. If you ever end up trying it out, let me know what you think!






