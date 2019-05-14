

## Stream
Similarly to iterators, a stream can be traversed only once.

Streams and collections philosophically
For readers who like philosophical viewpoints, you can see a stream as a set of val- ues spread out in time. 
In contrast, a collection is a set of values spread out in space (here, computer memory), 
which all exist at a single point in time—and which you access using an iterator to access members inside a for-each loop.


Using the Collection interface requires iteration to be done by the user (for exam- ple, using for-each); 
this is called **external iteration**. The Streams library, by contrast, uses **internal iteration**—it does the iteration 
for you and takes care of storing the result- ing stream value somewhere; 
you merely provide a function saying what’s to be done.


