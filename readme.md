# Forms and Resources
## Aww, CRUD!


## Intro

| Objectives |
| :---- |
| Review **CRUD** in the context of a Rails application, especially **Updating** and **Deleting** a resource. |
| Examine **form helpers** and **partials** (if time permits) in a  Rails Application. |
| Apply styling and **Bootstrap** to our site to create a custom layout. |


### Previously

* Routing requests to a controller method
	* **INDEX**
	* **NEW** 
	* **CREATE**
* RESTful resources
* Basic HTML Forms 	

### Outline

| Background |
| :--- |
| A bog is a swampy landmass that accumulates little green aliens, young padawans, and peat: a deposit of dead plant material—often mosses. |

Researchers are collecting data on a local bog and need an app to quickly record field data on all of the creepy crawly creatures that may lurk in the area. Our goal is to create a **Bog App**. 

When you hear bog you may think of this...

The Planet Dagobah!
![luke](http://1.bp.blogspot.com/-Aa0TuXoIMoU/T4M7_GbT8uI/AAAAAAAAin8/lcUZkoqoJM4/s1600/Yoda-and-Luke.jpg)  

The Swamps of Sadness
![Bye Bye, Artax!](https://readingotherpeople.files.wordpress.com/2015/06/artax.jpg)  

Louisiana!
![Louisiana](http://i.imgur.com/z56mk.jpg)  



### Part I Overview:

* Drop in Bootstrap
* Create a **Creature** model.
	* verify it works in console 
* Setup a **`/creatures/`** index
	* iterate over each post
* Setup a **`/creatures/new`** and **CREATE**
	* Use form helpers with a new `Creature`
* Setup a **`/creatures/:id`** to show a particular `creature`

### Part II Overview:

* Setup a **`/creatures/:id/edit`** and **UPDATE**
* Setup a **Delete**


## Part I: Review and Refactor


### CRUD and REST Reference

Typically we associate **CRUD** with the following **HTTP** methods

| CRUD Operation | HTTP Method | Example|
| :---  |	:--- | :-- |
| Create | POST | `POST "/puppies?name=spot"` (create a puppy named spot) |
| Read   | GET  | `GET "/puppies"` (Shows all puppies) |
| Update | PUT or UPDATE | `PUT "/puppies/1?name=lassy"` (change puppy number 1 to have name lassy) |
| Delete | DELETE | `Delete "/puppies/1"` (destroy the first puppy, yikes!!!!) |

REST stands for **REpresentational State Transfer**. We will demonstrate these practices throughout this lesson, but for now preparing don't worry too much about it yet.



### 1.  Setup Rails New

* `$ rails new bog_app -T`
* `$ cd bog_app`
* `rake db:create`
* `$ rails s`

Now our app is up and running, [localhost:3000](localhost:3000/). 

### 2.  Drop in Bootstrap

Just put the third party css libraries in `vendor/assets` and for bootstrap just file it under stylesheets.

```
 curl https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css > vendor/assets/stylesheets/bootstrap-3.2.0.min.css
```
### 3.  Create A Creature 


In terminal, we create our `Creature` model using a rails generator as follows,

	$ rails g model creature name description
	$ rake db:migrate

#### 4.  Verify it works

We go straight into terminal to enter *rails console*.

	$ rails console

	> Creature.create({name: "Yoda", description: "Green little man"}) 
	=> #<Creature ....>

*This will avoid issues later with `index` trying to render Creatures that aren't there.*

In `db/seeds.rb`

```ruby	
Creature.create({name: "Luke", description: "Jedi"}) 
Creature.create({name: "Darth Vader", description: "Father of Luke"}) 
```


 and then just run `$ rake db:seed` in console. This will now get run every time you `rake db:reset`.
 
#### Seeds

When we create an application in development we will want some mock data to play so we can just drop this into the `db/seeds.rb` file.



### 5.  Routes


Go to `config/routes.rb` and inside the routes block erase all the commented text. It should now look exactly as follows

`config/routes.rb`

```ruby
Rails.application.routes.draw do

end
```




Now we can define all our routes.

Your `routes.rb` will just be telling your app how to connect *HTTP* requests to a **Controller**. Let's get ready for our first route. 

NOTE: The nature of any route goes as follows:
 
```ruby
request_type '/for/some/path/goes', to: "controller#method"
```

e.g. if we had a `PuppiesController` that had a `index` method we could say

```ruby
get "/puppies", to: "puppies#index"
```


#### 6.  Creatures Index Route

Using the above routing pattern we'll write our first route

In `/config/routes.rb`
  
```ruby
RouteApp::Application.routes.draw do
	root to: 'creautres#index'
	## Also just to keep it RESTful
	get '/creatures', to: "creatures#index"
end
```



#### 7.  Creatures Controller and Index Method

Let's begin with the following 

	$ rails g controller creatures

which is a generator for creating our controller.

	
We first need to setup our `#index` method in `creatures`

In `app/controllers/creatures_controller.rb`

```ruby
class CreaturesController < ApplicationController
	
		def index
			@creatures = Creature.all
			render :index
		end
	
	...
	
end
```

#### 8.  Creatures Index View

If you look at your views the `views/creatures` folder has already be created so we just need to add the file below:

In `app/views/creatures/index.html.erb`

```html	
	<% @creatures.each do |creature| %>
		
		<div>
			Name: <%= creature.name %> <br>
			Description: <%=  creature.description %>
		</div>
	
	<% end %>
```



### 9.  A new route for Creatures

The *RESTful* convention would be to make a form available at `/creatures/new`. Let's add this route.

In `/config/routes.rb`

```ruby
	Rails.application.routes.draw do
		root to: 'creatures#index'
		
		# just to be RESTful
		get '/creatures', to: 'creatures#index'
		get '/creatures/new', to: 'creatures#new'
		
	end
```

### 10.  A new method for Creatures

The request for `/Creatures/new` will search for a `Creatures#new`, so we must create a method to handle this request. This will render the `new.html.erb` in the `app/views/Creatures` folder.

In `app/controllers/creatures_controller.rb`

```ruby
	class CreaturesController < ApplicationController
		
		...
			def new
				render :new
			end
		
		...
		
	end
```


### 11.  A new view for Creatures

Let's create the `app/views/Creatures/new.html.erb` with a form that the user can use to sumbit new Creatures to the application. Note: the action is `/Creatures` because it's the collection we are submiting to, and the method is `post` because we want to create.

In `app/views/creatures/new.html.erb`

```html
<%= form_for :creature, url: "/creatures", method: "post" do |f| %>
	
	<%= f.text_field :name %>
	<%= f.text_area :description %>
	<%= f.submit "save creature" %>
<% end %>
```



### 12.  A Create Route

 We have now defined our next `route` in our `new.html.erb` as we are directing all form posts to the following:

```ruby	
post "/creatures", to: "creatures#create"
```
	
when we said

```ruby	
url: "/creatures", method: "post"
```

and so we add it to our  `/config/routes.rb`

```ruby
Rails.application.routes.draw do
	root to: 'creatures#index'
	
	# just to be RESTful
	get '/creatures', to: 'creatures#index'
	get '/creatures/new', to: 'creatures#new'
	post "/creatures", to: "creatures#create"
end
```
	
#### 13.  A Create Method

Let's create the `creatures#create` method 

In `app/controllers/creatures_controller.rb`

```ruby
class CreaturesController < ApplicationController
	
	...
		def create
			new_creature = params.require(:creature).permit(:name, :description)
			Creature.create(new_creature)
			redirect_to "/creatures"
		end
	
	...
	
end
```

#### 14.  A smarter new view for Creatures

Let's update our `creatures#new` method 

In `app/controllers/creatures_controller.rb`

```ruby
class CreaturesController < ApplicationController
	
	...
		def new
			@creature = Creature.new
			render :new
		end
	
	...
	
end
```

This sets `@creature` to a new instance of a `Creature` which we can now share with or `new.html.erb` and thus our `form_helper`
	
In `app/views/creatures/new.html.erb`

```html	
<%= form_for @creature do |f| %>
	
	<%= f.text_field :name %>
	<%= f.text_area :description %>
	<%= f.submit "save creature" %>
	
<% end %>
```



### 15.  Show Route

Right now, our app redirects to  `#index` after a create, which isn't helpful for quickly verifying what you just created. To do this we create a `#show`.

Let's add our `show` route.

In `/config/routes.rb`

```ruby
Rails.application.routes.draw do
	root to: 'creatures#index'
	
	# just to be RESTful
	get '/creatures', to: 'creatures#index'
	get '/creatures/new', to: 'creatures#new'
	# rake routes to check this route out
	get '/creatures/:id', to: 'creatures#show'
	post "/creatures", to: "creatures#create"
end
```


Our `/creatures/:id` path is below our `/creatures/new` path. If we had `creatures/new` below the show route then the pattern matching will cause an error where all requests for `/creaturess/new` get sent to the show.


####16.  A controller method  


In `app/controllers/creatures_controller.rb`

```ruby
class CreaturesController < ApplicationController
	
	...
		def show
			id = params[:id]
			@creature = Creature.find(id)
			render :show
		end
	
	...
	
end
```

####17.  A view for showing a creature


In `app/views/creatures/show.html.erb`

```html
<div>
	Name: <%= @creature.name %> <br>
	Description: <%=  @creature.description %>
</div>
```	

####18.  Changing the `#create` redirect

The `#create` method redirects to `#index` (the `/creaures` path), but this isn't very helpful for verrifying that a newly created creature was properly created. The best way to fix this is to have it redirect to `#show`.


In `app/controllers/creatures_controller.rb`

```ruby
class CreaturesController < ApplicationController
	
	...
		def create
			new_creature = params.require(:creature).permit(:name, :description)
			creature = Creature.create(new_creature)
			redirect_to "/creatures/#{creature.id}"
		end
	
	...
	
end
```

## Part II: Setup Edit, Update, and Delete

Editing a Creature model requires two seperate methods. One to display the model information to be edited by the client, and another to handle updates submitted by the client.

If look back at how we handled the getting of our `new` form we see the following pattern.

* Make a route first
* Define a controller method 	
* render view

The only difference is that now we need to use the `id` of the object to be edited. We get the following battle plan.

* Make a route first
	* Make sure it specifies the `id` of the thing to be edited
* Define a controller method 
	* Retrieve the `id` of the model to be edited from `params`
	* use the `id` to find the model
* render view
	* use model to display in the form 

###19.   Getting to an Edit

We begin with handling the request from a client for an edit page. 

* We can easily define a **RESTful** route to handle getting the edit page as follows

In `/config/routes.rb`

```ruby	
Rails.application.routes.draw do
	root to: 'creatures#index'
	
	get '/creatures', to: 'creatures#index'
	
	get '/creatures/new', to: 'creatures#new'
	
	get '/creatures/:id', to: 'creatures#show'
		
	get '/creatures/:id/edit', to: 'creatures#show'		
	
	post "/creatures", to: "creatures#create"
end
```		

* Similarly, using our `#show` method as inspiration we can write an `#edit` method

In `app/controllers/creatures_controller.rb`

```ruby	
class CreaturesController < ApplicationController
	
	...
		def edit
			id = params[:id]
			@creature = Creature.find(id)
			render :edit
		end
	
	...
	
end
```

* Let's quickly begin the setup of an `edit` form using our `new.html.erb` from earlier. To see how the form is different we will need to render it and check it out in Chrome console.

In `app/views/creatures/new.html.erb`

```html	
<%= form_for @creature do |f| %>
	
	<%= f.text_field :name %>
	<%= f.text_area :description %>
	<%= f.submit "update creature" %>
	
<% end %>
```

Going to [creatures/1/edit](localhost:3000/creatures/1/edit) we get the following error:
	
	undefined method `creature_path' for #<#<Class:0x007fc5fc41be68>:0x007fc5fc40ea38>

This is because when we `rake routes` we notice that there is no `prefix` for the `creature` which rails uses internally to generate methods for you.

In `/config/routes.rb`

```ruby
Rails.application.routes.draw do
	root to: 'creatures#index'
	
	# Add prefixes to routes using `as: "some_prefix"` syntax
	get '/creatures', to: 'creatures#index', as: "creatures"
	
	get '/creatures/new', to: 'creatures#new', as: "new_creature"
	
	get '/creatures/:id', to: 'creatures#show', as: "creature"
		
	get '/creatures/:id/edit', to: 'creatures#show', as: "edit_creature"		
	
	post "/creatures", to: "creatures#create"
end
```		
	


That's pretty much the whole-shebang when comes to getting an edit page. Our previous knowledge has really come to help us understand what we need to do. We'll see this also true for the update that still needs to be handled witht the submission of the form above.


### 20.  Putting updated form data 

Looking back at how we handled the submission of our `new` form we see the following pattern.

* Make a route first
* Define a controller method 
* redirect to something

The only difference now is that we will need to use the `id` of the object being update.

* Make a route first
	* Make sure it specifies the `id` of the thing to be **updated**
* Define a controller method 
	* Retrieve the `id` of the model to be **updated** from `params`
	* use the `id` to find the model
	* retrieve the updated info sent from the form in `params`
	* update the model
* redirect to show
	* use `id` to redirect to `#show` 
	
### 21.  Putting it into action

* **Make a route** that uses the `id` of the object to be updated

In `/config/routes.rb`

```ruby	
Rails.application.routes.draw do
	root to: 'creatures#index'
	
	# Add prefixes to routes using `as: "some_prefix"` syntax
	get '/creatures', to: 'creatures#index', as: "creatures"
	
	get '/creatures/new', to: 'creatures#new', as: "new_creature"
	
	get '/creatures/:id', to: 'creatures#show', as: "creature"
		
	get '/creatures/:id/edit', to: 'creatures#show', as: "edit_creature"		
	
	post "/creatures", to: "creatures#create"
	
	# The update route
	patch "/creatures/:id", to: "creatures#update"
end
```		
**Note** the method we now need to create is called `#update`

* In the `CreaturesController` we will create the `#update` method mentioned above
	
In `app/controllers/creatures_controller.rb`

```ruby	
class CreaturesController < ApplicationController
	
	...
	
	def update
		creature_id = params[:id]
		creature = Creature.find(Creature_id)
		
		# get updated data
		updated_attributes = params.require(:creatue).permit(:name, :description)
		# update the creature
		creature.update_attributes(updated_attributes)
		
		#redirect to show
		redirect_to "/creatures/#{creature_id}"
	end
	
end
```

###22.  Destroy!!!!


Following a similar pattern to the above we create a route for a destroy that uses the `id` of the model to be deleted.

In `/config/routes.rb`

```ruby	
Rails.application.routes.draw do
	root to: 'creatures#index'
	
	# Add prefixes to routes using `as: "some_prefix"` syntax
	get '/creatures', to: 'creatures#index', as: "creatures"
	
	get '/creatures/new', to: 'creatures#new', as: "new_creature"
	
	get '/creatures/:id', to: 'creatures#show', as: "creature"
		
	get '/creatures/:id/edit', to: 'creatures#show', as: "edit_creature"		
	
	post "/creatures", to: "creatures#create"
	
	# The update route
	patch "/creatures/:id", to: "creatures#update"
	
	# the destroy route
	delete "/creatures/:id", to: "creatures#destroy"
end
```	

Next we create a method for it in the `CreaturesController`

In `app/controllers/creatures_controller.rb`

```ruby
class CreaturesController < ApplicationController
	
	...
	
	def destroy
		id = params[:id]
		creature = Creature.find(id)
		creature.destroy
		redirect_to "/creatures"
	end
	
end
```

If you were tempted to use [`Creature.delete`](http://apidock.com/rails/ActiveRecord/Base/delete/class) that would be fine here because there are no associations. However, we need to use `model.destroy` if we want to avoid issues later.

Let's add a delete button to another view. 


In `app/views/creatures/index.html.erb`

```html
<% @creatures.each do |creature| %>
  <h2> <%= creature.name %> </h2>
  <p> <%= creature.description %></p>
  <%= button_to "Delete", creature, method: :delete %>
<% end %>
```

#CONGRATULATIONS! You have created a Bog App! Take a break, you look *Swamped*!

![Swamp Thing](http://orig11.deviantart.net/4b3a/f/2010/088/e/2/swamp_thing_by_killbabykill.jpg)
