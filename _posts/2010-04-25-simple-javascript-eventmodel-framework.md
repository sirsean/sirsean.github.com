---
layout: post
title: Simple Javascript event/model framework
tags:
- javascript
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Last weekend I wrote my first iPhone app, Tabata Sprinter, and Apple let it into their App Store late in the week. I took that as validation that using Appcelerator Titanium to write iPhone apps is still acceptable, because that's what I'd used. So this weekend, I wrote a second app, Choose4me, which is currently awaiting review.

While making the first app I started a model/event binding library, and I have improved upon it for the second app.

The concept is that you want to store your values in a central location, and listen for any changes in their values so you can update the UI or fire any other events that you want. Without further ado, here's the code:

    var VH = {
        Model: function () {
            this._listeners = {};
            this._fields = {};
            this._boundListeners = {};

            this.listen = function(key, listener) {
                if (this._listeners[key] == null) {
                    this._listeners[key] = [];
                }
                this._listeners[key].push(listener);
            };

            this.set = function(key, value) {
                var oldValue = this._fields[key];
                this._fields[key] = value;
                if (oldValue !== value) {
                    this.poke(key);
                }
            };

            this.poke = function(key) {
                if (this._listeners[key] != null) {
                    for (var i=0; i < this._listeners[key].length; i++) {
                        var listener = this._listeners[key][i];
                        listener(this._fields[key]);
                    }
                }
            };

            this.get = function(key) {
                return this._fields[key];
            };
        }
    };

To listen for a changing variable, you call listen() and pass it a callback that takes a single argument -- the newly changed value.

When you set() a variable, it checks to see if the new value is different from the old value, and if so, notifies all listeners using poke(). Note that you can call poke() yourself, if you want to notify listeners without setting a value (for events, perhaps).

### Example

You instantiate your model at the beginning; it should be among the first things you do.

    var model = new VH.Model();
    var delegate = new VH.Model();

I'm using two of them; the "model" is for storing values, and the "delegate" is just for firing events (ie, you use the key/value pairs but you don't care about the values). This is just a convention I came up with, you may not find it useful.

Then you set default values, which your components may need during their setup.

    model.set(C4.fields.numOptions, 2);

(Note that C4.fields.numOptions is just a string constant. You'll want each key to be a unique value, and using constants helps keep track.)

While you're setting up your components, you'll want to listen for changing values.

    delegate.listen(C4.events.choose, function(v) {
        model.set(C4.fields.currentChoice, VH.random(1, model.get(C4.fields.numOptions)));
    });

Here, we're listening for any time someone calls delegate.poke(C4.events.choose), which indicates that the "choose" event has been fired. So when does that happen?

    chooseButton.addEventListener("click", function(e) {
        delegate.poke(C4.events.choose);
    });

I have a button, and I attached an event listener to it, so that any time they click the button we'll poke the delegate to fire the choose event. I could have done this differently -- either by doing the random calculation right in the click event handler and setting the model field directly, or by using a function rather than listening for the choose event. But this works better because we get the benefit of writing our logic only once just like a function, while avoiding any "function is called before it's defined" warnings that crop up during cross-compilation.

Using this model has worked out great for me in my first two apps, and what's nice is that I expect it'll also work nicely in a regular webapp the next time I write one.

So hopefully Appcelerator Titanium remains kosher on the App Store, because this is a pretty fun way to program.
