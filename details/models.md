# <span id="contents">Models</span>

View modules return the UI configuration of the components. They describe the visual aspect of an application and shouldn't contain any data. For data, there is another type of modules - a **model**. Models are JS files that are placed in a separate folder. The model object stores all functionality related to some data entity. This division of UI and data is an advantage because if the data changes, all you have to do is change the data model. There's no need to modify view files. 

There are several ways of loading data in Webix Jet:

- [shared models](#shared)
- [dynamic models](#dynamic)
- [remote models](#remote)
- [services for data](#services)
- [models with webix.remote](#webix_remote)

### [1. <span id="shared">Shared Data &uarr;</span>](#contents)

If you have some *relatively small data* and plan to use them in *many components*, you can create a model, initialize a data collection in it and load the data into the data collection. Here's an example of a data collection with static data:

```js
// models/records.js
export const records = new webix.DataCollection({ data:[
	{ id:1, title:"The Shawshank Redemption", year:1994, votes:678790, rating:9.2, rank:1},
	{ id:2, title:"The Godfather", year:1972, votes:511495, rating:9.2, rank:2},
	//...other records
]});
```

This is how the data is loaded from and saved to the server:

```js
// models/records.js
export const records = new webix.DataCollection({ 
	url:"data.php",
	save:"data.php"
});
```

To use the data in a component, you need to parse it. You must parse data in **init**, not in **config** (leave *config* for UI only). Have a look at the example with a datatable:

```js
// views/data.js
import {JetView} from "webix-jet";
import {records} from "../models/records";

export default class DataView extends JetView{
	config: () => {
		view:"datatable", autoConfig:true, editable:true
	}
	init(view){
		view.parse(records);
	}
}
```

All the changes made in the datatable are saved to the server.

You can also sync a data component inside a Jet view to a DataCollection. Let's sync the datatable with records:

```js
// views/data.js
import {JetView} from "webix-jet";
import {records} from "../models/records";

export default class DataView extends JetView{
	config: () => {
		view:"datatable", autoConfig:true, editable:true
	}
	init(view){
		view.sync(records);
	}
	removeRecord(id){
		records.remove(id);
	}
}
```

Mind that if you synced a data component to a DataCollection, you must *perform **add/remove** operations on the master collection* while the synced view will reflect these changes automatically. Slave views can only update the master.

###### Shared Data Transport

To return several types of data and afterwards distribute data chunks to different views, a shared data transport can be used.

For example, this is the data model for grid:

```js
//models/griddata.js
import {sharedData} from "models/shared";

const gridData = new webix.DataCollection();
gridData.parse(sharedData("grid"));

export gridData;
```

And here is a separate file "shareddata" which communicates with shared data feed and can provide different data chunks for different consumers:

```js
//models/shareddata.js
var data = webix.ajax("some.json").then(a => a.json());
export function sharedData(name){
    return data.then(a => {
		switch name:
			case "grid":
				return a.grid;
			default:
				return [];
    });
}
```

At the same time, each entity has its own model that stores data and all related API and just uses shareddata for data loading.

### [<span id="dynamic">2. Dynamic Data &uarr;</span>](#contents)

This is the model for *big data* (less than 10K records). These data can be *used only once* and mustn't be cached. The data can be loaded from a server with an AJAX request:

```js
// models/records.js
export function getData(){
	return webix.ajax("data.php");
}
```

or from local storage:

```js
// models/records.js
export function getData(){
	return webix.storage.local.get("data");
}
```

To parse data, pass the return value of *getData* to *view.parse*:

```js
// views/data.js
import {JetView} from "webix-jet";
import {getData} from "../models/records";

export default class DataView extends JetView{
	config: () => { 
		view:"datatable", autoConfig:true
	}
	init(view){ 
		view.parse(getData());
	}
}
```

To save data, you can add one more function to the *records* model:

```js
// models/records.js
export function getData(){
	return webix.ajax("data.php");
};
export function saveData(){
	return webix.ajax().post("data.php", { data });
}
```

In *init* you need to define a way to save data:

```js
// views/data.js
import {JetView} from "webix-jet";
import {getData, saveData} from "../models/records";

export default class DataView extends JetView{
	config: () => { 
		view:"datatable", autoConfig:true, editable:true
	}
	init(view) {
		view.parse(getData());
		view.define("save", saveData);
	}
}
```

Loading and saving data can also be in **config** of the view module, if needed:

```js
// views/data.js
import {JetView} from "webix-jet";
import {getData, saveData} from "../models/records";

export default class DataView extends JetView{
	config: () => {
		view:"datatable", autoConfig:true,
		url: getData,
		save: saveData
	}
}
```

### [<span id="remote">3. Remote Models for Huge Data &uarr;</span>](#contents)

For really *huge data* (more than 10K records), you can use dynamic loading of Webix components and drop models :) Data will be loaded in portions when needed. For that, load data in the view code. You can do it with the **url** property. For saving data, use the **save** property.

```js
// views/data.js
import {JetView} from "webix-jet";

export default class DataView extends JetView{
	config() => {
		view:"datatable", autoConfig:true,
		url:"data.php",
		save:"data.php"
	}
}
```

Data can also be loaded dynamically with the **load** method:

```js
// views/data.js
import {JetView} from "webix-jet";

export default class DataView extends JetView{
	config(){
		var ui = { view:"datatable", autoConfig:true };
		ui.load("data.php");
		return ui;
	}
}
```

Remote models are also convenient for prototyping.

### Shared vs Dynamic vs Remote models

Let's recap main differences of the three ways to load and save data:

| Model   | Data Size | Usage                 |
|---------|-----------|-----------------------|
| Shared  | Small     | Many times            |
| Dynamic | Big       | Once                  |
| Remote  | Huge      | Handy for prototyping |


### [<span id="services">4. Services as Data Sources &uarr;</span>](#contents)

You can use services instead of models as data sources. Suppose there's a list of customers and a grid that displays records on a selected customer. Here's how you can use a service that returns the ID of a selected list item:

```js
var id = service.getSelected();
var data = records.getData(id);
```

A better and shorter way is:

```js
var data = service.getNomenclature();
```

### [<span id="webix_remote">5. Using Webix Remote with Webix Jet &uarr;</span>](#contents)

You can use [webix.remote](https://docs.webix.com/desktop__webix_remote_php.html) instead of sending AJAX requests. You must have a server-side script, e.g.:

```html
<script src="api.php"></script>
```

The script can contain a *nm* class like this:

```php
class Nomencl {
	public function getData(id) { /*..*/ }
}
$api->setClass("nm", new Nomencl());
```

Here's a model that gets the class and all its methods:

```js
// models/nomencl.js
export default webix.remote.nm;
```

You can create a DataCollection:

```js
// models/nomencl.js
var data = new DataCollection({
	url: webix.remote.nm.getData
})
```

You can import the model and parse it into a view component:

```js
// views/data.js
import records from "../models/nomencl";
...
init(view){
	view.parse(records.getData(id));
}
```

**webix.remote** is better then an AJAX request for several reasons.

First, you don't need to serialize data after loading. Compare the results of these two requests:

```js
var data1 = webix.ajax("data/nomencl/154"); 		//"{"id":154,"name":"John"}"
var data2 = webix.remote.nm.getNomenclature(154);	//{id:154,name:"John"}
```

After receiving the response of the AJAX request, you have to call <code>JSON.parse(data1)</code> to turn a string into an object. The response of the second request is already an object.

Second, requests with *webix.remote* are safer due to CSRF-security.

And third, several requests are sent as one, which makes operations faster.

## Further reading

For more details on services, read:

- [Services](services.md)