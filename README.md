Backbone Defered View Loader
============================

A backbone view overlay that loads view only when needed.
It adds two components:
- `Backbone.TemplateManager`
- `Backbone.DeferedView` that extends `Backbone.View`

In order to use the defered loader:
- Your view need to extends `Backbone.DeferedView`
- You have to render using `renderTo` or `deferedRender` function.

## Backbone.TemplateManager

This object have the charge to keep copies of loaded templates and load templates if they are needed by any view. 
It is using a jQuery defered object to load them and call-back the rendering process.

*Configuration:*
- `Backbone.TemplateManager.baseUrl` define the default templates URL, based on their names.
  _default: `/templates/{name}`_

## Backbone.DeferedView

It adds 5 functions to the backbone default view:
- `deferedRender` which start the rendering process.
- `renderTo(container)` which start the rendering process in a specific container. *Recommended* because when you `renderTo` another view to this `container`, the previous view is `close()`ed.
- `isLoaded(loaded)` which is used to set the loading state. You can use it to add/remove a "loading" class for instance to the current element.
- `close` which is used to close (remove view) and call `onClose` function (that you may implements to unbind some models)
- `getHelpers` returns an object with 2 helpers: `displaySize(bytes)` and `displayDate(unixtimestamp)`

The template name is set by the `templateName` attribute.

### View example



```javascript
var ListItemView = Backbone.DeferedView.extend({

    tagName:'div',
    templateName: "browser-list",
    className: 'row-fluid',
    
    // Use to have a loading state also with model
    synched: false,
    templateLoaded: false,
  
    initialize:function () {
        this.model.bind("sync", function () {
            this.synched = true;
      	    this.isLoaded(undefined);
        }, this);
    },
    
    // Intercept isLoaded function to call super (which just add or remove "loading" class)
    // only if template is loaded AND model synchronized
    isLoaded: function (loaded) {
        if (loaded != undefined) {
      	  this.templateLoaded = loaded;
        }
        
        Backbone.DeferedView.prototype.isLoaded.apply(this, [this.templateLoaded && this.synched]);
    },
    
    render: function () {
        this.$el.html(this.template(
            this.model.toJSON()
        ));
    }
});

...

// Create a new collection model
var files = new FileCollection([], {
    children_url: userOptions.children_url,
    parentId: this.currentPath
});

// Defered render list
this.list = new ListItemView({model: files});
this.list.renderTo($('div#browser-list'));
files.fetch();

```
