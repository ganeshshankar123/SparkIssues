# SparkIssues

Data Skewness:-
Let's say some key is getting repeated across the dataset i.e key is not distributed uniformly 

Data Skewness impact:-
1)Performance of Spark Job is impacted
2)Jobs runs for longer time than usual
3)Resouces allocated are not utilised
4) Most of the resoucres will be idle without performing any task
5)Although spark is used in distributed processing ,we will not get advantage of distribution processing

Data Skewness can be resolved:
1)Repartition:
Increasing the number of partition for data distribution based on the key column

2)Salting:
Let's say some key is getting repeated across the dataset i.e key is not distributed uniformly .So in this case we add some prefix or some suffix to original key to make it distributed across other partitions
