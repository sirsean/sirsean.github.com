---
layout: post
title: Cocos2d - Drag from the point you're touching
tags: [Objective-C, cocos2d]
author: sirsean
---

Lately, I've been making apps for kids -- having a kid makes you do that, apparently -- and a big part of making it fun for them is to let them drag stuff around the screen. The way I'd been doing it was in my ```ccTouchesBegan``` method I'd detect which object has been touched and mark it as the "current" object, and in my ```ccTouchesMoved``` method I'd set the object's position to the current touch position.

For naively dragging things around, that works. But the "position" of my objects is their center, which means that as soon as you start dragging the object, it jumps such that your finger is right in the middle of it. Having your finger in the middle of the object isn't a problem at all, especially while dragging, but that initial jump -- which can be kind of a long way if you have a big object to drag -- is pretty crappy.

So, what I want to do is calculate the distance from where you're touching to the center of the object, and then set the position of the object such that it takes that difference into account.

First, I'll edit ```ThingToBeDragged.h```:

    @property(nonatomic, assign) CGPoint dragDifference;

    -(void) setDragPosition:(CGPoint)dragPosition;
    -(void) dragTo:(CGPoint)location;

The ```dragDifference``` variable is where I'll store the difference between where the user touched the object and the object's center. ```setDragPosition``` is where that calculation is made (you should call this when the touch begins), and ```dragTo``` actually does the move (called when the touch moves).

In ```ThingToBeDragged.m``` (remember to ```@synthesize dragDifference``` too):

    -(void) setDragPosition:(CGPoint)dragPosition {
        self.dragDifference = ccpSub([self position], dragPosition);
    }

    -(void) dragTo:(CGPoint)location {
        self.position = ccpAdd(location, [self dragDifference]);
    }

That's the meat of the thing. When the touch begins, we subtract the point where they touched (```dragPosition```) from the center of our object. Then, when it's time to move the object, we add that difference back onto the ```location``` they're currently touching and set that as the new center of the object.

Over in ```GameLayer.m```:

    -(void) ccTouchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
        CGPoint location = [self convertTouchToNodeSpace:[touches anyObject]];

        // ... a bunch of stuff determining which thing was touched ...

        [self currentThing].dragPosition = location;
    }

    -(void) ccTouchesMoved:(NSSet *)touches withEvent:(UIEvent *)event {
        CGPoint location = [self convertTouchToNodeSpace:[touches anyObject]];
        
        if ([self currentThing] != nil) {
            [[self currentThing] dragTo:location];
        }
        
    }

And now, when you start dragging something around, if you put your finger somewhere other than the center of the object, your finger will stay on that part of the object the entire time you're dragging it. (The thing doesn't jump such that your finger is always in the center of the object.)

Makes things better, and is pretty easy.
