---
layout: post
title: Creating and populating a database using Python and SQLalchemy.
    Part 1&#58; Communicating with your database
category: programming
tags: [Python, SQL, SQLalchemy]

---

I'm spending the summer at the [Recurse Center](https://www.recurse.com/), where I'm
working with a group of other awesome programmers to learn and
self-study programming full-time for three months.

My first few weeks here, I've primarily been learning
how to populate a database with Python. Yelp has given me access to
the [Yelp
academic dataset](https://www.yelp.com/academic_dataset), which I
recieved as a JSON file, but to make it
easy to query I wanted to put it in a relational database. I started
off using sqlite3, which is a great module but essentially requires
you to write SQL by hand. Python then passes your queries directly to
the database. This is a big hassle if you want to automate queries,
say, to run a bunch of insert statements.

This is where SQLalchemy comes in very handy. This module is a
powerful object-relational mapper (ORM) that allows the user to
abstract away most of the actual
SQL. Instead, you can define classes for each of your tables, and run
queries using methods and an abstracted 'SQL session'. This has
several advantages. First, and most important to me, it means that you
don't have to build queries by hand, which is much less of a hassle
than sqlite3. For more complicated queries, SQLalchemy does allow you
to write and submit your own SQL statements (I found this especially
useful for debugging). Second, it makes your code less
database-specific. If you're using the object-oriented features of
SQLalchemy, most of your code should work regardless of which
SQL flavor you choose, since all of the language-specific details are
abstracted away. Finally, it's easier to keep your database safe
from injection attacks without hassle, since SQLalchemy handles it
under the hood for you. However, this abstraction and convenience comes with a
fairly steep learning curve, and the documentation is good but
sometimes overwhelming. Hopefully this post will be a useful resource
for anyone hoping to learn the basics!

# Engine and Session setup
There are several layers of abstraction to help you communicate with
your database. The process that communicates between Python and your
database is called the engine. Here's how I created my engine:

```
import logging
from sqlalchemy import create_engine
engine =
create_engine('postgresql://postgres:password@localhost/yelp_restaurant',
echo = True)
```

The logging module is used for the `echo = True` option, which prints
helpful error messages as needed. You can see that I'm using
postgresql, with username `postgres` and password `password` (I've set
up the database on my computer rather than an external server, so I'm
not overly worried about anyone hacking into it). `localhost` is the
host name, so I'm just telling the engine that it can find the
database on my computer. `yelp_restaurant` is what I've named my
database, since I'm going to be storing restaurant and review
information.

If you wanted to deal with the engine directly, you would need to set
up a connection, and communicate via the connection. However, it's
more convenient to use a session, which handles the engine under the
hood. I like to think of the session as temporary version control for your
database queries. The session takes in your SQL, stores it in a safe
place, and submits it all when you tell it to. First we need to create
a session to work with:

```
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind = engine)
session = Session()
```

Here I'm linking a session to the engine and intializing it. From now
on, I can totally ignore the engine and let the session do all of the
communication work for me.

# Using the session

There are four major session methods that I have used so far: query,
add, flush, and commit. As the name suggests, the query method allows
you to query the database, letting SQLalchemy write the SELECT
statements for you. It returns a query object, which can
be updated or refined in future calls. Here is an example from my
code:

```
id = session.query(Restaurant).filter(Restaurant.restaurant_id == val)
id = session.query(id.exists()).scalar()
```

I will talk more about the table classes in my next post, but for now,
know that Restaurant is a class I created for generating and
interacting with a table of restaurants in Yelp. Here I am querying
the restaurant database for all restaurants where the restaurant_id is
equal to a specific value. I assign this query to `id` so that I can
refine this query. Using `exists()` and `scalar()`, I am turning this
query into an EXISTS query, and I'm assigning the output (1 or
0) to `id`.

The other three methods are related to the "session version control" I
mentioned before. The version control analogy breaks down fairly
quickly, since you can't check out old commits or branch, but I found
it to be a useful analogy for distinguishing the different session
methods. If I run `session.add(myrestaurant)`, I am creating
the SQL to add the object myrestaurant (an instance of class
Restaurant) to my table. SQLalchemy stores this as a pending statment,
but does not run it yet. I think of this like staging changes using
`git add`. This is nice because you can update the SQL if you need to,
or you can get rid of pending changes using `session.rollback()`. Note
that these changes are pending on the SQLalchemy side; your database
of choice has not seen your SQL at all.

Flushing a session sends all pending changes to the database, but does
not run them. This means that the changes are still pending, but are
now pending on the database side. It's not exactly equivalent, but I
consider this similar to `git commit`, where you are submitting
changes, but others on the project can't see the effects yet. You can
flush a session yourself using `session.flush()`.

Finally, committing a session is similar to a `git push`, in that it
updates the database to reflect all of the changes you've made on the
SQLalchemy side. When you use `session.commit()`, your SQL commands
are finally run on the database side. This means that your changes
will be permanently made. One thing to note is that committing
automatically flushes any pending changes, so you may not actually
need to flush changes yourself.

It's not necessarily intuitive to know when to flush or commit a
session. My goal is to try to commit sensible chunks at once: if I'm
inserting a bunch of lines, to send one commit for those; if I'm
deleting lines and then querying the database, to explicitly commit
before the query. If you query a session, it will flush anything
pending first, but I find that it's easier to understand what you are
querying if you make it explicit in your code.

In Part 2, I'll discuss how to set up classes for working with your
tables, and how to create, add to, and query these tables using the
architecture you've set up.
