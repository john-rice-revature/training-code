- In broad terms, what is a Join?
- How does a shuffle hash join work in Spark?
- How does a broadcast join work in Spark?
- Why are broadcast joins significantly faster than shuffle joins?
- Why can't we always use broadcast joins?

- What is Spark SQL?
- How does Spark SQL relate to the Spark applications we've been writing, using RDDs?
- How does Spark SQL evaluate a SQL query?
- What is the catalyst optimizer?
- Why are there multiple APIs to work with Spark SQL?
- What are DataFrames?
- What are DataSets?
- How are DataFrames and DataSets "unified" in Spark 2.0?
- What is the SparkSession?
- Can we access the SparkContext via a SparkSession?
- What other contexts are superseded by SparkSession?
- What are some data formats we can query with Spark SQL?
- Are DataSets lazily evalauted, like RDDs?
- What are some functions available to us when using DataFrames?
- What's the difference between aggregate and scalar functions?
- How do we convert a DataFrame to a DataSet?
- How do we provide structure to the data contained in a DataSet?
- How do we make a Dataset queryable using SQL strings?
- What is the return type of spark.sql("SELECT * FROM mytable") ?
- How do we see the logical and physical plans produced to evaluate a DataSet?