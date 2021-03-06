In Backbone application, we often need to take care of cleanup the zombie views. In some other use case, we might want to reuse existing view because it rarely changes. This simple View Manager will take care of the two use cases minimal change your existing code.

# Setup
Just include path to vm.js after jQuery, underscore and Backbone, and you are ready to go.
```javascript
<script src="path/to/jquery"></script>
<script src="path/to/underscore"></script>
<script src="path/to/backbone"></script>
<script src="path/to/vm.js"></script>
```

# How to use

If in your existing application, you may create a new view by:
```javascript
ProductView = Backbone.View.extend({...});
productView = new ProductView({...});
$(this.el).append(productView.render().el);
```
This code, although simple, will create zombie view, because we constantly create new view without remove previous view. If you are not familiar with zombie view, you might refer to an excellent explanation here: http://lostechies.com/derickbailey/2011/09/15/zombies-run-managing-page-transitions-in-backbone-apps/

However it's not convenient to call view.dispose() for all Backbone Views, especially if we create views in for loop.

I think the best timing to put cleanup code is before creating new view. My solution is to create a View Manager helper to do this cleanup:
```javascript
ProductView = Backbone.View.extend({…});
productView = VM.createView("productView", function() {
    return new ProductView({…});
});
$(this.el).append(productView.render().el);
```

Using this minimal change, VM will automatically look for view with name equals to "productView", and if it existed, VM will cleanup the view for you before creating new view instance.

If you have a view that is rarely changes, (e.g views which do not bind to any model/collection), you might consider reuse existing view, so it will speed up your app. You can do it by:
```javascript
ProductView = Backbone.View.extend({…});
productView = VM.reuseView("productView", function() {
    return new ProductView({…});
});
$(this.el).append(productView.render().el);
```
The different between VM.reuseView and VM.createView is that, reuseView will look for existing view with name "productView", if found it will return to you. Else, it will execute the callback function and cache result. VM.createView will always execute the callback function and cleanup existing view for you. Hence you may like to use VM.createView if the views is dynamic and frequently change

Sometime we just need to close existing view, there is a simple function for it:
```javascript
VM.closeView("productView");
```

# About
I got inspired by implementation from https://github.com/thomasdavis/backboneboilerplate. You might like to check it out :)
