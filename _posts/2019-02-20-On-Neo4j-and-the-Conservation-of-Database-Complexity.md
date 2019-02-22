---
layout: post
title: On Neo4j and the Conservation of Database Complexity
tags: neo4j databases
---

There is a lot to love about Neo4j, especially if you are modeling data that has a high relationship density
and highly heterogenous relationship types, patterns, and structures.  

Social networks are often used as a prime example here
because they're familiar and easy to motivate graph databases in general.  If we think in terms of a "Person" table,
then one can imagine all kinds of zany, heirarchical, and recursive relationships amongst its rows.  The diversity in
the patterns of relationships is matched by the sheer number of relationship types that may exist.  Do you
create a bazillion bridge tables?  Do you denormalize like crazy?  From a database user's perspective, the SQL
queries can get truly unruly (whoa, nice rhyme!).  From an internal perspective, excessive table scans
can drive down the performance.  

The major sell of Neo4j is getting rid of all this icky complexity... But in this post, I ask: have they
been successful?  Or have they merely shifted where the complexity resides?  After several months of 
tinkering, I wonder if there some fundamental conservation law at play.  Is database complexity like a game
of Wack-a-Mole?  

**Word of Warning**:  I'm a STEM mutt, not a pure bred DBA.  Tread carefully.


# I don't think that query means what you think it means
Joins get a lot of flak from NoSQL folks.  Well, maybe not the simple joins -- i.e., joins that are just "one join 
deep."  But "analytical" joins certainly do -- you know, inception joins (joins within joins within joins) that a
data scientist might string together to generate an insightful data product.  There
have certainly been times where I wonder if a complex query is doing what I think it's doing, which can require
careful analysis of the query and the result set. 

Graph databases like Neo4j proclaim freedom from this nightmare, but I've also experienced a similar 
pit in my stomach with some simple Cypher queries.

For example: be careful with MERGE.  I know I've written about this before, but I have found it to be easy
to slip into query patterns that don't do what you think they're doing.  

For example, say you created a "Tool" node and "Job" node like so:
```
CREATE (:Tool {name: "Hammer"});
CREATE (:Job {name: "Build Backyard Deck"});
```

Then, what do you think this query does:
```
MERGE (job:Job {name: "Build Backyard Deck"})-[:REQUIRES]->(:Tool {name: "Hammer"});
```

Answer:  It looked to see if this pattern existed in the database.  It didn't.  So
this pattern was created -- from scratch.  Importantly, it didn't  look to see if parts of
the pattern existed, then only create what was needed.  You now have two "Tool" nodes named
"Hammer" and two "Job" nodes named "Build Backyard Deck".  

What you have to do if first MATCH on the nodes that you've already CREATED and will be using in the pattern, assign
them a variable name, then MERGE against the pattern.  Neo4j will know not to create the nodes
that it already matched against:
```
MATCH (tool:Tool {name: "Hammer"})
MATCH (job:Job {name: "Build Backyard Deck"})
MERGE (job)-[:REQUIRES]->(tool);
```

Moral:  Remember that MERGE means "MATCH OR CREATE" and that it operates against an entire pattern at once.

In this particular case, we could do away with MATCH and CREATE statments altogether:
```
MERGE (tool:Tool {name: "Hammer"})  // MATCH or CREATE this :Tool node
MERGE (job:Job {name: "Build Backyard Deck"})  // MATCH or CREATE this :Job node
MERGE (job)-[:REQUIRES]->(tool)  // MATCH or CREATE this :REQUIRES relationship
```

That all said, sometimes you don't want that behavior.  Sometimes you do want to create additional
nodes.  And if confusion at the syntactic level did not motivate it enough, I think this is really
where the hypothesis for "conservation of database complexity" comes in.  It all has to do with
keys and constraints, or the lackthereof.


# Are Keys and Constraints More or Less Complex?

