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
- `isLoaded(loaded)` which is used to increase or dicrease the loading count (see `loadedCountDown` attribute). The default behaviour is to add a "loading" class to the current element if loading count is greater than 0 and remove class "loading" when it is 0.
- `close` which is used to close (remove view) and call `onClose` function (that you may implements to unbind some models)
- `getHelpers` returns an object with 2 helpers: `displaySize(bytes)` and `displayDate(unixtimestamp)`

There's also two attributes:
- `templateName` which set the template name.
- `loadedCountDown` which set the loading count. `isLoaded(true)` must be called `loadedCountDown` times to set that the view is loaded. The default value is 1, you may change it to wait model loading.

### View example


```javascript
var ListItemView = Backbone.DeferedView.extend({

    tagName:'div',
    templateName: "browser-list",
    className: 'row-fluid',
    loadedCountDown: 2,
  
    initialize:function () {
        this.model.bind("sync", function () {
      	    this.isLoaded(true);
        }, this);
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
