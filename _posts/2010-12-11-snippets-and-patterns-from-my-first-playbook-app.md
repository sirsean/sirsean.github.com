---
layout: post
title: Snippets and patterns from my first Playbook app
tags:
- Actioneer
- ActionScript
- Flex
- Mobile
- Playbook
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _wp_old_slug: ''
author: sirsean
---
This week I heard about RIM's plan to build a developer base for the upcoming Playbook tablet -- give a free one to everyone who builds an app and gets it accepted into their App World. Now, I don't know how stringent their review policy is, but the promise of a free tablet is enough to get me to write an app. I decided to port one of my iPhone apps, Actioneer, which has seemed constrained by the size of the phone's screen; hopefully an app that's good enough for Apple's App Store is good enough for RIM's App World.

The app is done now. The basic concept of the app is that you can define a set of "Actions" that you'll perform from time to time; when you click "Perform" on one of them, it records the time the action was performed, and it will display a history of all the times each action was performed. Darlene uses it to help her remember how often Rusty craps, which apparently is valuable information for a mother.

There are a few interesting things in the implementation of the app, that I want to highlight here. Playbook apps are written in Flex, and I've used the Cairngorm library pretty extensively in the past -- I had thought it was a little heavy, and I took this as an opportunity to take some ideas from it but make it a little bit lighter and easier to deal with.

I start with the ModelLocator, which is a concept stolen from Cairngorm. It's a singleton that stores data that'll be shared between different views -- but I've added the concepts of persisting that data to local storage, as well as dispatching events. As I get around to larger apps, I'll probably find it necessary to split the persistence code out into another class, but I think the event dispatching works better here than in Cairngorm's FrontController.

Here's my ActionModelLocator:

    package model
    {
        import events.ActionAddedEvent;
        
        import flash.events.Event;
        import flash.events.EventDispatcher;
        import flash.net.SharedObject;
        
        import mx.collections.ArrayCollection;

        [Bindable]
        public class ActionModelLocator extends EventDispatcher
        {
            private static var instance:ActionModelLocator;
            
            private var sharedObject:SharedObject;
            
            public var actions:ArrayCollection;
            
            public function addAction(action:Action):void {
                actions.addItem(action);
                dispatchEvent(new ActionAddedEvent(action));
                flush();
            }
            
            public function removeAction(action:Action):void {
                var index:int = actions.getItemIndex(action);
                if (index >= 0) {
                    actions.removeItemAt(index);
                    flush();
                }
            }
            
            private function flush():void {
                var serializedActions:ArrayCollection = new ArrayCollection();
                for each (var action:Action in actions) {
                    serializedActions.addItem(action.serialize());
                }
                sharedObject.data.actions = serializedActions;
                sharedObject.flush();
            }
            
            private function load():void {
                actions = new ArrayCollection();
                if (sharedObject.size > 0) {
                    for each (var obj:Object in sharedObject.data.actions) {
                        actions.addItem(Action.deserialize(obj));
                    }
                }
            }
            
            public function ActionModelLocator()
            {
                if (instance != null) {
                    throw new Error("Can only be one ActionModelLocator");
                }
                
                sharedObject = SharedObject.getLocal("Actioneer_Actions");
                load();
                
            }
            
            public static function getInstance():ActionModelLocator {
                if (instance == null) {
                    instance = new ActionModelLocator();
                }
                return instance;
            }
        }
    }

The constructor, which can only be executed once, initializes the sharedObject variable by grabbing a local persistence object and calling load(), which will pull any data out of the local datastore and deserialize it for use. I'll get to the serialization/deserialization in a second, but first I want to finish explaining the ModelLocator.

The converse of load() is flush(), which serializes all of our model objects and saves them to the SharedObject, and takes the extra step of flushing the SharedObject to disk. This last step probably isn't necessary -- the SharedObject is supposed to flush to disk when it needs to, like when the app is quit or event pushed from memory in the background. I don't trust that at the moment, though, so maybe once I actually have a Playbook I can investigate whether that works, and adjust this later.

I have addAction/removeAction methods that handle adding and removing Action objects from the list and calling the aforementioned flush() method. The addAction method also has the added benefit of dispatching an ActionAddedEvent, because we'll want to be able to add an Action in one view and respond to an Action being added in another view.

The events themselves are very simple; they extend the basic flash.events.Event class, and all they need to do is define a name (which needs to be public/static and unique per VM) and encapsulate any objects that you want to pass along in the event. Here's the ActionAddedEvent:

    package events
    {
        import flash.events.Event;
        
        import model.Action;
        
        public class ActionAddedEvent extends Event
        {
            public static const NAME:String = "AddActionEvent";
            
            public var action:Action;
            
            public function ActionAddedEvent(action:Action, type:String=NAME, bubbles:Boolean=false, cancelable:Boolean=false)
            {
                super(type, bubbles, cancelable);
                this.action = action;
            }
        }
    }

To respond to an event, you just need to add an event listener to the model locator that will dispatch the events. I did mine in the creationComplete handler of my views:

    private function onCreationComplete(event:Event):void {
        actionModelLocator.addEventListener(ActionAddedEvent.NAME, function(e:ActionAddedEvent):void {
            ...
        });
    }

Finally, I'll show you the Action model object, which holds the data for each action and handles serialization/deserialization of each action.

    package model
    {
        import mx.collections.ArrayCollection;

        [Bindable]
        public class Action
        {
            public var name:String;
            public var history:ArrayCollection;
            
            public function Action(name:String=null, history:ArrayCollection=null)
            {
                this.name = name;
                if (history == null) {
                    history = new ArrayCollection();
                }
                this.history = history;
            }
            
            public function perform():void {
                history.addItemAt(new Date(), 0);
            }
            
            public function deleteHistory(date:Date):void {
                var index:int = history.getItemIndex(date);
                if (index >= 0) {
                    history.removeItemAt(index);
                }
            }
            
            public function serialize():Object {
                var obj:Object = {
                    name: this.name,
                    history: this.history
                };
                return obj;
            }
            
            public static function deserialize(obj:Object):Action {
                return new Action(obj.name, obj.history);
            }
        }
    }

I'd heard a rumor that when you save something to SharedObject, it would serialize ActionScript objects for you, so you'd be able to put them in and pull them out easily. No such luck, however. When I tried to save an Action directly into the SharedObject, it came back out as a generic Object that couldn't be cast or coerced into an Action -- it was stuck. So my serialize/deserialize methods convert the Action into and out of an Object that the SharedObject persistence can use. (Note: the serialize() method may not be totally necessary in this case, but I'm using it to demonstrate a pattern I expect to have to use in general.)

Hopefully that gives you some ideas as you build your own Playbook apps. I'm not thrilled with the level of documentation, and the FlashBuilder IDE keeps steering me away from components I would normally use that aren't "mobile optimized," so I have a lot more to learn when it comes to working with MXML in my Playbook apps.

And I'm curious to see how much of this stuff I can pull out into libraries as I build my next Playbook apps.
