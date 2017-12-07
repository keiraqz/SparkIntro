## Intro to Spark


### Presentation Slides

- Thanks to [Amy](https://github.com/eastcoastdaffodil) for making the slides, which can be downloaded from here:
[https://github.com/keiraqz/SparkIntro/blob/master/wwc_sparkintro.pdf](https://github.com/keiraqz/SparkIntro/blob/master/wwc_sparkintro.pdf)


### Install Spark on Local Machine

1. Set the JAVA_HOME environment variable pointing to a Java installation

	Run ```echo $JAVA_HOME```.


	```
	export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_72.jdk/Contents/Home
	```
	Note: If you don't have Java, you will need to download it, Java 7 or above prefered.


2. OPTION 1: Download spark source code (will require Maven build manager)

	Download spark: [here](http://spark.apache.org/downloads.html)

	Click on: "Download Spark: spark-1.6.1.tgz"

	Unzip Spark to whatever directory you like.

	```
	cd ~/Downloads
	tar -xvzf spark-1.6.1.tgz -C ~
	cd ~/spark-1.6.1
	```

	Add Spark Home to Sys env

	```
	export SPARK_HOME=~/spark-1.6.1
	```

  OPTION 2: Download pre-built Spark via Hadoop version

  Use ```hadoop version``` to determine which package type to download.

  Unzip the pre-built Spark (direct download make take longer for pre-built spark). The below assumes Hadoop 2.6 and newer - be sure to replace that version with your system's own version, if different.

  ```
  cd ~/Downloads
  tar -xvzf spark-1.6.1-bin-hadoop2.6.tgz -C ~
  cd ~/spark-1.6.1-bin-hadoop2.6
  ```

  Add Spark Home to Sys env

  ```
  export SPARK_HOME=~/spark-1.6.1-bin-hadoop2.6
  ```


3. Start Pyspark Shell

	```
	cd $SPARK_HOME
	./bin/pyspark
	```

  In starting the PySpark shell, if you run into errors initializing spark context, you may need to add the following lines to your `$SPARK_HOME/conf/spark-env.sh` file:

  ```
  export  SPARK_MASTER_IP=127.0.0.1
  export  SPARK_LOCAL_IP=127.0.0.1
  ```


### Spark Example

1. Word count

	- Create a text file

		```
		touch example.txt
		```

	- Copy into the text file:

		```
		hello.
		this is about the life, universe and everything.
		life is good.
		universe is magnificent.
		everything is awesome.
		```

	- Go to Spark: ./bin/pyspark

		```python
		# load the file
		textFile = sc.textFile("example.txt")

		# create one RDD per line
		textFile.count() # should return 5

		# print the first RDD (first line)
		textFile.first()

		# print the first couple lines (maybe 3 lines?)
		textFile.take(3)
		```

		Now do something a little more complicated. Count the total word in this doc

		```python
		# .map: pass a lambda function to each line that returns the length of the line
		rdd_1 = textFile.map(lambda line: len(line.split()))
		rdd_1.first()

		# .reduce: take two RDD and perform some operation, in this case, add up the length of two lines
		result = rdd_1.reduce(lambda a, b: a+b)
		print result
		```

		Now do something even more complicated. Count the occurance of each word in this doc

		```python
		# remove the symbols in each line, split each line and put everything together
		import re
		wordCounts_1 = textFile.flatMap(lambda line: re.sub("[^a-zA-Z]+", " ", line).split())
		wordCounts_1.take(10)

		# map word to a key-value pair: word => (word, 1)
		wordCounts_2 = wordCounts_1.map(lambda word: (word, 1))
		wordCounts_2.take(10)

		# for a given key, perform (a+b) for all values in the key group
		# it's like group by "key" and "count"
		result = wordCounts_2.reduceByKey(lambda a, b: a+b)
		result.collect()
		```


2. Pyspark SQL

	- Create one json file

		```
		touch veggies.json
		```

	- Copy into this json file

		```json
		{"name": "kale", "count": 3}
		{"name": "avocado", "count": 2}
		{"name": "tomato", "count": 3}
		```

	- Create another json file

		```
		touch fruits.json
		```

	- Copy into this json file

		```json
		{"name": "apple", "count": 1, "color": "red"}
		{"name": "banana", "count": 3, "color": "yellow"}
		{"name": "avocado", "count": 5, "color": "green"}
		{"name": "tomato", "count": 2, "color": "orange"}
		```
		note: we can't agree on whether tomato and avocado are veggie or fruit

	- Go to Spark: ./bin/pyspark

		```python
		# create sqlContext
		from pyspark.sql import SQLContext
		sqlContext = SQLContext(sc)

		# import json files
		df_veggie = sqlContext.read.json("veggies.json")
		df_veggie.show()
		df_fruit = sqlContext.read.json("fruits.json")
		df_fruit.show()

		# check out a column
		df_veggie.select("name").show()

		# filter by column value
		df_fruit.filter(df_fruit['count'] > 2).show()
		```

		Join two tables

		```python
		# show everything
		df_everything = df_veggie.join(df_fruit, df_veggie.name == df_fruit.name, 'outer')
		df_everything.show()

		# avocado, tomato - are you veggie or fruit
		df_unclear = df_veggie.join(df_fruit, df_veggie.name == df_fruit.name, 'inner').drop(df_veggie.name)
		df_unclear.show()

		# get the total counts
		df_unclear_total = df_veggie.join(df_fruit, df_veggie.name == df_fruit.name, 'inner') \
								.drop(df_veggie.name) \
								.withColumn("total_count", df_fruit['count'] + df_veggie['count'])
		df_unclear_total.show()
		```

