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

###Setting Parameter(StandardScalerModel):
None  
###Class StandardScalerModel:
此function為StandardScaler所產生的model，此model才可將資料standarize。

1. **transform(vector)**:
透過StandardScaler設定的參數和資料產生的model，將資料standarize。

	使用時機:  
	欲將資料standarize。

	Example:
	```python
	from pyspark.mllib.feature import Vectors, StandardScaler
	trans = StandardScaler()
	vec = [Vectors.dense(range(3)), Vectors.dense(range(3,6))]
	rdd = sc.parallelize(vec) #translate Vectors to RDD
	model = trans.fit(rdd) #create the standardizer through your data
	model.transform(rdd).collect() #[DenseVector([0.0, 0.4714, 0.9428]), DenseVector([1.4142, 1.8856, 2.357])]
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
1. **numFeatures**: Features的數量，建立hashing table時所需的參數，基本上代表hashing table的大小，初始值為1048576。

###Class HashingTF:
此function提供一個內建的hashing trick，能提供使用者對feature做hashing。

1. **indexOf(term)**:
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

###Setting Parameter(IDFModel):
None  
###Class IDFModel:
此function為IDF所產生的model，此model可產生TF-IDF。

1. **transform(vector)**:
透過IDF設定的參數和資料產生的model，將TF轉換成TF-IDF。

	使用時機:  
	欲算出資料的TF-IDF。

	Example:
	```python
	n = 4
	freqs = [Vectors.sparse(n, (1, 3), (1.0, 2.0)),Vectors.dense([0.0, 1.0, 2.0, 3.0]),Vectors.sparse(n, [1], [1.0])]
	data = sc.parallelize(freqs)
	idf = IDF()
	model = idf.fit(data)
	tfidf = model.transform(data)
	tfidf.collect() #[SparseVector(4, {1: 0.0, 3: 0.5754}), DenseVector([0.0, 0.0, 1.3863, 0.863]),SparseVector(4, {1: 0.0})]
	```

###Setting Parameter(IDF):
1. **minDocFreq**: 此參數為有效出現次數的最小值，意指word或vector要出現超過最小值，才是有效的。

###Class IDF:
透過使用者給定的TF來建立TF-IDF的transformation model。

1. **fit(dataset)**:
此function是透過使用者給定的TF用來建立model。

	使用時機:
	以給定的資料(TF)為標準產生的TF-IDF轉換model。

	Example:
	```python
	n = 4
	freqs = [Vectors.sparse(n, (1, 3), (1.0, 2.0)),Vectors.dense([0.0, 1.0, 2.0, 3.0]),Vectors.sparse(n, [1], [1.0])]
	data = sc.parallelize(freqs)
	idf = IDF()
	model = idf.fit(data)
	tfidf = model.transform(data)
	tfidf.collect() #[SparseVector(4, {1: 0.0, 3: 0.5754}), DenseVector([0.0, 0.0, 1.3863, 0.863]),SparseVector(4, {1: 0.0})]
	```seVector(4, {1: 0.0})]
	```

2. **minDocFreq**:
更改minDocFreq此參數。
	
	使用時機:
	欲更改minDecFreq的值。

	Example:
		```python
	n = 4
	freqs = [Vectors.sparse(n, (1, 3), (1.0, 2.0)),Vectors.dense([0.0, 1.0, 2.0, 3.0]),Vectors.sparse(n, [1], [1.0])]
	data = sc.parallelize(freqs)
	idf = IDF()
	model = idf.fit(data)
	tfidf = model.transform(data)
	tfidf.collect() #[SparseVector(4, {1: 0.0, 3: 0.5754}), DenseVector([0.0, 0.0, 1.3863, 0.863]),SparseVector(4, {1: 0.0})]
	#Change minDocFreq to 2
	idf.minDocFreq = 2
	model = idf.fit(data)
	tfidf = model.transform(data)
	tfidf.collect() #[SparseVector(4, {1: 0.0, 3: 0.5754}), DenseVector([0.0, 0.0, 0.0, 0.863]),SparseVector(4, {1: 0.0})]
	```

###Setting Parameter(Word2Vec):
None
###Class Word2Vec:
此function是將文字向量化，首先會先透過使用者給的詞庫或文章建立字典，而此種向量表示法常被應用在自然語言處理和機器學習上。
在spark中，使用skip-gram的方式來建立model。

1. **fit(data)**:
透過使用者給的文章或詞庫，來建立向量化的字典。
	
	使用時機:
	建立向量化的字典。

	Example:
	```python
	from pyspark.mllib.feature import Word2Vec

	sentence = ["cat cut cute cot cet " * 10]
	doc = sc.parallelize(doc).map(lambda line: line.split(' '))

	model = Word2Vec().fit(doc)
	model.findSynonyms("cut", 2) #[(u'cute', 0.077099844813346863), (u'cet', 0.067038781940937042)]
	```

