# Backbone, The Primer

The most common sentiment I hear from developers coming to Backbone is that they don't know where to start with it. Unlike full-featured frameworks with prescribed workflows (ie: Angular or Ember), Backbone is a lightweight library with few opinions. At its worst, some would say that Backbone has TOO few opinions. At its best thought, Backbone is a flexible component library designed to provide a baseline solution for common application design patterns.

At its core, Backbone provides a comprehensive RESTful service package. This primer assumes that you have a basic understanding of REST services, and will focus on the interactions of Backbone components with RESTful data services.


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


## Usings Models

First, let's build a single model that manages data for Kermit. We know that Kermit's REST endpoint is `"/muppets/1"` (ie: `/muppets/:id`, and Kermit's id is "1"). Configured as a Backbone model, Kermit looks like this:

	var KermitModel = Backbone.Model.extend({
	  url: '/muppets/1',
	  defaults: {
	    id: null,
	    name: null,
	    occupation: null
	  }
	});

Kermit's model does two things:

- It defines a RESTful URL for his model to sync with, and…
- It defines default attributes for his model. Default attributes are useful for representing API data composition within your front-end code. Also, these defaults guarentee that your model is always fully formed, even before loading its data from ther server.

However, what IS that `KermitModel` object? When you extend a Backbone component, **you always get back a *constructor function***. That means we need to create an instance of the model before using it:

	var kermit = new KermitModel();
	
	kermit.fetch().then(function() {
	  kermit.get('name'); // >> "Kermit"
	  kermit.get('occupation'); // >> "being green"
	  kermit.set('occupation', 'muppet leader');
	  kermit.save();
	});

After creating a new instance of our Kermit model, we call `fetch` to have it load data from its REST endpoint. Calling `fetch` returns a promise object, onto which we can chain success and error callbacks. In the above example, we perform some basic actions on our model after loading it. Commonly used Model methods include:

- `fetch`: fetches the model's data from its REST service using a `GET` request.  
- `get`: gets a named attribute from the model.  
- `set`: sets values for named model attributes (without saving to the server).
- `save`: sets attributes, and then saves the model data to the server using a `PUT` request.
- `destroy`: decomissions the model, and removes it from the server using a `DELETE` request.

## Using Collections

Collections handle the loading and management of a list of models. We must first define a Model class for the list's items, and then attach that model class to a managing collection:

	var MuppetModel = Backbone.Model.extend({
		defaults: {
			id: null,
			name: null,
			occupation: null
		}
	});
	
	var MuppetCollection = Backbone.Collection.extend({
		url: '/muppets'
		model: MuppetModel
	});

In the above example, `MuppetsCollection` will load data from the `"/muppets"` list endpoint. It will then construct the loaded data into a list of `MuppetModel` instances.

To load our collection of Muppet models, we build a collection instance and then call `fetch`:

	var muppets = new MuppetCollection();
	
	muppets.fetch().then(function() {
		console.log(muppets.length); // >> length: 1
	});

Easy, right? However, there's a problem here: our collection only created a single model. We were *supposed* to get back a list of two items. Let's review again what the `GET /muppets/` service returns…

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
	
We can see that this list data does indeed contain two records, however our collection only created one model instance. Why? The reason is beacuse Collections are derived from Arrays, while Models are derived from Objects. In this case, our root data structure is an *Object* (not an Array), so our collection tried to parse the returned data directly into a model.

What we really want is for our collection to populate its list from the `"muppets"` array property of the returned data object. To address this, we simply add a `parse` method onto our collection:

	var MuppetCollection = Backbone.Collection.extend({
		url: '/muppets'
		model: MuppetModel,
		
		parse: function(data) {
			return data.muppets;
		}
	});

A Collection's `parse` method recieves raw data loaded from REST services, and may return a specific portion of that data to be loaded into the collection. In Backbone, both Models and Collections support the definition of a `parse` method. Using `parse` is very useful for reconciling minor differences between your API design and your front-end application architecture (which often times won't map one-to-one, and that's okay!).

With the `parse` method in place, the following now happens upon fetching the collection:

	var muppets = new MuppetCollection();
	
	muppets.fetch().then(function() {
		console.log(muppets.length); // >> length: 2
	});
	
	muppets.get(1); // >> Returns the "Kermit" model, by id reference
	muppets.get(2); // >> Returns the "Gonzo" model, by id reference
	muppets.at(0); // >> Returns the "Kermit" model, by index
	muppets.findWhere({name: 'Gonzo'}); // >> returns the "Gonzo" model
	
Success! The returned list of Muppets were parsed as expected into a collection of `MuppetModel` instances, and the Collection provided some basic methods for querying them. Commonly used Collection methods include:

- `fetch`: fetches the collection's data from its REST service using a `GET` request.
- `create`: adds a new model into the collection, and creates it at the API via `POST`.
- `add`: adds a new model into the collection without telling the API.
- `remove`: removes a model from the collection without telling the API.
- `get`: gets a model from the collection by id reference.
- `at`: gets a model from the collection by index.
- `find`: finds all records matching a specific search criteria.

## Backbone CRUD

Create, Read, Update, and Destroy are the four major data interactions that an application must manage. Backbone Models and Collections work closely together to delegate these roles. Infact, the relationship of Models and Collections (not so coincidentally) mirrors the design of a RESTful API. To review:

	var MuppetModel = Backbone.Model.extend({
	    defaults: {
	        id: null,
	        name: null,
	        occupation: null
	    }
	});
	
	var MuppetCollection = Backbone.Collection.extend({
	    url: '/muppets'
	    model: MuppetModel
	});

Notice above that the model class does NOT define a `url` endpoint to sync with. This is because models within a collection will automatically construct their `url` reference as `"[collection.url]/[model.id]"`. This means that after fetching the collection, our Kermit model (with an id of "1") will automatically be configured to sync with a url of `"/muppets/1"`.

### Create

Use `Collection.create` to `POST` new data to a list endpoint. The API should return complete data for the new database record, including its assigned id. The new model is created immediately within the front-end collection.

	muppetsList.create({name: 'Piggy', occupation: 'fashionista'});

### Read

Use `Collection.fetch` or `Model.fetch` to load data via `GET`. For models, you'll generally only need to call `fetch` for models without a parent collection.

	kermit.fetch();
	muppetsList.fetch();

### Update

Use `Model.save` to `PUT` updated data for a model. The model's complete data is sent to the API.

	kermit.save('occupation', 'being awesome');

### Destroy

Use `Model.destroy` to `DELETE` a model instance. The model will remove itself from any parent collection, and issue a `DELETE` request to the API.

	kermit.destroy();

### Patch

Some API designs may also support using the `PATCH` method to perform partial model updates (where only modified data attributes are sent to the API). This design is less common, however can be achieved in Backbone by calling `Model.save` and passing a `{patch: true}` option.

	kermit.save('occupation', 'being awesome', {patch: true});


## Binding Events

Backbone also provides a best-of-class Events framework. The major differentiator of Backbone Events is the support for *context passing*, or, specifing what `this` refers to when an event handler is triggered:

	target.on(event, handler, context)
	target.off(event, handler, context)
	target.trigger(event)

The other key feature of the Backbone Events framework are the inversion-of-control event binders:

	this.listenTo(target, event, handler)
	this.stopListening()

These reverse event binders make life easier by quickly releasing all of an object's bound events, thus aiding memory management. As a general rule, objects with a shorter lifespan should listen to objects with a longer lifespan, and clean up their own event references when deprecated.

### Model & Collection Events

You may bind event handlers onto any model or collection (optionally passing in a handler context):

	kermit.on('change', function() {
		// do stuff...
	}, this);

Commonly tracked Model events include:

- `"change"`: triggered when the value of any model attribute changes.
- `"change:[attribute]"`: triggered when the value of the named *attribute* changes.
- `sync`: called when the model completes a data exchange with the API.

Commonly tracked Collection events include:

- `"add"`: triggered when a model is added to the collection.
- `"remove"`: triggered when a model is removed from the collection.
- `"reset"`: triggered when the collection is purged with a hard reset.
- `"sync"`: triggered when the collection completes a data exchange with the API.
- `[model event]`: all child model events are proxied by their parent collection.

Review Backbone's [catalog of built-in events](http://backbonejs.org/#Events-catalog) for all available event triggers.

## Using Views

Views create linkage between data sources (Models and Collections) and display elements. As a general rule, Views should map one-to-one with each data source present – meaning  that one view controller is created for each collection and each model represented within the display.

### Creating a view's container element

All Backbone Views are attached to a *container element*, or a main HTML document element into which all nested display and behavior is allocated. A common approach while getting started is to bind your major views onto pre-defined elements within your HTML Document Object Model (hereforth referred to as the "DOM"). For example:

	<ul id="muppets-list"></ul>
	
	<script>
	var MuppetsListView = Backbone.View.extend({
		el: '#muppets-list'
	});
	</script>

In the above example, we define a new Backbone View class, and reference `"#muppets-list"` as its target `el`, or *element*. This element reference is a selector string that gets resolved into a DOM element reference.

Another common workflow is to have Backbone create a new view container element for us. To do this, we simply provide a `tagName` and an optional `className` for the container element to create. This approach is commonly used when generating list items for a collection:

	var MuppetsListItemView = Backbone.View.extend({
		tagName: 'li',
		className: 'muppet'
	});
	
Note that these two view element patterns (selected/created) are commonly used together, especially when rendering lists.

Once a view class is defined, we'll next need to instance it:

	var MuppetsListView = Backbone.View.extend({
		el: '#muppets-list'
	});
	
	var muppetsList = new MuppetsListView();
	
	muppetsList.$el.append('<li>Hello World</li>');
	
When a view is instanced, Backbone will configure an `$el` property for us – this is a jQuery object wrapping the view's attached container element. This reference provides a convenient way to work with the container element using the jQuery API.

Backbone also encourages efficient DOM practices using jQuery. Rather than performing large and expensive operations across the entire HTML document, Backbone Views provide a `$` method used for performing jQuery operations locally within the view's container element. For example:

	muppetsList.$('li'); // << Finds all "li" tags within the view's container.

Under the hood, using `view.$('…')` is synonymous to calling `view.$el.find('…')`. These localized queries greatly cut down the overhead of superflous DOM operations.

### Attaching a view's data source

Once a view class has defined its relationship with the DOM, we then construct instances of the view, and provide each instance with a data source. When constructing a view instance, we may provide it with a `model` or `collection` reference.

Attaching a Model to a View (see final two lines):

	var KermitModel = Backbone.Model.extend({
		url: '/muppets/1'
		defaults: { . . . }
	});
	
	var MuppetsListItemView = Backbone.View.extend({
		tagName: 'li',
		className: 'muppet',
		
		initialize: function() {
			console.log(this.model); // << KermitModel!!
		}
	});
	
	// Create Model and View instances:
	var kermitModel = new KermitModel();
	var kermitView = new MuppetsListItemView({model: kermitModel});

Attaching a Collection to a View (see final two lines):

	var MuppetsModel = Backbone.Model.extend({ . . . });
	
	var MuppetsCollection = Backbone.Collection.extend({
		model: MuppetsModel,
		url: '/muppets'
	});
	
	var MuppetsListView = Backbone.View.extend({
		el: '#muppets-list',
		
		initialize: function() {
			console.log(this.collection); // << MuppetsCollection!!
		}
	});
	
	// Create Collection and View instances:
	var muppetsList = new MuppetsCollection();
	var muppetsView = new MuppetsListView({collection: muppetsList});

In the above examples, a data source is provideded to each view as a *constructor option*. The provided data source is attached directly to its view instance, allowing the view to reference that source as `this.model` or `this.collection`. It will be the view's job to render this data source into its DOM element, and pass user input data from the DOM back into this data source.

Also notice the `initialize` methods in the above examples. An `initialize` method is called on each new Backbone component at the time it's created, giving you an opportunity to configure the new object. All Backbone components provide `initialize` methods.

### Rendering a View

Among the primary responsibility of a view is to render data from its data source into its bound DOM element. Now, Backbone is notoriously unopinionated about this task (for better or worse), and provides no fixture for translating a data source into display-ready HTML. That's for us to define.

However, Backbone does dictate a general workflow for *where* and *when* rendering occurs:

1. A views defines a `render` method. This method generates HTML from its data source, and installs that markup into the view's container element.
2. A view binds event listeners to its model. Any changes to the model should trigger the view to re-render.

A simple implementation:

	<div id="#kermit-view"></div>
	
	<script>
	var KermitModel = Backbone.Model.extend({
		url: '/muppets/1',
		defaults: {
			name: '',
			occupation: ''
		}
	});
	
	ver KermitView = Backbone.View.extend({
		el: '#kermit-view',
		
		initialize: function() {
			this.listenTo(this.model, 'sync change', this.render);
			this.model.fetch();
			this.render();
		},
		
		render: function() {
			var html = '<b>Name:</b> ' + this.model.get('name');
			html += ', occupation: ' + this.model.get('occupation');
			this.$el.html(html);
			return this;
		}
	});	
	
	var kermit = new KermitModel();
	var kermitView = new KermitView({model: kermit});
	</script>

In the above example, a simple render cycle is formed:

1. The view's `render` method translates its bound model into display-ready HTML. The rendered HTML is inserted into the view's container element. A `render` method normally returns a reference to the view for method-chaining purposes.
2. The view's `initialize` method binds event listeners to the model for `sync` and `change` events. Either of these model events will trigger the view to re-render. The view then fetches (loads) its model, and renders its initial appearance.

The core focus of this workflow is *event-driven behavior*. View rendering should NOT be a direct result of user interactions or application behaviors. Manually calling `render` is prone to errors and inconsistancies. Instead, rendering should be a simple union of data and display: when the data changes, the display updates.

### Rendering with templates

The most efficient way to render model data into the DOM is by creating a template that parses data attributes into a string of HTML markup.

There are numerous JavaScript template libraries available… for some reason, [Handlebars]() is incredibly popular among Backbone developers. Personally, I find this odd considering that Underscore has a perfectly capable template renderer built in, and is already included in all Backbone apps. For this primer, we'll be focusing on the Underscore template renderer.

First, we need to define raw-text HTML for a template's markup. There's a quick an easy trick for hiding raw text within HTML documents: include the raw text in a `<script>` tag with a bogus script type. For example:

	<script type="text/template" id="muppet-item-tmpl">
		<p><b><%= name %></b></p>
		<p>Job: <i><%= occupation %></i></p>
	</script>

The above script block uses a bogus script type, so will be ignored by HTML renderers. However, we can still find the ignored element within the DOM, extract its contents, and parse that into a template function. To create our template, we do this:

	var tmplText = $('#muppet-item-tmpl').html();
	var muppetTmpl = _.template(tmplText);
	
The Underscore template method parses our raw text into a reusable *template function*. This template function may be called repeatedly with different data objects, and will generate a unique HTML string for each. For example, let's render Kermit's data using this template function:

	var kermitHtml = muppetTmpl({
		id: 1,
		name: "Kermit",
		occupation: "being green"
	});
	
	// Resulting HTML:
	<p><b>Kermit</b></p>
	<p>Job: <i>being green</i></p>
	<button class="remove">x</button>
	
Be mindful that the most expensive operation involved with template rendering is parsing that initial template function. Therefor, it's best to retain a parsed template function for future use rather than reparse the template function every time you need to render a view.

### Binding DOM events

Next, a view must capture user input events – whether those are button clicks, typing into an input field, or changing a select menu.

Views provide a convenient and efficient way of delegating user interface events into the view controller. Backbone offers a declarative events interface of setting up user interface bindings.

	var KermitModel = Backbone.Model.extend({
		url: '/muppets/1',
		defaults: { . . . }
	});
	
	ver KermitView = Backbone.View.extend({
		el: '#kermit-view',
		
		events: {
			'click .update': 'onUpdate'
		},
		
		onUpdate: function(evt) {
		
		}
	});
	
	var kermit = new KermitModel();
	var kermitView = new KermitView({model: kermit});
	
### Rendering an application view

Now its time to make our application render out a list of muppets. Here we go:

	<ul id="muppets-list"></ul>
	
	<script type="text/template" id="muppet-item-tmpl">
		<p><b><%= name %></b></p>
		<p>Job: <i><%= occupation %></i></p>
	</script>
	
	<script>
		// 1. Model class for each Muppet:
		var MuppetModel = Backbone.Model.extend({
			defaults: {
				id: null,
				name: null,
				occupation: null
			}
		});
		
		// 2. Collection class for Muppets list endpoint:
		var MuppetCollection = Backbone.Collection.extend({
			model: MuppetModel,
			url: '/muppets'
		});
		
		// 3. View class for displaying each muppet list item:
		var MuppetsListItemView = Backbone.View.extend({
			tagName: 'li',
			className: 'muppet',
			template: _.template($('#muppet-item-tmpl').html()),
			
			render: function() {
				var data = this.model.toJSON();
				var html = this.template(data);
				this.$el.html(html);
			}
		});
		
		// 4. View class for rendering the list of all muppets:
		var MuppetsListView = Backbone.View.extend({
			el: '#muppets-list',
			
			initialize: function() {
				this.listenTo(this.collection, 'sync', this.render);
			},
			
			render: function() {
				this.$el.empty();
				this.collection.each(function(model) {
					var item = new MuppetsListItemView({model: model});
					item.render();
					this.$el.append(item.$el);
				}, this);
			}
		});
		
		// 5. Create new list collection, list view, and then fetch list data:
		var muppetsList = new MuppetsCollection();
		var muppetsView = new MuppetsListView({collection: muppetsList});
		muppetsList.fetch();
	</script>

View management is the least regulated component of Backbone, and yet is –ironically– among the most uniquely disciplined roles in front-end engineering. While `Backbone.View` provides some useful low-level utility features, it provides very few high-level workflow features. As a result, major Backbone extenstions including [Marionette]() and [Thorax]() have become popular.

...For this primer, we'll focus on functional view programming using only the native Backbone View component.