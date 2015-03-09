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

###Setting Parameter(StandardScaler)
1. **withMean**:  
將每個instance的資料，轉為該筆資料的mean，並減掉全部資料的mean，初始值為False。

2. **withStd**:  
將資料同除standard devitation，初始值為True。 (spark的std有點問題?)

###Class StandardScaler:
此function可將Vector轉換成Standardization。

1. **fit(dataset)**:  
透過使用者給定的dataset來建立transformation的model，需特別注意的是dataset必須使用spark提供的dense vector。  
	
	使用時機:
	欲將該dataset當作transformation的基底時。

	Example:
	```python
	from pyspark.mllib.feature import Vectors, StandardScaler
	trans = StandardScaler()
	vec = [Vectors.dense(range(3)), Vectors.dense(range(3,6))]
	rdd = sc.parallelize(vec) #translate Vectors to RDD
	model = trans.fit(rdd) #create the standardizer through your data
	model.transform(rdd).collect() #[DenseVector([0.0, 0.4714, 0.9428]), DenseVector([1.4142, 1.8856, 2.357])]
	```

2. **withMean**:
可直接更改withMean的參數。
	
	使用時機:
	欲修改withMean的參數時。

	Example:
	```python
	from pyspark.mllib.feature import Vectors, StandardScaler
	trans = StandardScaler()
	vec = [Vectors.dense(range(3)), Vectors.dense(range(3,6))]
	rdd = sc.parallelize(vec) #translate Vectors to RDD
	model = trans.fit(rdd) #create the standardizer through your data
	model.transform(rdd).collect() #[DenseVector([0.0, 0.4714, 0.9428]), DenseVector([1.4142, 1.8856, 2.357])]

	trans.withMean = True
	model = trans.fit(rdd) #create the standardizer through your data (withMean = True, withStd = True)
	model.transform(rdd).collect() #[DenseVector([-0.7071, -0.7071, -0.7071]), DenseVector([0.7071, 0.7071, 0.7071])]
	```

3. **withStd**:
可直接更改withStd的參數。
	
	使用時機:
	欲修改withStd的參數時。

	Example:
	```python
	from pyspark.mllib.feature import Vectors, StandardScaler
	trans = StandardScaler()
	vec = [Vectors.dense(range(3)), Vectors.dense(range(3,6))]
	rdd = sc.parallelize(vec) #translate Vectors to RDD
	model = trans.fit(rdd) #create the standardizer through your data
	model.transform(rdd).collect() #[DenseVector([0.0, 0.4714, 0.9428]), DenseVector([1.4142, 1.8856, 2.357])]

	trans.withStd = False
	trans.withMean = True
	model = trans.fit(rdd) #create the standardizer through your data (withMean = True, withStd = False)
	model.transform(rdd).collect() #[DenseVector([-1.5, -1.5, -1.5]), DenseVector([1.5, 1.5, 1.5])]
	```

###Setting Parameter(HashingTF):
1. numFeatures: Features的數量，建立hashing table時所需的參數，基本上代表hashing table的大小，初始值為1048576。

###Class HashingTF:
此function提供一個內建的hashing trick，能提供使用者對feature做hashing。

1. **indexOf(term):
產生該資料hashing後的index。

	使用時機:
	欲取得該資料hashing後的index。

	Example:
	```python 
	from pyspark.mllib.feature import HashingTF
	htf = HashingTF(numFeatures = 100)
	htf.indexOf('a') #44
	htf.indexOf('b') #31
	```
2. **transform(data)**:
將輸入的轉換成hashing table相對的hashing code出現的頻率。
	
	使用時機:
	欲查詢該data在該hashing table內是否有個別hash成功，而沒有產生collision。

	Example:
	```python
	from pyspark.mllib.feature import HashingTF
	htf = HashingTF(numFeatures = 10)
	data = ['a','b','f','g']
	table = htf.transform(data) #SparseVector(10, {1: 2.0, 4: 2.0})
	sum(table.toArray != 0) # 2, 總和的hashing數量只有2種，但事實上資料是4筆不同的值，所以可看出產生collision

	htf = HashingTF(numFeatures = 100)
	table = htf.transform(data) #SparseVector(100, {31: 1.0, 44: 1.0, 54: 1.0, 71: 1.0})
	sum(table.toArray != 0) # 4, hashing的數量與4種不同的值相符，故此table大小不會讓該筆資料產生collision
	```


