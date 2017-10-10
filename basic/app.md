## App

A Webix Jet app is single-page and is divided into views. The application structure consists of the following parts:

- the *index.html* file that is a start page of the app;
- *sources/myapp.js* that contains the configuration of the app, you can give any name to this file;
- the *sources/views* folder that contains elements of the interface;
- the *sources/models* folder that contains data modules.

App represents an application or an application module. It is used to group views and modules which implement some specific scenario. Later you will be able to combine separate app modules in high-level apps.

### The Syntax of Creation

An app module is created as a new instance of JetApp class. You are to pass an object with your app configuration like the app name, version, etc as the parameter.

~~~js
/* myapp.js */
import {JetApp} from "webix-jet";

var app = new JetApp({
    start:"/top/layout"
}).render(); //mandatory!
~~~

After you specify the app configuration, you must call the **render** method to build the UI.

You can open the needed URL and the UI will be rendered from the URL elements. The app splits the URL into parts, finds the corresponding files in the **views** folder and creates an interface by combining UI modules from those files.

In the app config, for example, you can set the mode in which the app will work:

```js
var app = new JetApp({
	mode:"readonly",  //application wide configuration
	start:"/top/layout"
});
```

Later in the code, you can do some actions according to the mode:

```js
if (this.app.config.readonly){
	//...
}
```

New Webix Jet has four types of routers. You should specify the preferred router in the app configuration as well. The default router is *HashRouter*. If you don't want to display the hashbang in the URL, you can change the route to type to URLRouter:

```js
import {UrlRouter} from "webix-jet"
var app = new JetApp({
	router: UrlRouter,
    start:"/top/layout"
});
```

The rest of the routers are *StoreRouter* and *EmptyRouter*. For more information about routers, [go to the dedicated section](../details/routers.md) of the advanced chapter.