---
layout: post
title: "JMock: Mocking Concrete Classes"
tags: [Java, JMock, Mock]
author: sirsean
---

Normally when you're testing with JMock, you want to design your code to use interfaces as much as possible, and mock the interfaces; that way, the only implementation code that's involved in your tests is the code you actually want to test.

But sometimes, you find yourself using something that doesn't have an interface, but you don't want to include its functionality within your test. You just want to be able to mock it. JMock normally doesn't let you do that. If you try, you get something like this (a test failure):

    java.lang.IllegalArgumentException: com.vikinghammer.sample.SomeConcreteClass is not an interface
    at java.lang.reflect.Proxy.getProxyClass(Proxy.java:362)
    at java.lang.reflect.Proxy.newProxyInstance(Proxy.java:581)
    at org.jmock.lib.JavaReflectionImposteriser.imposterise(JavaReflectionImposteriser.java:31)
    at org.jmock.Mockery.mock(Mockery.java:139)
    at org.jmock.Mockery.mock(Mockery.java:120)

I've done this before, a couple of times, but now that I find I need to do it again I'd forgotten exactly how to do it. Maybe writing it down will help me remember, for next time! ([JMock's website explains how to do it](http://www.jmock.org/mocking-classes.html), but I thought I'd show how I do it in my environment, which is slightly different.)

First, add the necessary dependency to your pom.xml (that is, if you're using Maven).

    <dependency>
        <groupId>org.jmock</groupId>
        <artifactId>jmock-legacy</artifactId>
        <version>2.5.1</version>
        <scope>test</scope>
    </dependency>

Then, you'll need to import your ```ClassImposteriser```.

    import org.jmock.lib.legacy.ClassImposteriser;

And then:

    Mockery context = new JUnit4Mockery();

    @Before
    public void setupMocks() {
        context.setImposteriser(ClassImposteriser.INSTANCE);
        concreteThing = context.mock(SomeConcreteClass.class);
    }

And now you can use it just like you would any mocked interface. Nice and simple.