2. **setLearningRate(learningRate)**:
設定Learning Rate大小，初始值為0.025。

	使用時機:
	欲修改learning rate的大小。

	Example:
	```python
	from pyspark.mllib.feature import Word2Vec

	sentence = ["cat cut cute cot cet " * 10]
	doc = sc.parallelize(doc).map(lambda line: line.split(' '))

	word2vec = Word2Vec()
	word2vec.setLearningRate(0.01)
	model = word2vec.fit(doc)
	model.findSynonyms("cut", 3) #[(u'cute', 0.20835110545158386), (u'cot', 0.0087498482316732407), 
	(u'cet', 0.0081800231710076332)]
	```	
3. **setNumInterations(numInterations)**: 
設定Interation得值，初始值為1。
	
	使用時機:
	設定Interation的上限。

	Example:
	```python
	from pyspark.mllib.feature import Word2Vec

	sentence = ["cat cut cute cot cet " * 10]
	doc = sc.parallelize(doc).map(lambda line: line.split(' '))

	word2vec = Word2Vec()
	word2vec.setNumInterations(10)
	model = word2vec.fit(doc)
	model.findSynonyms("cut", 3) #[(u'cet', 0.98964089155197144), (u'cot', 0.96385848522186279), 
	(u'cat', 0.82072669267654419)]
	```

4. **setNumPartitions(numPartitions)**:
設定資料切割的區塊，資料若越大，可考慮用較大的partitions。
	
	使用時機:
	設定切割的區塊大小。

	Example:
	```python
	from pyspark.mllib.feature import Word2Vec

	sentence = ["cat cut cute cot cet " * 100]
	doc = sc.parallelize(doc).map(lambda line: line.split(' '))

	word2vec = Word2Vec()
	#資料量大時，partition的大小設定才會比較有效益。
	word2vec.setNumPartitions(2)
	model = word2vec.fit(doc)
	model.findSynonyms("cut", 3) #[(u'cet', 0.98814201354980469), (u'cot', 0.96120059490203857),
 	(u'cute', 0.94603407382965088)]
	```

5. **setSeed(seed)**:
設定隨機取樣的種子。
	
	使用時機:
	設定隨機取樣的種子。

	Example:
	```python
	from pyspark.mllib.feature import Word2Vec

	sentence = ["cat cut cute cot cet " * 100]
	doc = sc.parallelize(doc).map(lambda line: line.split(' '))

	word2vec = Word2Vec()
	
	word2vec.setSeed(12L)
	model = word2vec.fit(doc)
	model.findSynonyms("cut", 3) #[(u'cet', 0.97919058799743652), (u'cot', 0.96118772029876709), 
	(u'cat', 0.94353121519088745)]

	word2vec.setSeed(32L)
	model = word2vec.fit(doc)
	model.findSynonyms("cut", 3) #[(u'cet', 0.98944127559661865), (u'cot', 0.96248012781143188), 
	(u'cat', 0.93806272745132446)]
	```


6. **setVectorSize(vectorSize)**:
設定輸出的向量化矩陣的size，越複雜的詞庫或文章，可用較大的size，初始值為100。

	使用時機:
	設定輸出的向量化矩陣的size。

	Example:
	```python
	from pyspark.mllib.feature import Word2Vec

	sentence = ["cat cut cute cot cet " * 100]
	doc = sc.parallelize(doc).map(lambda line: line.split(' '))

	word2vec = Word2Vec()
	word2vec.setVectorSize(10)
	
	model = word2vec.fit(doc)
	vec = model.transform("cut") #DenseVector([0.3652, 0.0744, -0.4375, -0.0526, 0.079, -0.0049, 0.3399, -0.0595, -0.2702, -0.1888])

	#透過向量化的矩陣，還原word，並找尋同義字。 
	model.findSynonyms(vec, 2) #[(u'cet', 0.96490895748138428), (u'cot', 0.81680792570114136)]
	```

###Setting Parameter(Word2VecModel):
None
###Class Word2VecModel:
此為已經training完成的model，由Word2Vec的fit training後得到Word2VecModel，提供findSynonyms及transform的function。

1. **findSynonyms(word, num)**:
從dictionary內找尋相似字。

	使用時機:
	從dictionary找尋與傳入的word之相似字。

	Example:
	```python
	from pyspark.mllib.feature import Word2Vec

	sentence = ["cat cut cute cot cet " * 10]
	doc = sc.parallelize(doc).map(lambda line: line.split(' '))

	word2vec = Word2Vec()
	word2vec.setLearningRate(0.01)
	model = word2vec.fit(doc)
	model.findSynonyms("cut", 3) #[(u'cute', 0.20835110545158386), (u'cot', 0.0087498482316732407), 
	(u'cet', 0.0081800231710076332)]
	```	

2. **transform(word)**:
將word透過training好的model向量化。

	使用時機:
	欲將word透過training好的model轉換成向量表示。

	Example:
	```python
	from pyspark.mllib.feature import Word2Vec

	sentence = ["cat cut cute cot cet " * 100]
	doc = sc.parallelize(doc).map(lambda line: line.split(' '))

	word2vec = Word2Vec()
	word2vec.setVectorSize(10)
	
	model = word2vec.fit(doc)
	vec = model.transform("cut") #DenseVector([0.3652, 0.0744, -0.4375, -0.0526, 0.079, -0.0049, 0.3399, -0.0595, -0.2702, -0.1888])
	```
