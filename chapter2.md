In the previous part of the guide we’ve described the client side part of the app and basics of building interface from modules. Now we will tell you about data saving on the server side and a more extended usage of views.


## Loading and saving data on the server side

All data related logic should be stored in separate modules - models. A model is created for each particular data category (users, books, etc.). It guarantees code safety of the whole app in case you need to change something in a single model. 

In order to load and save data with the help of REST API, we should initialize a data collection in the following way:

```js
// models/records.js

define([],function(){

	var collection = new webix.DataCollection({ 
        url:"rest->/records.php", 
        save:"rest->/records.php" 
    });
	return {
		data: collection
	};
});
```

where:
- url:"rest->/records.php" defines the URL of the data loading script
- save:"rest->/records.php" defines the URL of the data saving script

Those parameters allow applying changes made on the client side to the database on the server

The described functionality is presented in the following [demo](https://github.com/webix-hub/jet-demos/tree/04_server).


## View Handlers


A view module returns an object which can have a set of special properties:

```js
define([], function(){
    return {
        $ui:{ /*ui config*/},
        $oninit:function(view, $scope){ /*after creating*/},
        $onurlchange:function(config, url, $scope){ /*after navigation*/},
        $ondestroy:function(){ /*before destroy*/}
    }
});    
```

Let’s consider each property in detail.

**$ui**

The ui property gives the description of Webix interface. It can include any possible content of a component (settings, events, data, templates).

Our interface can also include other views. Such a structure is called composition:

```js
// views/top.js

define(["app"], function(app){
    var ui =  {
        cols:[
            { view:"menu", id:"top:menu"},
            { $subview:true}
        ]
    };
    
    return {
        $ui: ui
    };
});
```

**$oninit**

This method is called each time when a view is created. It’s used to load data into components and to set event handlers. The handler gets two parameters: the object of a newly created view and the current scope.

```js
// views/data.js

define(["models/records"],function(records){
    return {
        $ui: {
            view:"datatable"
        },
        $oninit:function(view,$scope){
            view.parse(records.data);
        }
    }
});
```

The important note: when the url is changed, the oninit method won’t be called for existing views. For example, if we go from /top/data to /top/start, $oninit will be called for the "start" only. As the top view has already been rendered, $oninit won’t be called for it.

**$onurlchange**

The method is called when the "local" url is changed (after going to the next page). It’s used to restore the view’s state by means of the url’s parameters.

```js
// views/top.js

define([
	"app"
],function(app){
    var ui =  {
        cols:[
            { view:"menu", id:"top:menu"},
            { $subview:true}
        ]
    };
    return {
        $ui: ui,
        $onurlchange:function(config, url, $scope){
            $$("top:menu").select(url[0].page);
        }
    }
});
```

where:

- config - parameters of the url
- url - contains the names of segments and all parameters of the url
- $scope - the current scope

**$ondestroy**

The method is called each time when a view is destroyed. It can be used to destroy temporary objects (such as popups and event handlers) and prevent memory leaks.

```js
// views/data.js

define([
    "models/records"
], function(records){
    var popup, eventid;
    
	return {
        $ui: {
            view:"datatable", editable:true
        },
        $oninit:function(view,$scope){
            popup = webix.ui({ 
                view:"popup", 
                body:"Data is updated"
            });
            eventid = records.data.attachEvent("onDateUpdate", function(){
                popup.show();
            });
            view.parse(records.data);
        },
        $ondestroy:function(){
            popup.destructor();
            records.detachEvent(eventid);
        }
    }
});
```

Notice that the change of the url won’t call $ondestroy for the views that present in the new url. For example, if we go from the page /top/data to the page /top/start, the $ondestroy method will be called for the data view only. 

The main use case of the ondestroy handler is removing of any unnecessary UI or unused event handlers. There are some other, simpler ways to handle this use-case though.It’s more handy to use scope for both initializing and destructing popups and event handlers.

You can check the [demo](https://github.com/webix-hub/jet-demos/tree/05_view_handlers) showing how handlers are used in an application.


## What is scope?

As you can see, the *$oninit* and *$onurlchange* properties contain functions that take the *$scope* parameter as an argument. So, we need to explain the term scope.

Scope is an object that keeps the state of a part of an interface. Scope is automatically created for the whole application. What is more, scope is created for each subview which is presented in the code.

For example, for an app which shows the path */top/data* we will have 2 scopes. One for top view and one for data view.


## Using scope

Scope helps to create temporary elements, e.g. popups which are automatically destroyed together with the current view. It also allows making event handlers which are detached when the current view is no longer needed and destroyed.

For example:

```js
define([
       "models/records"
],function(records){
    var ui = {
		view:"datatable"
    };
    
    return {
        $ui :ui,
        $oninit:function(view){
            var popup = ({ 
                view:"popup", 
                body:"Data is updated"
            });
            records.data.attachEvent("onDataUpdate", function(){
				popup.show();
            });
            view.parse(records.data);
        }
    }
});
```

The above code defines that after updating a record in datatable a popup "Data is updated" appears. This message will try to appear, even when another view will be loaded instead of the datatable, as event handler never detaches by itself. To solve this issue, we can use the *$scope.on* method:

```js
$oninit:function(view, $scope){
    $scope.on(records.data, "onDataUpdate", function(){
		popup.show();
	});
}
```

Due to attaching event by the *$scope.on* method, the handler of the onDateUpdate event will be detached when a new view will be loaded on the page.

In order to create a popup that will be destroyed together with the current view, the *$scope.ui()* method can be used instead of webix.ui(). Thus, the code below

```js
define([
    "models/records"
],function(records){
    var popup;
	return {
        $ui: {
            view:"datatable"
        },
        $oninit:function(view){
            popup = webix.ui({ view:"popup"});
        },
	    $ondestroy:function(){
		    popup.destructor();
        }
    }
});
```

can be redefined as:

```js
$oninit:function(view,$scope){
    var popup = $scope.ui({ view:"popup"});
});
//no need to define $ondestroy
```


## In-app navigation

Navigation between elements of an app is implemented by means of changing the url
of the page. 

1) direct url navigation ( A tag or document.location )

```html
<a href="#!/top/data">Data</a>
```

Direct url can be specified within Webix menu as value of *href* parameter: 

~~~js
// views/top.js

var menu = {
    view:"menu",
	data:[
		{ value:"DashBoard", href:"#!/top/start" },
		{ value:"Data",	href:"#!/top/data" }
	]
};
~~~

2) the *app.show* method is applied to the whole application:

