# Hands-on-with-MovieLens


Data set
--------

We will use the MovieLens data set for this hands-on, that is a network of movies, reviewers, and their ratings. We will first import the raw data from the MovieLens 1 million ratings network into a graph using the included groovy script. The script can be run through the gremlin shell to import the data.

- One million ratings from 6000 users on 4000 movies
- [http://grouplens.org/datasets/movielens/1m/](http://grouplens.org/datasets/movielens/1m/)  



Download Apache TinkerPop console application
--------------------------------------------- 

```
cd /desktop
wget http://mirrors.estointernet.in/apache/tinkerpop/3.4.0/apache-tinkerpop-gremlin-console-3.4.0-bin.zip
unzip apache-tinkerpop-gremlin-console-3.4.0-bin.zip
```

- The TinkerPop reference documentation can found from:
- [http://tinkerpop.apache.org/docs/current/reference/](TinkerPop reference documentation)


Staring Gremlin console application
-----------------------------------
Now to run the console application, enter the extracted gremlin console application directory and execute the following command:

```
cd /apache-tinkerpop-gremlin-console-3.4.0-bin/
bin/gremlin.sh
````

You should now see the Gremlin shell start with a nice ascii art!


Download and extract the data
-----------------------------
To download the MovieLens data, execute the commands below. Remember for the sake of neatness we will create a `\tmp` directory within the extracted console application directory. 

```
cd /tmp
wget http://files.grouplens.org/datasets/movielens/ml-1m.zip
unzip ml-1m.zip
```


Creating a graph from raw data
------------------------------

```
gremlin> :load tmp/ml.groovy
==>true
gremlin> graph = TinkerGraph.open()
==>tinkergraph[vertices:0 edges:0]
gremlin> MovieLensParser.load(graph, './tmp/ml-1m')
Adding occupation vertices...
Processing movies.dat...
Processing users.dat...
Processing ratings.dat...
Loading took (ms): 16493
==>null
gremlin> graph.io(IoCore.gryo()).writeGraph("./tmp/movie-lens.kryo")
==>null
```


Loading the graph in the console
--------------------------------
Now you have a serialized graph in kryo format for repeated use. You can start the gremlin shell, read the graph, and are ready to play with it.

```
gremlin> graph = TinkerGraph.open()
==>tinkergraph[vertices:0 edges:0]
gremlin> graph.io(gryo()).readGraph('./tmp/movie-lens.kryo')
==>null
gremlin> g = graph.traversal()
==>graphtraversalsource[tinkergraph[vertices:9983 edges:1012657], standard]
```


Traversing the graph in Gremlin
-------------------------------

Counting the number of vertices/nodes
```
gremlin> g.V().count()
==>9983
```

Counting the number of edges/relationships
```
gremlin> g.E().count()
==>1012657
gremlin> 
```


Perhaps try to do something like find the average rating for a particular movie:


```
g.V().has('movie','name','Toy Story').inE('rated').values('stars').mean()
g.V().has('movie','name','Titanic').inE('rated').values('stars').mean()
```


Get the list of genres:

```
g.V().hasLabel('genre').valueMap()
```

Number of movies in each genre:

```
g.V().hasLabel('genre').as('a','b').select('a','b').by('name').by(inE('genre').count())
```

You can see how many of each type of vertex or edge there is:

```
g.V().label().groupCount()
g.E().label().groupCount()
```

How many users of what age are present OR group the users by their age:
```
g.V().hasLabel("user").groupCount().by("age")
```

Return the list of movies who have at least 11 ratings 
```
g.V().hasLabel('movie').as('a','b').where(inE('rated').count().is(gt(10)))
``` 

If you want the name of the movies in the results, add the values() step:
```
g.V().hasLabel('movie').as('a','b').where(inE('rated').count().is(gt(10))).values('name')
```


What's the year of the oldest and most recent movie in the data set?  

```
g.V().hasLabel('movie').values('year').min()
g.V().hasLabel('movie').values('year').max()
```


How many movies are there for each year?
```
g.V().hasLabel('movie').groupCount().by('year').order(local).by(keys, asc).unfold()
```



For each movie with at least 11 ratings, emit a map of its name and average rating. Sort the maps in decreasing order by their average rating. Emit the ﬁrst 10 maps (i.e. top 10).
```
g.V().hasLabel('movie').as('a','b').where(inE('rated').count().is(gt(10))).select('a','b').by('name').by(inE('rated').values('stars').mean()).order().by(select('b'),decr).limit(10)
```


For programmers that gave Die Hard 5 stars, what are the top 10 other movies that they gave 5 stars?

```
g.V().has('movie','name','Die Hard').as('a').inE('rated').has('stars',5).outV().where(out('occupation').has('name','programmer')).outE('rated').has('stars',5).inV().where(neq('a')).groupCount().by('name').order(local).by(values, desc).limit(local,10).unfold()
```

Reading this from the beginning in English:

> Give me the vertex with the movie name Die Hard.  Take a look at all the edges from users that rated it 5 stars and 
filter the users down by those whose occupation is a programmer.  Take a look at all the movies that those programmers
rated 5 stars.  Filter out 'a' or 'Die Hard' because we already know they rated that 5 stars.  Count those movies
they also rated 5 stars, order them by the count highest to lowest, and limit it to the top ten.  Print each of the
results on their own line.



References
----------

- Presentation by Marko Rodriguez on [The Gremlin Traversal Language](http://www.slideshare.net/slidarko/the-gremlin-traversal-language)
- For the script itself, borrowed from gists from [Marko Rodriguez](https://gist.github.com/okram/d9f158dee789689759da) and [Daniel Kuppitz](https://gist.github.com/dkuppitz/64a9f1ba30ca652d067a)
- Material for Hands-on adopted from [Jeremy Hanna](https://github.com/jeromatron/graph-resources/tree/master/movie-lens)
