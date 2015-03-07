#MLlib - Feature
這是由MLlib所提供的library，主要提供多種實用方法來處理feature，如常見的Normalization及StandardScaler，讓使用者能不必將資料處理好，才轉成RDD，有了這個library，即可直接讓資料以RDD的型態處理。

###Setting Parameter(Normalizer):
1. **p**:  p代表使用 p norm作為normalization的基底，將所有資料同除sum(abs(vector) p) (1/p)
	     所得到的結果，即為normalize後，p初始值為2。

###Class Normalizer
Normalizer提供的normalization是以Lp norm為基底，將所有資料同除Lp norm。
1. **transform(vector)**:
此function可將輸入的Vector，透過設定好的normalizer來normalize，Vector可搭配mllib-feature提供的Vectors使用，用一般的Vector也可以。

	使用時機:
	欲得到資料或Vector的Normaliztion。

	Example:
	```python
	from pyspark.mllib.feature import Normalizer as norm
	from pyspark.mllib.feature import Vectors
	nor = norm(p = 1)
	vec = Vectors.dense(range(4))
	nor.transform(vec) #DenseVector([0.0, 0.1667, 0.3333, 0.5])
	#Change p to 2
	nor = norm(p = 2)
	vec = range(4)
	nor.transform(vec) #DenseVector([0.0, 0.2673, 0.5345, 0.8018])
	```
2. **p**:  
可直接更改normalizer的p。

	使用時機: 
	欲修改normalizer的p。

	Example: 
	```python
	from pyspark.mllib.feature import Normalizer as norm
	from pyspark.mllib.feature import Vectors
	nor = norm(p = 1)
	vec = Vectors.dense(range(4))
	nor.transform(vec) #DenseVector([0.0, 0.1667, 0.3333, 0.5])
	nor.p = 2
	vec = range(4)
	nor.transform(vec) #DenseVector([0.0, 0.2673, 0.5345, 0.8018])
	```

 Setting Parameter(StandardScaler)