```js
// views/top.js

define([
	"app"
],function(app){
    var ui = {
        view:"menu", 
        select:true,
        data:[
            { id:"start", value:"Dashboard"},
            { id:"data", value:"Data"}
        ],
        on:{
		    onAfterSelect:function(id){
			    app.show("/top/"+id);
		    }
	    }
    };
    return {
		$ui: ui
	};
});
```

In the above code it’s stated that on clicking an item in the menu, we’ll get its id in the url of the page, e.g.: *#!/top/data*. Each time when we select some other item, the whole app will be rerendered and the url will change. 

3) the *$scope.show* method

This way is more convenient, as the method allows reloading just the part of the app. There are two variants of loading data with the scope.show method:

- loading a subview of the same level (sibling):

```js
// views/start.js

this.$scope.show("data");
```

The current subview will be changed to "data". For example: the path *index.html#!/top/start* will be reloaded as *#!/top/data*

- loading a child subview

```js
//views/start.js

this.$scope.show("./news"); 
```

In this case the *news* subview will be loaded inside of the "start" view and the path *#!/top/start* will be changed into *#!/top/start/news*.


## Working with menus

It is quite common to have some kind of menu and subview next to it. When item in menu selected we can navigate subview to the selected page

```js
{
	view:"menu", id:"top:menu",
	select:true,
	data:[
	    { value:"DashBoard", id:"start", href:"#!/top/start" },
		{ value:"Data", id:"data", href:"#!/top/data" }
	]
}
```

The above code is enough, but we have a problem. When we are entering some url manually in the address bar, the correct page will be loaded, but the related item in the list won't be selected automatically. 

We can use the onUrlChange handler:

```js
return {
	$ui:ui,
    $onurlchange:function(config, url, $scope){
		$$("top:menu").select(url[0].page);
	}
}
```

or we can use the *menu* plugin. 

A plugin is an independent module used to extend the app’s functionality. 

The *menu* plugin can be used for highlighting a menu item or an element when a page is changed. To enable menu, you need to include the plugin in the *app.js* file in the following way:

```js
define( 
    "libs/webix-mvc-core/plugins/menu"
],function(menu){
    // after the description of the app configuration
    app.use(menu);
});
```

The plugin is used in a view by means of the *$menu* parameter which takes the id of the menu view as its value. The id of the menu view is specified in the description of the app’s interface.

```js
return {
	$ui:ui,
	$menu:"top:menu"
}
```








