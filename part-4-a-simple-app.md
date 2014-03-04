# Backbone, The Primer: A Simple App #
This is the fourth of four posts that guest author Greg MacWilliam has put together for Data Community DC: _Backbone, The Primer_. For more details about the API and an overview, please go back to the Introduction. For more on Models and Collections please review part two of this primer. A comprehensive look at Views and event bindings was in part three.

## A simple REST application

Now its time to put it all together. Let's breakdown a complete RESTful application that performs all CRUD methods with our API.

### 1. The DOM

The first step in setting up any small application is to establish a simple interface for managing the tasks you intend to perform. Here, we've establish a `"muppets-app"` container element with a list (`<ul>`) for displaying all Muppet items, and a simple input form for defining new muppets.

Down below, a template is defined for rendering individual list items. Note that our list item template includes a "remove" button for clearing the item from the list.

Finally, we'll include our application's JavaScript as an external script. We can assume that all further example code will be included in `muppet-app.js`.

	<div id="muppets-app">
		<ul class="muppets-list"></ul>
		
		<div class="muppet-create">
			<b>Add a Muppet</b>
			<fieldset>
				<label for="muppet-name">Name:</label>
				<input id="muppet-name" type="text">
			</fieldset>
			<fieldset>
				<label for="muppet-job">Job:</label>
				<input id="muppet-job" type="text">
			</fieldset>
			<button class="create">Create Muppet!</button>
		</div>
	</div>
	
	<script type="text/template" id="muppet-item-tmpl">
		<p><a href="/muppets/<%= id %>"><%= name %></a></p>
		<p>Job: <i><%= occupation %></i></p>
		<button class="remove">x</button>
	</script>
	
	<script src="muppet-app.js"></script>
	
### 2. The Model and Collection

Now in `"muppet-app.js"`, the first structures we'll define is the Model class for individual list items, and the Collection class for managing a list of models. The Collection class is configured with the URL of our API endpoint.

	// Model class for each Muppet item
	var MuppetModel = Backbone.Model.extend({
		defaults: {
			id: null,
			name: null,
			occupation: null
		}
	});
	
	// Collection class for the Muppets list endpoint
	var MuppetCollection = Backbone.Collection.extend({
		model: MuppetModel,
		url: '/muppets'
	});
	
### 3. A List Item View

The first View class that we'll want to define is for individual list items. This class will generate its own `<li>` container element, and will render itself with our list item template. That template function is being generated once, and then stored as a member of the class. All instances of this class will utilize that one parsed template function.

This view also configures an event for mapping clicks on the "remove" button to its model's `destroy` method (which will remove the model from its parent collection, and then dispatch a `DELETE` request from the model to the API).

	// View class for displaying each muppet list item
	var MuppetsListItemView = Backbone.View.extend({
		tagName: 'li',
		className: 'muppet',
		template: _.template($('#muppet-item-tmpl').html()),
		
		render: function() {
			var html = this.template(this.model.toJSON());
			this.$el.html(html);
			return this;
		},
		
		events: {
			'click .remove': 'onRemove'
		},
		
		onRemove: function() {
			this.model.destroy();
		}
	});
	
### 4. A List View

Now we need a view class for rendering out lists of items, and capturing input from the "create" form.

This view binds a listener to its collection that will trigger the view to render whenever the collection finishes syncing with the API. That will force our view to re-render when initial data is loaded, or when items are created or destroyed.

This view renders a list item for each model in its collection. It first finds and empties its list container (`"ul.muppets-list"`), and then loops through its collection, building a new list item view for each model in the collection.

Lastly, this view configures an event that maps clicks on the "create" button to collecting form input, and creating a new collection item based on the input data.

	// View class for rendering the list of all muppets
	var MuppetsListView = Backbone.View.extend({
		el: '#muppets-app',
		
		initialize: function() {
			this.listenTo(this.collection, 'sync', this.render);
		},
		
		render: function() {
			var $list = this.$('ul.muppets-list').empty();
			
			this.collection.each(function(model) {
				var item = new MuppetsListItemView({model: model});
				$list.append(item.render().$el);
			}, this);
			
			return this;
		},
		
		events: {
			'click .create': 'onCreate'
		},
		
		onCreate: function() {
			var $name = this.$('#muppet-name');
			var $job = this.$('#muppet-job');
			
			if ($name.val()) {
				this.collection.create({
					name: $name.val(),
					occupation: $job.val()
				});
				
				$name.val('');
				$job.val('');
			}
		}
	});
	
### 5. Instantiation

Finally, we need to build instances of our components. We'll construct a collection instance to load data, and then construct a list view instance to display it. When our application components are all configured, all that's left to do is tell the collection to `fetch` for data!

	// Create a new list collection, a list view, and then fetch list data:
	var muppetsList = new MuppetsCollection();
	var muppetsView = new MuppetsListView({collection: muppetsList});
	muppetsList.fetch();

## Getting View Support

View management is by far the least regulated component of Backbone, and yet is –ironically– among the most uniquely disciplined roles in front-end engineering. While `Backbone.View` provides some very useful low-level utility features, it provides few high-level workflow features. As a result, major Backbone extenstions including [Marionette](http://marionettejs.com/) and [LayoutManager](https://github.com/tbranyen/backbone.layoutmanager) have become popular. Also see [ContainerView](https://github.com/gmac/backbone.containerview) for a minimalist extension of core Backbone.View features.

Thanks for reading. That's Backbone in a nutshell.

### About the Author ###
Greg is primarily focused on web front-end with JavaScript, specializing in large-scale web applications and single-page applications. He is also experienced with HTML5, CSS3, SCSS, responsive design practices, and visualizing large data sets using SVG, Canvas, and major web mapping frameworks. He is also fluent with RESTful API design, Django, Rails, Node, and some WordPress experience.

Current side projects include EpoxyJS (data binding extension for Backbone), ConstellationJS (a grid-based geometry toolkit), and the HTML5/Django-based Lassie Constellation admin system.