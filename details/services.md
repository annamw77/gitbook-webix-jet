# Services

Services are one more means of view communication besides [the event bus and methods](events.md). Methods are preferable when you want to connect views with their children. This is impossible if views aren't related to each other, e.g. if you want to connect two subviews of the same view. This is where services help. You can create a service connected to one of the views, and other views can communicate with the view through the service. In short, a service here is created as shared functionality.

The same communication can be implemented with events, but this way is much more complex. With JS6, the problem can also be solved with creating and requiring a module. Services are better, because if you create two apps that require the same module, there's only one instance of the shared code. With services, a new instance of a service is created each time.

JetApp provides a means to initialize services. For example, you have a *masterTree* view and want other views to get the ID of a selected item. To set a service, call the **setService** method:

```js
init() {
	this.app.setService("masterTree", {
		getSelected : () => this.getRoot().getSelectedId()
	})
}
```

*getSelected* of the *masterTree* service returns the ID of the selected node of the tree. To call *getSelected*, use the **getService** method:

```js
this.app.getService("masterTree").getSelected();
```

<!-- example
/* data.js */
import {JetView} from "webix-jet";
import {records} from "../models/records"

export default class DataView extends JetView {
    config(){
        return {
            view:"datatable", autoConfig:true
        };
    }
    init(view){
        view.parse(records);
        this.app.setService("masterData", {
            getSelected : () => this.getRoot().getSelectedItem()
        });
        view.select(1);
    }
}
/* services.js */
import {JetApp, JetView} from "webix-jet";
import DataView from "views/data";

class SmallData extends JetView {
    config(){
        return {
            view:"datatable", autoConfig:true
        };
    }
    init(view){
        var item = this.app.getService("masterData").getSelected();
        view.parse(item);
    }
}

class Layout extends JetView {
    config(){
        return {
            cols:[
                {$subview: DataView},
                {$subview: SmallData}
            ]
        };
    }
}

webix.ready(() => {
	const app = new JetApp({
		start:		"/start",
		views:{
			start: Layout
		}
	}).render();
});
-->

Apart from view communication, services can be used for [loading and saving data](models.md).