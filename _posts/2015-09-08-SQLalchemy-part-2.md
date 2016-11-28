---
layout: post
title: Creating and populating a database using Python and SQLalchemy.
    Part 2&#58; Classes and queries
category: programming
tags: [Python, SQL, SQLalchemy]

---

Last month I wrote a post on
[the SQLalchemy engine and session](http://elsander.github.io/2015/08/04/SQLalchemy-part-1.html). Now
I'm going to describe how you can set up a mapping for your schema so
that you can populate and query your database.

# Setting up your Schema

Setting up your schema correctly is what will allow you to get the
most out of SQLalchemy. I'll assume you've already set up your
session and that it's stored in the object `session`. After this, you'll
want to import some functions: 

	from sqlalchemy.ext.declarative import declarative_base
	from sqlalchemy import Column, ForeignKey, Integer, String, Float, Boolean
	from sqlalchemy import Index
	from sqlalchemy.orm import relationship, backref

	Base = declarative_base()

All I've done here besides importing is to set up `Base`. This is a
basic table class provided by SQLalchemy. When you set up your own
table classes, they will inherit from `Base`, so that many basic table
methods will automatically be available to you.

Continuing with my example from the previous post, I'll be using code
snippets that I used to convert the [Yelp
academic dataset](https://www.yelp.com/academic_dataset) from JSON
files to a PostgreSQL relational database. Here is what a basic table
class looks like:

	class Restaurant(Base):
		__tablename__ = 'restaurant'
		restaurant_id = Column(String(250), index = True, primary_key = True)
		ages_allowed = Column(String(250))
		price_range = Column(Integer)

This code is fairly readable; I've told SQLalchemy to call this table
`restaurant`, and I've given it the names and types of a few
columns. The `restaurant_id` column is the primary key for this table,
and I've also created an index for the column, to make queries more efficient.

# Basic Table Relationships

If your schema includes multiple tables, you will probably want to
establish relationships between them. For example, I also created a
table of restaurant reviews, called `review`, using a class much like
the one above, called `Review`. This is a
many-to-one relationship, since a restaurant may have multiple
reviews, but a review will only be associated with a single
restaurant. If I want to establish this relationship through the ORM,
my `Restaurant` class will have an additional line:

	class Restaurant(Base):
		__tablename__ = 'restaurant'
		restaurant_id = Column(String(250), index = True, primary_key = True)
		ages_allowed = Column(String(250))
		price_range = Column(Integer)
        reviews = relationship('Review', backref = 'restaurant')

This last line creates a `.reviews` attribute for `Restaurant`. The
`backref` argument also creates a `.restaurant` attribute for class
`Review`. This is syntactic sugar that allows me to set up the whole
relationship in this line, without specifying the relationship
in the class `Review`.

Many to many relationships are slightly more complicated. In my
database, a restaurant may be associated with many categories (Cafe,
Italian, Chinese Food, etc.), and each category will be associated
with many restaurants. This means that I need to set up a table for
categories, and a junction table that joins the restaurant and
category information. I also need to establish relationships between
the restaurant/category tables and the junction table, as follows:

	class Restaurant(Base):
		__tablename__ = 'restaurant'
		restaurant_id = Column(String(250), index = True, primary_key = True)
		ages_allowed = Column(String(250))
		price_range = Column(Integer)
		categories = relationship('Category', secondary = 'restaurant_category')

	class Category(Base):
		__tablename__  = 'category'
		category_id = Column(Integer, primary_key = True)
		restaurants = relationship('Restaurant', secondary = 'restaurant_category')
		name = Column(String(250), nullable = False)

	class Restaurant_Category(Base):
		__tablename__ = 'restaurant_category'
		category_id = Column(Integer, ForeignKey('category.category_id'),
		                     primary_key = True)
		restaurant_id = Column(String(250), ForeignKey('restaurant.restaurant_id'),
							   primary_key = True)

The gist of this is that I set up a separate junction table in
SQLalchemy, which specifies that its columns are foreign keys, and
that they form a composite primary key for the table (by setting
`primary_key = True` for both columns). I also set up a relationship
for both `Restaurant` and `Category`, telling SQLalchemy that this
relationship is specified by a secondary table, in this case
`restaurant_category`.

# Populating and Querying your Database

To create these tables in your database of choice, take your engine
object and run:

    Base.metadata.create_all(engine)

Populating the database is pretty easy, once you've set up the
schema. Adding a category is as simple as:

    category = Category(name=item)
    session.add(category)

If you want to add a restaurant and link it to the category, you
can append the category to the `Restaurant` object:

    restaurant = Restaurant(restaurant_id = 'ABC123', price_range = 1)
    restaurant.categories.append(category)
    session.add(restaurant)
	session.commit()

Querying tables with the ORM is a powerful way to work with your
database, but it takes some getting used to.

    category = session.query(Category)

A basic query on a class like the one above is equivalent to the SQL
`SELECT * FROM category`. Note that it returns a query object, so that
the query can be refined over multiple lines.

	category = category.filter(Category.name == 'Cafe')

This line will find all category rows with the name 'Cafe'. Note that
this can also be run as a single line:

    category = session.query(Category).filter(Category.name == 'Cafe')

There are many other ways to filter and adapt your queries, many of
which are listed in
[this tutorial](http://docs.sqlalchemy.org/en/rel_1_0/orm/tutorial.html#querying). If
you set up logging, you can look at the actual SQL that is being run,
which can help you debug and improve your queries.

The aspect of SQLalchemy I found most confusing is figuring out how to
access actual table information from the query object. There are a few
approaches to this. If there are multiple rows that match your query,
you can iterate over them or put them all in a list:

	for row in category:
		print row.name

    ## this is equivalent to the code above,
    ## but stores each Category object in a list
    cats = category.all()
	for cat in cats:
	    print cat.name

If you only want a single row from your query, you can use
`category.first()` to get the first match. It's important to note that
the query object is giving you rows in the form of a `Category`
object. We can use these objects to take advantage of all of the
relationships we set up before:

	mycategory = category.first()
	## let's get all of the restaurants associated with this category:
	category_rest = mycategory.restaurants
	## what is the first restaurant that matches?
	firstrest = category_rest[0]
	## what are all of the categories associated with this restaurant?
	some_categories = firstrest.categories
	for cat in some_categories:
	    print cat.name

As you can see, you can do some complex and recursive things with
these objects! If you have a complicated schema, the overhead of
setting up the ORM in SQLalchemy is, in my opinion, really worth it. 
