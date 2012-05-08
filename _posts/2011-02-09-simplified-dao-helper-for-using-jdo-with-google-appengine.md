---
layout: post
title: Simplified DAO helper for using JDO with Google Appengine
tags:
- Appengine
- Java
- JDO
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _wp_old_slug: ''
author: sirsean
---
I've been playing with Appengine for Java lately, and using JDO (which stands for Java Data Objects) for the first time. All the example code is pretty annoying; apparently, in every DAO method, you have to grab a PersistenceManager from the PersistenceManager factory, use it to build and execute a Query, and then close the PersistenceManager before you return. The framework that has to be included in every method looks like this:

    PersistenceManager pm = pmFactory.getPersistenceManager();
    try {
        // do something with the pm here
    } finally {
        pm.close();
    }

So that's five lines of crap, in every method. But the only thing that actually matters, that I actually want to write in my DAO methods, is the meaty part, the part that says "do something" in that comment there. So, I wrote a little library to let me do just that: it's my [vh-dao](https://github.com/sirsean/vh-dao) project at Github.

It starts with a DAO interface, which says that every DAO class has a store method; I've been on too many projects where sometimes that method is called "store" and sometimes it's "save" and sometimes it's "storeWidget" or "saveWidget" (where "Widget" is the name of the model class). Enough.

    public interface VHDao<T> {
        public void store(T model);
    }
        
Then, I have an abstract base class that implements a few things that'll be the same for every DAO; including that store method, so when you create a new DAO you never have to worry about that.

    public abstract class BaseVHDao<T> implements VHDao<T> {

        @Autowired
        protected PersistenceManagerFactory pmFactory;

        @Override
        public void store(T model) {
            PersistenceManager pm = pmFactory.getPersistenceManager();
            try {
                pm.makePersistent(model);
            } finally {
                pm.close();
            }
        }

        protected List<T> list(VHQuery vhQuery, Object... args) {
            PersistenceManager pm = pmFactory.getPersistenceManager();
            try {
                Query query = pm.newQuery(vhQuery.getClazz());
                if (vhQuery.hasFilter()) {
                    query.setFilter(vhQuery.getFilter());
                }
                if (vhQuery.hasOrdering()) {
                    query.setOrdering(vhQuery.getOrdering());
                }
                if (vhQuery.hasRange()) {
                    query.setRange(vhQuery.getRangeStart(), vhQuery.getRangeEnd());
                }
                List<T> list = (List<T>)query.executeWithArray(args);
                list.size();
                return list;
            } finally {
                pm.close();
            }
        }

        protected T first(VHQuery vhQuery, Object... args) {
            PersistenceManager pm = pmFactory.getPersistenceManager();
            try {
                Query query = pm.newQuery(vhQuery.getClazz());
                if (vhQuery.hasFilter()) {
                    query.setFilter(vhQuery.getFilter());
                }
                if (vhQuery.hasOrdering()) {
                    query.setOrdering(vhQuery.getOrdering());
                }
                query.setRange(0, 1);
                List<T> list = (List<T>)query.executeWithArray(args);
                if (list.size() > 0) {
                    return list.get(0);
                } else {
                    return null;
                }
            } finally {
                pm.close();
            }
        }

    }

As you can see, there are two methods that we can use in the subclass: list() and first(). They get a PersistenceManager, set up a Query based on the VHQuery you pass in, execute the Query based on all the (optional) parameters you give, and then close the PersistenceManager.

That VHQuery object is there to define the filter/ordering/range that I need to enter into the actual JDO Query object; its reason for being is twofold:

- The JDO Query object is created by calling the PersistenceManager, which I won't have access to in the DAO subclass since I want to instantiate it in the base methods
- Prevent a leaky abstraction of requiring the subclass to know about the JDO Query object

I originally had a method that built and returned the Query object that I could then fill in and execute, but I couldn't close the PersistenceManager after executing it, which would have led to annoying memory leaks. This seems nicer.

Here's what a method looks like that just wants to grab a single item from the database:

    public Greeting getLatestByAuthor(User author) {
        VHQuery query = new VHQuery(Greeting.class);
        query.setFilter("author == :author");
        query.setOrdering("date desc");
        return first(query, author);
    }

It just sets up the VHQuery and asks the base class to give it the first result that matches the given query parameters.

And here's one that returns a list:

    public List<Greeting> mostRecent(int count) {
        VHQuery query = new VHQuery(Greeting.class);
        query.setOrdering("date desc");
        query.setRange(0, count);
        return list(query);
    }

As you can see, your DAO methods now have no extraneous lines that don't have anything to do with the query you're trying to execute. That's all handled generically for you, and you don't have to worry about it.

**Note**: I've written this so I can use it with the Appengine datastore, and haven't tried it with an SQL database. JDO is supposed to work just fine with an RDBMS, so this will probably help. I guess I'll find out when I get around to that.

I'm guessing there was already something that does this, because I can't imagine that everyone using JDO is okay with typing all that boilerplate in every method; I couldn't find it, though, and am open to someone pointing me in the right direction. I'm also sure there will be more additions to this DAO abstraction as I find needs -- what do you think I've missed?
