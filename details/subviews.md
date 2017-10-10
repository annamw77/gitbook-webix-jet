## Views

After reading the first chapter of this guide, you are familiar with the concept of _view_. Now it's time to find out all the ways of creating views. You can create views in three ways.

### 1. Simple Views

Views can be created as objects.

+ simple

* **Disadvantages**
 
- static
- had no init and other handlers

Here's a simple template object:

```js
/* views/dash.js */
const DashView = { template:"Dash" };
```

export default {
	view:"list"
}

### 2. Object "Factory Pattern"

View objects can also be returned by a factory function.

+ still simple
+ dynamic
- had no init

Here's a simple template view returned by a factory:

```js
/* views/details.js */
const DetailsView = () => ({ template:"App" });
```

export default () => {
	var data = [];
	for (var i=0; i<10; i++) data.push({ value:i });

	return {
		view:"list", options:data
	}
}

export const Form = () => ({
    view:"form", elements:[
        { view:"text", name:"email", required:true, label:"Email" },
        { view:"button", value:"save", click:() => this.show("Details") }
    ]
})

something like inheritance

const DetailsView = (n) => ({ template:"App", data:mydata });
...
export DetailsView(200)

### 3. Class Views

Views can be defined as JS6 classes.

#### Advantages of classes

+ dynamic
+ has init and other methods that can be redefined by users

* Multiple instances

All instances have their individual inner states. E.g. if you use the same Toolbar class for to add identical toolbars, there are two instances of a Toolbar class and the toolbars will behave independently.

* **this**

The **this** pointer is a reference to the object inside methods and handlers.

* Inheritance

With JS6 classes, inheritance is closer to classic OOP and the syntax is nicer. Inheritance can help you reuse old components for creating slightly different ones. For example, if you already have a toolbar and want to create a similar one, but with one additional button, define a new class and inherite from the old toolbar.

* Data

Data collections can be created inside a method if you need to process the data before loading and don't want to create additional modules for that.

#### JetView Methods

View classes inherit from **JetView**. Webix UI events are implemented through class methods. Here are the methods that you can define while defining your class views:

* **config\(ui\)**

This method returns the initial UI configuration of a view, all its contents.

```js
/* views/toolbar.js */
class ToolbarView extends JetView{
    config(){
        return { 
            view:"toolbar", elements:[
                { view:"label", label:"Demo" },
                { view:"segmented", localId:"control", options:["Details", "Dash"], click:function(){
                    this.$scope.app.show("/Demo/"+this.getValue())
                }}
            ]
        };
    }
}
```

The method receives one parameter -- the UI of the view.

* **init\(view, url\)**

The method is called only once for every instance of a view class when the view is rendered. It can be used to change the initial UI configuration of a view. For instance, the above defined toolbar will be always rendered with the first segment active regardless of the URL. You can link the control state to the URL:

```js
class ToolbarView extends JetView{
    config(){
        return { 
            view:"toolbar", elements:[
                { view:"label", label:"Demo" },
                { view:"segmented", localId:"control", options:["Details", "Dash"], click:function(){
                    this.$scope.app.show("/Demo/"+this.getValue())
                }}
            ]
        };
    }
    init(view, url){
        if (url.length)
            this.$$("control").setValue(url[1].page);
    }
}
```

The method receives two parameters:

* the view UI
* the URL

* **urlChange\(view,url\)**

This method is called every time there's a change of views. It reacts to the change in the URL after **!\#**. **urlChange** is only called for the view that is rendered and for its parent. Consider the following example. The initial URL is:

```
/Layout/Demo/Details
```

If you change it to

```
/Layout/Demo/Preview
```

