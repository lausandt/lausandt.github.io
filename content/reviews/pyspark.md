Title:  Data Analysis with Python and PySpark
Date: 2024-08-05
Modified: 2024-08-06
Author: Lau Sandt
Summary: A review of Johnathan Rioux's Data Analysis with Python and PySpark 

At request of my partner, who retrained as a data engineer, I reviewed [Johnathan Rioux's Data Analysis with Python and PySpark](https://www.manning.com/books/data-analysis-with-python-and-pyspark). The easy conclusion is that if you are interested in using Spark with Python, then you should read this book. Rioux is very knowledgeable, especially about PySpark and data analysis.

However good the book is in giving you an introduction to (Py)Spark, the code used in the examples is not that good. For the kind of scripting that is done by your average data scientist or engineer, the basics of writing clean, safer, and understandable code are not that complicated. It follows four basic steps.

1. Type your code.
2. Name your functions clearly.
3. Document your code.
4. Test your code.

Firmly believing that these four steps need to be implemented in code, I took the effort to rewrite the examples from the book in what I consider to be modern Python. Furthermore, I took the opportunity to shine some light on programming concepts that Rioux does not clarify. I believe some theoretical background will enlighten some opaque concepts. Concepts such as eager evaluation that Python uses to [evaluate expressions](https://en.wikipedia.org/wiki/Evaluation_strategy) and lazy evaluation that PySpark uses for those same expressions.  

You can find these notebooks at the following [github page](https://github.com/lausandt/PySparkNotebooks)