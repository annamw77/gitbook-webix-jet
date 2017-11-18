# Navigation

There are several ways to navigate through an app with Webix Jet.

### 1. Jet Links

Jet links are created with the **route** attribute instead of the usual **href**. The **route** attribute allows you to stop users from leaving the current view. This can be useful in the case of unsaved data, for instance. See the details in the [section on plugins](plugins.md). Here's an example of a Jet link:

```html
<a route="/details/data"></a>
```

After you click on the link, the app UI will be rebuilt and will consist of the main view _Details_ and a subview _Data_. Note that there's no hashbang in the path.

The **route** attribute can be assigned not only to links, but to *any control* like a button or a menu option.

You can pass one or more parameters with a Jet link:

```html
<a route="/details/data?id=2"></a>
<!-- or several -->
<a route="/details/data?id=2&name=some"></a>
```

### 2. Navigation with HTML links

Apart from navigating with links with the **route** attribute, you can create HTML links. This way is not so convenient, as the first one. You cannot prevent users from leaving the current view through an HTML link. In case there's no worries and you don't plan to guard users' unsaved data, you can create links with the **href** attribute. Suppose you have these view modules:

~~~js
/* views/demo.js */
import {ToolbarView} from "toolbar"
const DemoView = {
    rows: [
        ToolbarView,
        { $subview: true }
    ]
};
/* views/details.js */
const DetailsView = {
    template: "App"
};
~~~

This is a link to the **Details** view as a subview of **DemoView**:

~~~html
<!--index.html-->
<a href="#!/Demo/Details">Data</a>
~~~

You can pass one or more parameters in the link, e.g.:

```html
<a href="#!/Demo/Details?id=2">Data</a>
<!-- or more -->
<a href="#!/Demo/Details?id=2&name=some">Data</a>
```

### 3. app.show\(\)

Apart from links, you can use the **show** method to switch views. **app.show\(\)** will rebuild the whole app or app module that called the method. A specific instance of the related view class is referenced with **this** if your handler is an *arrow function*. For more

Reference app as **this.app** to call the **show** method from control handlers if they are arrow functions, for instance:

```js
{ view:"button", value:"details", click: () => {
    this.app.show("/demo/details");
}}
```

After a button click, the URL will change, and the app UI will be rebuilt according to it.

You can read more about ["Referencing views"](../detailed/referencing.md).

You can pass one or more parameters to show alongside the URL:

```js
this.app.show("/demo/details?id=2");
// or many
this.app.show("/demo/details?id=2&name=some");
```

### 4. view.show\(\)

You can also change the URL by calling the **show\(\)** method from a specific view. A specific instance of the related view class is referenced with **this**. Calling **show** from a view gives you more freedom, as it allows rebuilding only this view or only its subview, not the whole app or app module. For example, suppose you have a view like this:

```js
// layout.js
cols:[
    layout,
    { $subview:true }
]
```

The current URL is _"/layout/details"_, so the subview is **details**. To replace **details** with a different subview, specify the name as it is or with *./*:

```js
{ view:"button", value:"demo", click: () => {
    this.show("demo");
}}

//or

{ view:"button", value:"demo", click: () => {
    this.show("./demo");
}}
```

The resulting URL is going to be */layout/demo*.

If you want to rebuild the whole app and load the **details** view as the only view, specify the name of the view with a slash:

```js
// toolbar.js
...
{ view:"button", value:"Details", click: () => {
    this.show("/details");
}}
...
```

The syntax of showing views resembles the way you navigate through directories. So you can move some levels up from _"/layout/details"_, for example:

```js
{ view:"button", value:"demo", click: () => {
    this.show("../../bigview");
}}
```

As a result, the app URL will be */bigview*.

You can pass one or more parameters to show alongside the URL:

```js
this.show("details?id=2");
// or many
this.show("details?id=2&name=some");
```