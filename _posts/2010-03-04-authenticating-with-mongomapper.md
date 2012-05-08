---
layout: post
title: Authenticating with MongoMapper
tags:
- Mongo
- MongoMapper
- Ruby
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
For my new [poll4.me](http://poll4.me/) application, I'm using MongoDB and need to be able to connect to the authenticated database in production, but not necessarily in my development environment. I couldn't find anywhere online that shows specifically how to do that, so I'll post it here.

    MongoMapper.connection = Mongo::Connection.new(config['db_hostname'])
    MongoMapper.database = config['db_name']
    if config['db_username']
        MongoMapper.connection[config['db_name']].authenticate(config['db_username'], config['db_password'])
    end

Note that I'm using MongoMapper, which is excellent.

The key here is that I set up a connection using the hostname (or IP address) of the database server, then set the database I want to use. Then _if I've specified authentication parameters_, it attempts to authenticate against the database server. If I haven't specified authentication information, this will assume I want to try to connect unauthenticated (and my database is running in non-authenticated mode).

Pretty simple.