**urlChange** will be called for **Demo** and **Preview**. [Check out the demo](https://github.com/webix-hub/jet-core/blob/master/samples/02_life_stages.html).

The **urlChange** method can be used to restore the state of the view according to the URL, e.g to highlight the right menu item.

```js
class ToolbarView extends JetView{
    config(){
        return { 
            view:"toolbar", elements:[
                { view:"label", label:"Demo" },
                { view:"segmented", localId:"control", options:["Details", "Dash"], click:function(){
                    this.$scope.app.show("/Demo/"+this.getValue())
                }}
            ]
        };
    }
    init(view, url){
        if (url.length)
            this.$$("control").setValue(url[1].page);
    }
    urlChange(view, url){
        if (url.length > 1)
            this.$$("control").setValue(url[1].page);
    }
}
```

The method receives two parameters:

* the view UI
* the URL

* **destroy**

**destroy** is called only once for each instance when the view is destroyed. The view is destroyed when the corresponding URL element is no longer present in the URL. **destroy** also can be used to destroy temporary objects like popups and other child components to prevent memory leaks.

```js
class ToolbarView extends JetView{
    config(){
        return { 
            view:"toolbar", elements:[
                { view:"label", label:"Demo" },
                { view:"segmented", localId:"control", options:["Details", "Dash"], click:function(){
                    this.$scope.app.show("/Demo/"+this.getValue())
                }}
            ]
        };
    }
    init(view, url){
        var popup = webix.ui({
            view:"popup", 
            body:"Toolbar is created"
        });

        if (url.length)
            this.$$("control").setValue(url[1].page);
    }
    urlChange(view, url){
        if (url.length > 1)
            this.$$("control").setValue(url[1].page);
    }
    destroy(){
        popup.destructor();
    }
}
```

### ready


config
init
urlChange
	config
	init
	urlChange
	ready
ready



## Subview

Apart from direct injection, there are two more options of creating subviews.

### 1. View Injection

You can inject views into other views. For example, here are three views:

```js
/* views/myview.js */
export class MyView extends JetView {
    config() => { template:"MyView text" };
}
/* views/details.js */
export const Details = { 
    cols: [
        { template:"Details text" },
        { $subview:true } 
    ]    
};
/* views/form.js */
export const Form = () => {
    view:"form", elements:[
        { view:"text", name:"email", required:true, label:"Email" },
        { view:"button", value:"save", click:() => this.show("Details") }
    ]
}
```

Let's group them into a bigger view:

```js
/* views/bigview.js */
import {MyView} from "myview"
import {Details} from "details"
import {Form} from "form"
const BigView = {
    rows:[
        MyView,
        { $subview:"/Details/Form" }
    ]
}
```

### 2. App Injection

App is a part of the whole application that implements some scenario and is quite autonomous. It can be a subview as well. Thus you can create high level appications.

```js
/* views/form.js */
export class FormView extends JetView {
    config() {
        return {
            view: "form",
            elements: [
                { view: "text", name: "email", required: true, label: "Email" },
                { view: "button", value: "save", click: () => this.show("Details") }
            ]
        };
    }
}
/* view/details.js */
export const DetailsView = () => ({
    template: "Data saved"
});
```

Let's group these views into an app module:

```js
/* views/app1.js */
import {FormView} from "form"
import {DetailsView} from "details"
export var app1 = new JetApp({
    start: "/Form",
    router: JetApp.routers.EmptyRouter,
    views: {
        "Form": FormView,
        "Details": DetailsView
    }
});
```

Note that this app module isn't rendered. Next, the app is included into another view:

```js
/* views/page.js */
import {app1} from "app1"
export const PageView = () => ({
    rows: [ Toolbar, app1 ]
});
```

Finally, the view can be put into an app and rendered:

```js
/* app2.js */
import {PageView} from "page"
var app2 = new JetApp({
    start: "/Page",
    router: JetApp.routers.HashRouter,
    views: {
        "Page": PageView
    }
}).render();
```

As a result, this is a two-level app. [Check out the demo](https://github.com/webix-hub/jet-core/blob/master/samples/06_highlevel.html).
