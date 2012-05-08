---
layout: post
title: ! 'Playbook/Flex: Delete button inside a List ItemRenderer'
tags:
- ActionScript
- Flex
- Mobile
- Playbook
status: publish
type: post
published: true
meta:
  _wp_old_slug: ''
  _edit_last: '2'
author: sirsean
---
I'm doing a bit more Playbook development, and I wanted to be able to delete items out of a List with a button inside each item. The way to do that is by defining your own ItemRenderer.

Here's what it looks like:

![Item renderer with a delete button](/wp-content/images/item_renderer_with_delete_button.png)

I'm using a ModelLocator just like the one I described in [my last post about Playbook development](http://vikinghammer.com/2010/12/11/snippets-and-patterns-from-my-first-playbook-app/), and I've defined a new MXML component that displays the name of the object and has a delete button in it.

    <?xml version="1.0" encoding="utf-8"?>
    <s:ItemRenderer xmlns:fx="http://ns.adobe.com/mxml/2009" 
                    xmlns:s="library://ns.adobe.com/flex/spark" 
                    autoDrawBackground="true" xmlns:mx="library://ns.adobe.com/flex/mx">
        
        <fx:Script>
            <![CDATA[
                import model.ModelLocator;
                import model.Family;
                
                [Bindable]
                private var modelLocator:ModelLocator = ModelLocator.getInstance();
                
                [Bindable]
                private var family:Family;
                
                public override function set data(value:Object):void {
                    super.data = value;
                    
                    family = Family(value);
                }
                
                private function deleteClickHandler(event:Event):void {
                    modelLocator.removeFamily(family);
                }
                
            
        
        
        
        
        
            
            
            
        
        
    

And using the ItemRenderer (which I've named FamilyRenderer in this case, and placed in the views.renderer package where I've been putting all my renderers) is very simple.

    

The ModelLocator.removeFamily method is about what you'd expect:

    public function removeFamily(family:Family):void {
        var index:int = families.getItemIndex(family);
        families.removeItemAt(index);
        flush();
    }

What happens here is that when you click that delete button, we call out to the ModelLocator and tell it to remove the item that's currently being rendered. Since the entire ModelLocator is [Bindable], the change to ModelLocator.persons will update what's displayed in the familiesList, and the ItemRenderer on which you just clicked the delete button disappears.]]>
