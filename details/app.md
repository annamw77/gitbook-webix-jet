# <span id="contents">JetApp API</span>

Here you can find the list of all the **JetApp** methods, that you can make use of.

| Method | Use to  |
|--------|---------|
| [attachEvent(event, handler)](#attach)   | attach an event |
| [callEvent(event)](#call)                | call an event |
| [detachEvent(event)](#detach)            | manually detach an event listener |
| [getService(name,handler)](#get_service) | access a service by its name |
| [render(container)](#render)             | render the app or the app module |
| [setService(name)](#set_service)         | set a service |
| [show(url)](#show)                       | rebuild the app or app module according to the new URL |
| [use(plugin, config)](#use)              | switch on a plugin |

### [<span id="attach">app.attachEvent("event:name", handler) &uarr;</span>](#contents)

Use this method to attach a custom event, for example, from a Jet view class:

```js
// views/form.js
import {JetView} from "webix-jet";

export default class FormView extends JetView{
    init(){
        this.app.attachEvent("save:form", function(){
            this.show("aftersave");
        });
    }
}
```

Events can be attached both in the app file and in view modules. Yet, if you attach an event listener with this method, you had better to [detach it manually &darr;](#detach) to eliminate a possibility of memory leaks.

You can also attach an inner Jet event. For instance, from app:

```js
// myapp.js
...
app.attachEvent("app:guard", function(url, view, nav){
    if (url.indexOf("/blocked") !== -1)
        nav.redirect="/somewhere/else";
});
...
```

For more details on events, read ["Events and Methods"](events.md) and ["Inner Events and Error Handling"](inner_events.md).

### [<span id="call">app.callEvent("event:name") &uarr;</span>](#contents)

Use this method to call a custom event:

```js
// views/data.js
import {JetView} from "webix-jet";

export default class DataView extends JetView{
    config(){
        return {
            view:"button", click:() => {
                this.app.callEvent("save:form");
            }
        }
    }
}
```

Normally, inner events are called automatically, so there is no need to use **callEvent** for them.

For more details on events, read ["Events and Methods"](events.md) and ["Inner Events and Error Handling"](inner_events.md).

### [<span id="detach">app.detachEvent(event) &uarr;</span>](#contents)

Use this method to detach event listeners added by **attachEvent** from Jet view classes. You should do this, because the lifetime of such an event listener is longer than the lifetime of the Jet view. So if the view is destroyed, but the event listener isn't detached, this may cause memory leaks, especially in older browsers.

To detach an event, call **app.detachEvent** when the view that attached the event is destroyed:

```js
// views/form.js
import {JetView} from "webix-jet";

export default class FormView extends JetView{
    init(){
        this.app.attachEvent("save:form", function(){
            this.show("aftersave");
        });
    }
    destroy(){
        this.app.detachEvent("save:form");
    }
}
```

For more details on events, read ["Events and Methods"](events.md) and ["Inner Events and Error Handling"](inner_events.md).

### [<span id="get_service">app.getService(name) &uarr;</span>](#contents)

The method returns a service by its name, passed to the method as a parameter. Call this method to use a service:

```js
// views/form.js
import {JetView} from "webix-jet";
import {getData} from "models/records";

export default class FormView extends JetView{
    config(){
        return {
            view:"form", elements:[
                { view:"text", name:"name" }
            ]
        };
    }
    init(){
        var id = this.app.getService("masterTree").getSelected();
        this.getRoot().setValues(getData(id));
    }
}
```

You can read more about services in the ["Services"](services.md) chapter.

### [<span id="render">app.render() &uarr;</span>](#contents)

The **render** method builds the UI of the application. If called without any parameters, it just renders the UI inside the page according to the start URL, specified in the app configuration.

```js
// myapp.js
...
app.render();
```

But if you want to render the app inside a container, you can pass the string parameter to it with the ID of the container:

```js
// myapp.js
...
app.render("mybox");
```

### [<span id="set_service">app.setService(name,handler) &uarr;</span>](#contents)

The method initializes a service for view communication.

```js
// views/tree.js
import {JetView} from "webix-jet";

export default class treeView extends JetView{
    config(){
        return { view:"tree" };
    }
    init() {
        this.app.setService("masterTree", {
            getSelected : () => this.getRoot().getSelectedId();
        })
    }
}
```

**this** refers to the instance of the *treeView* class if it is used in an *arrow function*<sup><a href="#myfootnote1" id="origin1">1</a></sup>.

You can read more about services in the ["Services"](services.md) chapter.

### [<span id="show">app.show(url) &uarr;</span>](#contents)

The **show** method is used to change the app interface. This method rebuilds the whole UI of the app according to the URL passed as a parameter:

```js
// views/some.js
...
app.show("/demo/details")
```

For more info about showing UI components, visit the ["Navigation"](navigation.md) chapter.

### [<span id="use">app.use(plugin, config) &uarr;</span>](#contents)

The **use** method is used to switch on plugins. The method takes two parameters:

- the name of the plugin 
- the plugin configuration

```js
// myapp.js
import session from "models/session";
...
app.use(plugins.User, { model: session });
```

For more details, go to the ["Plugins"](plugins.md) chapter.

<!-- footnotes -->
- - -
<a id="myfootnote1" href="#origin1">1 &uarr;</a>:
To read more about how to reference apps and view classes, go to ["Referencing views"](../detailed/referencing.md).