In a relational database, a table has a primary key -- a unique way of identifying a row,
which represents some tuple of information representing some object.  Sometimes there is a column
that naturally lends itself to this, but more often than not (in the databases I've used) an ID column 
is used to ensure uniqueness.  Other times, a table's primary key is composite.  Point is, the idea of 
including IDs to ensure unique rows is pretty common...  And consistent.  Normalizing a database, 
or deciding on a denormalized
solution, can be daunting if you require hordes of foreign keys and bridge tables, but it's a battle that has been
fought and won many times over.  It's routine.  The complexity is understood and documented.

*\<SalesmanTone>  With Neo4j, you can develop at the speed of thought: your whiteboard 
model is your logical model.  No need to deal with all those bridge tables and foreign keys: throw 
'em away and get rid of that nasty complexity!  In Neo4j, relationships are first-class citizens.
Need a new relationship between entities?  No worries:  just do it!  No need to create a new bridge 
table.  In fact, no need to do anything at all: this is schema optional, pedal-to-floor development, 
baby!\<endSalesmanTone>*


It seems so simple, but from this simplicity, complexity seeps in.  For example, 
making things up as you go along is awesome until it's not.  In fact, this is not really even a dig
at Neo4j: they will be the first ones to tell you that "schema optional" doesn't mean you shouldn't 
design a great schema.  It's optional, and you should opt to do it!

It is because of the flexibility and the lack of constraints that you might find yourself
reinventing standard features of the relational wheel at the application layer.  For example,
check constraints:  Neo4j will not enforce any rule about what values a property of a node may take on
(note that a property of a node is the graph version of a column in a table) . In fact, Neo4j will not even 
ensure that a property of a node will always be of the same data type, like integer or string.  If
you want these types of constraints, you have enforce them  yourself at the application layer(s).
  

This is touching on what I mean by conservation of complexity: someone somewhere is going to deal
with it.  That may be the database administrator, the application designer, or the data scientist
at the end of the pipeline.  

# More on MERGE and Bridge Tables
## The Relational Storyline
In a relational database, you will might have table T1 and table T2, both having their own primary
key (column with ensured uniqueness).  If the objects (rows) in T1 have a many-to-many relationship
with the objects in T2, then a bridge table TB must be created such that T1-TB and T2-TB both have
many-to-one relationships.  The primary key of the bridge table is a composite key of the foreign
keys from T1 and T2.  

Let's make this less abstract.

Let T1 be a table called ArticleOfClothing.  And let T2 be a table called Color.  We know that
the relationship between clothes and color is many-to-many because (i) "jeans" may come in many
different colors, and (ii) "blue" might be the color of many different articles of clothing.

```
# Many-to-Many Relationship
 ___________________            _____________
| ArticleOfClothing |          | Color       |
|-------------------| \      / |-------------|
| a1 | Jeans        | - --- -  | c1 | Red    |
| a2 | T-Shirt      | /      \ | c1 | Green  |
| a3 | Socks        |          | c1 | Blue   |
 -------------------            -------------
```

To set up a relationship between tables, we want to use a foreign key...but we can't.  That is, what
color key should we put in the a1 row of ArticleOfClothing?  We already said that an article of clothing
can take on any color, so there is no single choice.  This is where accidental denormalization can 
occur, i.e., a design choice here can easily break a Normal Form (e.g., using two foreign key columns 
to represent color choices in the ArticleOfClothing table, or using multiple rows and thus repeating
the article name).  The proper thing to do here is to resolve the many-to-many relationship by creating
a bridge table.

```
 # (One-to-Many) BRIDGE (Many-to-One)
  ___________________        _____________         _____________
| ArticleOfClothing |       | AoCHasColor |       | Color       |
|-------------------| \     |-------------|     / |-------------|
| a1 | Jeans        | - --- |  a1 | c1    | ----  | c1 | Red    |
| a2 | T-Shirt      | /     |  a1 | c2    |     \ | c1 | Green  |
| a3 | Socks        |       |  a2 | c2    |       | c1 | Blue   |
 -------------------        |  a2 | c3    |        -------------
                            |  a3 | c1    |
                             -------------
```

Now we can maintain a "clothing dictionary", a "color dictionary", and a clothing/color relationship table.  We
can be assured that (i) any article of clothing will be uniquely represented, (ii) any color will be uniquely
represented, and (iii) and clothing/color combo will be uniquely represented.  

## The Property Graph Storyline


---------------

......STILL A WORK IN PROGRESS....

What's my point?  I guess I'm getting to it:  there is some complexity hidden in MERGE.  It has to do
with constaints, IDs, creation, etc.  

For the moment, I think it makes sense to think of the graph database like a relational database:
consider nodes of a certain label L1 to be rows in a table called "L1" and other nodes of label L2
to be rows in a table called "L2".  Say L1 is :Author and L2 is :Book.  Assume an author's name is 
unique, but a book's title is not -- but that a book's title and the book's author do make a unique 
combo.  Then the author's name would be a primary key in the "Author" table and a foreign key
in the "Book" table, which would have composite primary key (author's name and book title).  Point is,
placing a uniqueness constraint on both tables is easy, straigtforward, and routine.

What about the Neo4j version of this?  Well you can create a uniqueness constraint on the author's name. But
what about the :Book label nodes?  If you want to create a uniqueness constraint, then you must include
the author's name -- correctly.  The database won't exactly use a foreign key to ensure this... You have to ensure
it yourself.  The other possibility, also at your own application layer, is using MERGE...  This will ensure
that a book title is always associated with an author, and unique for the author.  However, this still isn't the same:
if you issued a CREATE statement in the relational case, it would get rejected: MERGE will MATCH or CREATE, and 
on MATCH will update the data as you specify.  It won't get rejected...


-----------------------------

I guess it's true for any language or technology, but you really have to know what you're doing.  

Another example:  say you want to MATCH something, then update it.  If you use MERGE and it doesn't exist,
it will be created and updated.  If you use Merge and it does exist, it will just be updated.  If use MATCH
and its exists, again it will be updated.  But if you use MATCH and it doesn't exist, nothing will be
updated -- but you also won't be notified.  There is no error or hiccup of any kind.  
