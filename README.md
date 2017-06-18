#  Scala for data science examples

This repository contains the code examples for [Scala for data science](http://pascalbugnion.net/book.html). The aim of the book is to teach people who know a bit of Scala about useful libraries and tools for writing data science applications.

The examples are divided as follows:

 - Chapter 2 teaches you how to use [Breeze](https://github.com/dlwh/breeze) for linear algebra and function optimization. The example programs show you how to write a basic logistic regression model.
 - Chapter 3 introduces [breeze-viz](https://github.com/scalanlp/breeze/wiki/Quickstart#breeze-viz) for drawing basic two-dimensional plots.
 - Chapter 4 teaches you how to parallelize cross-validation pipelines using parallel collections, and builds a small command line utility for querying a web API in parallel using futures.
 - Chapter 5 introduces how to interact with SQL databases from Scala. We write wrappers around JDBC to allow interacting with it more functionally. We introduce several design patterns commonly used in Scala, including implicit conversions to extend existing libraries and typeclasses.
 - Chapter 6 introduces [Slick](http://slick.typesafe.com), a wrapper around JDBC that maps SQL tables directly to case classes.
 - Chapter 7 teaches you how to interact with web APIs and how to convert JSON objects to Scala classes.
 - Chapter 8 introduces [Casbah](https://mongodb.github.io/casbah/), a library for interacting with [MongoDB](https://www.mongodb.org), a leading NoSQL database.
 - Chapter 9 introduces the [Akka](http://akka.io) framework for building concurrent applications. We build a web crawler that crawls the graph of followers on GitHub.
 - Chapters 10 and 11 introduce [Apache Spark](http://spark.apache.org), a framework for processing batches of data over several different computers.
 - Chapter 12 walks through the construction of a spam filter in MLlib, a machine learning library for training distributed algorithms on large datasets in memory.
 - Chapters 13 and 14 introduce the [Play framework](https://www.playframework.com) for building web applications. In chapter 13, we build a REST API to deliver information about a user's repositories on GitHub, and in chapter 14, we build on this API to build a single-page web application with [D3](d3js.org) graphs.

To run the examples, start by installing [SBT](http://www.scala-sbt.org/release/tutorial/Setup.html), the (main) build tool for Scala projects.

Once you have installed SBT, download this repository using:

    $ git clone https://github.com/pbugnion/s4ds.git

You can then navigate to each chapter and type `sbt run` in a terminal to run the source code, or `sbt console` to open a Scala console with all the dependencies for that particular chapter. Read the README for each chapter for more detail.



## Basic components of *`Breeze`*
    
    import breeze.linalg._

    val v = DenseVector(1.0, 2.0, 3.0)

    v(1)

    v :* 2.0

    v :+ DenseVector(4.0, 5.0, 6.0)

    // DenseVector
    // SparseVector
    // HashVector


    val m = DenseMatrix((1.0, 2.0, 3.0), (4.0, 5.0, 6.0))

    2.0 :* m 


    // Breeze offers several other powerful ways of building vectors and matrixes:
    val v2 = DenseVector.ones[Double](5)

    val v3 = DenseVector.zeros[Int](3)

    // the `linspace` method creates a Double vector of equally spaced values.
    linspace(0.0, 10.0, 10)

    // The `tabulate` method lets us construct vectors and matrices from functions:
    DenseVector.tabulate(4) { i => 5.0 * i }


    // The `rand` function lets us create random vectors and matrices:
    DenseVector.rand(2)

    DenseMatrix.rand(2, 3)

    DenseVector(Array(2, 3, 4))

    // To construct vectors from other Scala collections, you must use the splat operator, `: _*`
    val l = Seq(2, 3, 4)
    DenseVector(l: _*)

    // Advanced indexing and slicing

    val v = DenseVector.tabulate(5) { _.toDouble }  // DenseVector(0.0, 1.0, 2.0, 3.0, 4.0)

    v(-1)                       // last element
    v(1 to 3)                   // DenseVector(1.0, 2.0, 3.0)
    v(1 until 3)                // DenseVector(1.0, 2.0)
    v(v.length-1 to 0 by -1)    // reverse view of v

    val vSlice = v(1, 2)        // SliceVector(1.0, 2.0)
    vSlice.toDenseVector

    val mask = DenseVector(true, false, false, true, true)
    v(mask).toDenseVector       // DenseVector(0.0, 3.0, 4.0)

    val filtered = v(v :< 3.0)  // :< is element-wise "less than"
    filtered.toDenseVector


    val m = DenseMatrix((1.0, 2.0, 3.0), (4.0, 5.0, 6.0))
    m(1, 2)
    m(1, -1)
    m(0 until 2, 0 until 2)
    m(0 until 2, 0)             // DenseVector(1.0, 5.0)

    // The symbol `::` can be used to indicate every element along a particular direction
    m(::, 1)


    // Breeze vectors and matrices are mutable.
    val v = DenseVector(1.0, 2.0, 3.0)
    v(1) = 22.0         // v is now DenseVector(1.0, 22.0, 3.0)

    v(0 until 2) := DenseVector(50.0, 51.0)
    v(0 until 2) := 0.0         // equivalent to v(0 until 2) := DenseVector(0.0, 0.0)


    val v = DenseVector.tabulate(6){ _.toDouble }
    val viewEvens = v(0 until v.lengh by 2)             // DenseVector(0.0, 2.0, 4.0)
    viewEvens := 10.0                                   // DenseVector(10.0, 10.0, 10.0)

    // v is also mutated!!!
    v         // DenseVector(10.0, 1.0, 10.0, 3.0, 10.0, 5.0)

    // A vector slice `v(0 to 6 by 2)` of the `v` vector is just a different view of the array
    // underlying v. 
    // The view itself contains no data. It just contains pointers to the data in the original
    // array. 

    
    val copyEvens = v(0 to v.length by 2).copy          // to create independent copies of data


    //
    // Matrix multiplication, transposition, and the orientation of vectors
    //
    val m1 = DenseMatrix((2.0, 3.0), (5.0, 6.0), (8.0, 9.0))
    val m2 = DenseMatrix((10.0, 11.0), (12.0, 13.0))
    m1 * m2


    // m1 (3 * 2)
    val m1 = DenseMatrix((2.0, 3.0), (5.0, 6.0), (8.0, 9.0))
    m1: breeze.linalg.DenseMatrix[Double] =
        2.0  3.0
        5.0  6.0
        8.0  9.0

    // A vector should be viewed as an (n * 1) matrix
    // v (2 * 1)
    val v = DenseVector(1.0, 2.0)
    v: breeze.linalg.DenseVector[Double] = 
        DenseVector(1.0, 2.0)

    m1 * v
    res10: breeze.linalg.DenseVector[Double] = 
        DenseVector(8.0, 17.0, 26.0)


## Data preprocessing and feature engineering

> An important part of data science involves preprocessing datasets to construct useful
> features.


