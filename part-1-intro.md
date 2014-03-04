# Backbone, The Primer #
This is the first of four posts that guest author Greg MacWilliam has put together for Data Community DC: _Backbone, The Primer_. 

**Editors Note:**
Backbone.js is an MVC framework for front-end developers. It allows rapid development and prototyping of rich AJAX web applications in the same way that Django or Rails does for the backend. Part of the Data Science pipeline is reporting and visualization; and rich HTML reports that use Javascript libraries like D3 or Highcharts are already part of the Data Scientist's toolkit. As you'll see from this primer, leveraging Backbone will equip Data Scientists with the means to quickly deploy to the front-end without having to wade through many of the impediments  that typically plague browser development. 

-------

A common sentiment I hear from developers coming to Backbone is that they don't know where to start with it. Unlike full-featured frameworks with prescribed workflows (ie: Angular or Ember), Backbone is a lightweight library with few opinions. At its worst, some would say that Backbone has TOO few opinions. At its best thought, Backbone is a flexible component library designed to provide a baseline solution for common application design patterns.

The core of Backbone provides a comprehensive RESTful service package. This primer assumes that you have a basic understanding of REST services, and will focus on the interactions of Backbone components with RESTful data services.


## What's In The Box?

The first thing to familiarize with are the basic components provided by Backbone. There are three foundation components that make up Backbone applications:

- ### Backbone.Model
Models store application data, and sync with REST services. A model may predefine its default attributes, and will emit events when any of its managed data attributes change.

- ### Backbone.Collection
Collections manage a list of models, and sync with REST services. A collection provides basic search methods for querying its managed models, and emits events when the composition of its models change.

- ### Backbone.View
A view connects a model to its visual representation in the HTML Document Object Model (or, the "DOM"). Views render their associated model's data into the DOM, and capture user input from the DOM to send back into the model.

- ### Bonus… Underscore!
Backbone has two JavaScript library dependencies: jQuery and UnderscoreJS. Chances are good that you're familiar with jQuery. If you don't know Underscore, review its [documentation](http://underscorejs.org). Underscore provides common functional programming for working with data structures. When setting up Backbone, you get the full capabilities of Underscore as part of the package!

While Backbone does include additional useful features, we'll be focusing on these core components in this primer.

## Got REST Services?

For this primer, let's assume the following RESTful Muppets data service exists:

`GET /muppets/`  
Gets a list of all Muppets within the application. Returns an array of all Muppet models (with some additional meta data):

	{
		"total": 2,
		"page": 1,
		"perPage": 10,
		"muppets": [
		  {
		    "id": 1,
		    "name": "Kermit",
		    "occupation": "being green"
		  },
		  {
		    "id": 2,
		    "name": "Gonzo",
		    "occupation": "plumber"
		  }
		]
	}

`POST /muppets/`  
Creates a new Muppet model based on the posted data. Returns the newly created model:

	{
	  "id": 3,
	  "name": "Animal",
	  "occupation": "drummer"
	}


`GET /muppets/:id`  
`PUT /muppets/:id`  
`DEL /muppets/:id`  
Gets, modifies, and/or deletes a specific user model. All actions return the requested/modified model:

	{
	  "id": 1,
	  "name": "Kermit",
	  "occupation": "being green"
	}

## What's Next? ##

In the next three posts, we'll discuss in more detail Models and Collections, Views, Event Binding, and walk through the creation of a simple app based on the REST API described above.

* Part One: Introduction
* Part Two: Models and Collections
* Part Three: Views
* Part Four: A Simple App

### About the Author ###
Greg is primarily focused on web front-end with JavaScript, specializing in large-scale web applications and single-page applications. He is also experienced with HTML5, CSS3, SCSS, responsive design practices, and visualizing large data sets using SVG, Canvas, and major web mapping frameworks. He is also fluent with RESTful API design, Django, Rails, Node, and some WordPress experience.

Current side projects include EpoxyJS (data binding extension for Backbone), ConstellationJS (a grid-based geometry toolkit), and the HTML5/Django-based Lassie Constellation admin system.