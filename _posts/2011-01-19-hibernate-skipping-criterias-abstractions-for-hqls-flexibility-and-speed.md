---
layout: post
title: ! 'Hibernate: Skipping Criteria''s abstractions for HQL''s flexibility and
  speed'
tags:
- Criteria
- Hibernate
- HQL
- Java
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _wp_old_slug: ''
author: sirsean
---
I was writing a task that would run periodically and clear out old records from a database table, so the database doesn't grow unbounded; it's fortunate that we don't need to keep a perfect historical record in this table, and it sees a lot of inserts.

We're using Java/MySQL/Hibernate here.

My first pass used the standard "get them and delete them" pattern that seems so common when using Hibernate's Criteria API.
    
    public List<Record> getByNameBeforeDate(String name, Date before) {
        return getHibernateTemplate().getSessionFactory().getCurrentSession()
            .createCriteria(Record.class)
            .add(Restrictions.eq("name", name))
            .add(Restrictions.lt("recordedAt", before))
            .list();
    }

    ...

    List<Record> recordsToDelete = recordDao.getByNameBeforeDate(name, date);
    for (Record record : recordsToDelete) {
        recordDao.delete(record);
    }

But I quickly became afraid of the way that would perform. I don't have enough data in my development environment to see what would happen when it runs, but it didn't take a whole lot of imagination to see it using a ton of memory, being too slow, and executing quite a few queries.

I wanted something better. I couldn't figure a way around Criteria's limitations, so I went with HQL instead.

    public int deleteByNameBeforeDate(String name, Date before) {
        String hql = "delete from Record where name = :name and recordedAt < :before";
        Query query = getHibernateTemplate().getSessionFactory().getCurrentSession().createQuery(hql);
        query.setString("name", name);
        query.setDate("before", before);
        int rowCount = query.executeUpdate();
        return rowCount;
    }
    
    ...

    int count = recordDao.deleteByNameBeforeDate(name, date);

With the first method, memory use and queries executed were both O(N). With the second method, memory use should be close to constant (depending on what the database does), and queries executed will always be 1.

So don't be afraid to ditch Criteria where it makes sense; HQL can be much more flexible in certain cases. It gets us much closer to what we'd do if we weren't using Java, which is just executing a single SQL query: all the fancy abstractions that Hibernate's Criteria API provides don't help with that.

And remember to pay attention to the memory/speed characteristics of what you write. It can make a big difference.
