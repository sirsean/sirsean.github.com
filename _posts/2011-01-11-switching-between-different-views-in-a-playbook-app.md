---
layout: post
title: Switching between different views in a Playbook app
tags:
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
In my Playbook apps, I often need to switch from one view to another. A common example is that I'm on my base screen that you see when you first launch the app, and you want to add or edit something in the list of items I'm showing you. I'll try to explain how I've done that.

My add/edit screen is a subclass of View; in this case, I call it FamilyEditor.

    <?xml version="1.0" encoding="utf-8"?>
    <s:View xmlns:fx="http://ns.adobe.com/mxml/2009" 
            xmlns:s="library://ns.adobe.com/flex/spark" 
            title="Family Editor"
            xmlns:mx="library://ns.adobe.com/flex/mx">

To switch from the base view to FamilyEditor, I need to call navigator.pushView(). Sometimes I want to add a new Family, and other times I want to edit an existing one. So I want to pass a Family to FamilyEditor. I do that by connecting these event handlers to my add/edit buttons:

    private function addButtonClickHandler(event:Event):void {
        navigator.pushView(FamilyEditor, new Family());
    }

    private function editButtonClickHandler(event:Event):void {
        navigator.pushView(FamilyEditor, familiesList.selectedItem);
    }

When you use MXML to extend a class, you don't get to define a constructor. So when you pass your argument into FamilyEditor, you need to override the data setter (just like you would when creating an item renderer).

    public override function set data(value:Object):void {
        super.data = value;
        family = Family(value);
        ...
    }

I can do my thing in the FamilyEditor view, and when I'm done I just call navigator.popView(). I do that when the user cancels, and when they click save I do it after performing the necessary saving operations. Here's the cancel button's event handler:

	private function cancelButtonClickHandler(event:Event):void {
        navigator.popView();
    }

That goes back to the view the user was on before they got here.

So that's simple enough, I think. And you can do a whole lot with it.
