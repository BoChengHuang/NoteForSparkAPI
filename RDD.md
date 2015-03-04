#RDD
RDD(Resilient Distributed Dataset)是Spark的核心技術之一，可以提供平行化運算的環境，讓部份或全部的資料儲存在Memory內，讓運算或處理速度更快，RDD也提供了多種function，hadoop的mapReduce、spark獨特的function
(Ex:mapPartitions、mapPartitionsWithIndex)，可讓programmer有更多方法來處理data。
在Spark內，RDD會配合SparkContext內的一些method使用(Ex: textFile, wholeTextFiles,  parallelize)，這些method最後會回傳RDD的格式，讓programmer能使用RDD來處理資料。

###Setting Parameter:
(No parameter need to be set)


###Class RDD:   
1. **id (self)**: 
回傳該RDD的編號(ID)。

    使用時機:
	需取得RDD編號時。
    Example:
    ```python
      test1 = sc.parallelize([1, 2, 3, 4])
      test1.id() #0 
      test2 = sc.parallelize([5, 6, 7, 8])
      test2.id() #1
    ```
2. **cache (self)**:
將storage level設定成MEMORY ONLY，也就是讓此data儲存在memory。
    使用時機:
	想增加運算速度時，但要確保memory足夠大，不然可能會因為memory不夠，造成無法運算。

    Example:
    ```python
      test = sc.textFile(“README.md”).cache()	
    ```

3. **persist (self, storageLevel)**:
設定該RDD固定的storage level，沒設定過的RDD能使用，會配合Spark的
StorageLevel使用。StorageLevel細節請閱讀StorageLevel的文件。

    使用時機:
	自行設定RDD的存取方式。

    Example:
    ```python
      from pyspark import StorageLevel sl 
      test = sc.parallelize([1, 2, 3])
      test.persist(sl.MEMORY_ONLY)
    ```
4. **unpersist (self)**:
將RDD標住為未設定，並將memory和disk的資料都清空。

    使用時機:
	若想重新設定storage level，可利用此method，然後就能用persist重新設定storage level。

    Example:
    ```python
      from pyspark import StorageLevel sl 
      test = sc.parallelize([1, 2, 3])
      test.persist(sl.MEMORY_ONLY)
      test.persist(sl.DISK_ONLY)
    (Fail….)	
      test.unpersist()
      test.persist(sl.DISK_ONLY)
    ```

5. **checkpoint (self)**:
將此RDD標記為已checkpoint，checkpoint後，將會在sc.setCheckpointDir設定的目
錄下儲存一個檔案，並且移除掉與所有parents的關係。

    使用時機:
	在job開始執行前，呼叫checkpoint最適當。
    Example:
    ```python
      sc.setCheckpointDir("/usr/spark-1.0.0/checkpopint/")  
    #path設定為使用者要當checkpoint的目錄，以上路徑為示範。
      test = sc.parallelize(range(5))
      test.checkpoint()
    ```

6. **isCheckpointed (self)**: 
確認該RDD是否checkpoint。 

    使用時機:
	欲得知該RDD是否checkpoint。
    Example:
    ```python
      sc.setCheckpointDir("/usr/spark-1.0.0/checkpopint/")  
    #path設定為使用者要當checkpoint的目錄，以上路徑為示範。
      test = sc.parallelize(range(5))
      test.checkpoint()
      test.count() #14/07/22 12:05:17 INFO RDDCheckpointData: Done checkpointing RDD………
    #若有出現以上訊息，就代表成功checkpoint
      test.isCheckpointed() #True
	```

7. **getCheckpointFile (self)**:
取得該RDD checkpoint的檔案名稱。
    使用時機:
	欲取得該RDD checkpoint的檔案名稱。

    Example:
    ```python
      sc.setCheckpointDir("/usr/spark-1.0.0/checkpopint/")  
    #path設定為使用者要當checkpoint的目錄，以上路徑為示範。
      test = sc.parallelize(range(5))
      test.checkpoint()
      test.count()
      test.getCheckpointFile() #u'file:/usr/spark-1.0.0/checkpopint/8dce8e86-bc06-4614-adef-a0ca1177f3f7/rdd-1079'
	```
8. **map (self, func, perservesPartitioning=False)**:
回傳一個經過mapping後新的RDD。

    使用時機:
	需使用RDD內MapReduce一般的map。

    Example:
    ```python
      test = sc.parallelize([‘a’, ‘b’, ‘a’, ‘c’])
      test.map(lambda x: (x,1)).collect() #[('a', 1), ('b', 1), ('a', 1), ('c', 1)]
    ```

9. **flatMap (self, func, perservesPartitioning=False)**:
回傳一個經過mapping後新的RDD，並再將其flat。簡單而言，就是將mapping後的結果串起來。

    使用時機:
	通常是用flatMap來將關鍵字(keyword)串起來，然後再做map，以僅供參考。主要還是依照programmer的idea，來使用其method。

    Example:
    ```python
      test = sc.parallelize([“I love her”, “He love her”])
      test.flatMap(lambda x: x.split(‘ ‘)).collect() #['I', 'love', 'her', 'He', 'love', 'her']

    #Compare to map
      test.map(lambda x: x.split(‘ ‘)).collect() #[['I', 'love', 'her'], ['He', 'love', 'her']]
    ```
10. **mapPartitions (self, func, perservesPartitioning=False)**:
回傳一個RDD，內容為所有partition經過自訂function處理後，因為mapPartition內部處理是以iterator為主，所以function return的格式需為iterator。

    使用時機:
	需將所有partition經過自訂function處理。
    Example:
    ```python
      test = sc.parallelize(range(1,5), 2)
      def func(iter):
    	 	a = []
    		for i in iter:
        			a.append(i)
    		yield a
      test.mapPartition(func).collect()
    [[1, 2], [3, 4]]
    ```

11. **filter (self, func)**:
回傳一個RDD，內容為符合條件的元素。
    
    使用時機:
	需篩選出需要的資料時，可先使用filter將不需要的資料過慮掉。

    Example:
    ```python
      test = sc.parallelize([1, 2, 3, 4, 5, 6])
      test.filter(lambda x: x%3==0).collect() #[3, 6]
    ```

12. **distinct (self)**:
回傳一個RDD，內容為不重複的元素。
    使用時機:
	需過慮掉重複的元素。
    Example:
    ```python
      test = sc.parallelize([1,1,2,3,3])
      test.distinct().collect() #[1, 2, 3]
    ```

13. **sample (self, withReplacement, fraction, seed=None)**:
回傳一個RDD，內容為該RDD的取樣集合，該function並不強制要求一定要有numpy，spark有內建另一種方法是不需要numpy的。Fraction為取樣比例，seed為決定產生樣本的參數，withReplacement為是否重複取樣。

    使用時機:
	欲從該RDD取出樣本做實驗時。

    Example:
    ```python
      test = sc.parallelize(range(20))
      test.sample(False, 0.2, 3).collect() #[6, 8, 10, 16]
      test.sample(False, 0.2, 7).collect() #[0, 7, 13, 19]
      test.sample(True, 0.2, 4).collect() #[0, 1, 4, 9, 9, 14, 15, 16] 
    ```

14. **takeSample (self, withReplacement, num, seed=None)**:
從該RDD取得一個指定大小的sample subset。withReplacement決定是否用隨機數來替換，seed為決定產生樣本的參數。

    使用時機:
	抽取固定數量sample時。

    Example:
    ```python
      test = sc.parallelize(range(1,100))
      test.takeSample(False, 10, 2) #[59, 97, 93, 43, 71, 33, 45, 82, 70, 89]	
      test.takeSample(True, 10, 2) #[31, 10, 54, 23, 14, 50, 23, 10, 51, 23] (可能有重複的情況)
    ```

15. **union (self, other)**:
將該RDD與其他RDD union在一起。

    使用時機:
    需與其他RDD union時。	
    Example:
    ```python
      rdd1 = sc.parallelize([5, 6, 7, 8])	
      rdd2 = sc.parallelize([1, 2, 3, 4])
      rdd1.union(rdd2).collect() #[5, 6, 7, 8, 1, 2, 3, 4]
    ```

16. **intersection (self, other)**:
回傳一個RDD，內容為兩RDD的交集元素。

    使用時機:	
	欲取得兩RDD的交集時。

    Example:
    ```python
      rdd1 = sc.parallelize([1, 6, 2, 8])	
      rdd2 = sc.parallelize([1, 2, 3, 4])
      rdd1.intersection(rdd2).collect() #[1, 2]
    ```

17. **sortByKey(self, ascending=True, numPartition=None, keyfunc=lambda x: x)**:
將該RDD依據key來sort，RDD的內容需為key-value格式。ascending=True，升冪排序，ascending=False，降冪排列。

    使用時機:
	欲sort RDD的內容時，但RDD內容格式要為key-value。

    Example:
    **Ex1**
    ```python   
      test = sc.parallelize([('a', 1), ('c', 3), ('e', 5), ('b', 2), ('d', 4)])
      test.sortByKey(True, 2).collect() #[('a', 1), ('b', 2), ('c', 3), ('d', 4), ('e', 5)] 
    ```
    **Ex2**
    ```python   
      test = sc.parallelize([(1, 'a'), (3, 'b'), (2, 'c'), (4, 'e'), (5, 'd')])
      test.sortByKey(False).collect() #[(5, 'd'), (4, 'e'), (3, 'b'), (2, 'c'), (1, 'a')]
    ```

18. **glom (self)**:
回傳一個RDD，內容為將所有partition都合在一起。

    使用時機:
	需得知資料如何被分配時。

    Example:
    ```python
      test = sc.parallelize(range(1,10), 3)
      test.glom().collect() #[[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    ```

19. **cartesian (self, other)**:
回傳一個RDD，內容為兩RDD經過cartesian product的結果，會產生出兩兩一組的格式。

    使用時機:
	需讓兩筆資料做cartesian product時。

    Example:
    ```python
      rdd1 = sc.parallelize([1, 2, 3])
      rdd2 = sc.parallelize([4, 5, 6])
      rdd1.cartesian(rdd2).collect() #[(1, 4), (1, 5), (1, 6), (2, 4), (2, 5), (2, 6), (3, 4), (3, 5), (3, 6)]
    ```

20. **groupBy (self, f, numPartitions=None)**:
回傳一個RDD，內容為經過programmer設定的條件來分類。

    使用時機:
	欲將資料依據自行定義的條件分類，不過要注意，此method經過collect回傳後，經過function處理的資料會儲存在spark自訂的
    iterable(resultiterable)，若預將資料取出來，可透過(sorted、max、min、list 目前已知)取出值來。

    Example:
    ```python
      test = sc.parallelize([1, 2, 3, 4, 5, 6])
      result = test.groupBy(lambda x: x%2).collect()
      result #[(0, <pyspark.resultiterable.ResultIterable at 0x34ddb50>),(1, <pyspark.resultiterable.ResultIterable at 0x34ddb90>)]
      [(x, sorted(y)) for (x, y) in result ]  
      [(0, [2, 4, 6]), (1, [1, 3, 5])]
    ##OR..
      [(x, max(y)) for (x, y) in result ]  
      [(0, 6), (1, 5)]
    ```

21. **pipe (self, command, env={ })**:
回傳一個RDD，內容為經過shell-command處理。

    使用時機:
	欲將RDD藉由shell-command處理。
    
    Example:
    ```python
      test = sc.parallelize([1, 2, 3, 4, 5, 6], 3)
      test.pipe(“cat”).collect() #['1', '2', '3', '4', '5', '6']
      test.pipe(“head -n 1”).collect() #['1', '3', '5']
      test.pipe(“tail -n 1”).collect() #[‘2’, ‘4’, ‘6’]
      test = sc.parallelize([1, 2, 3, 4, 5, 6], 2)
      test.pipe(“head -n 2”).collect() #['1', '2', '4', '5']
    ```

22. **foreach (self, func)**:
將RDD的所有元素都經過function處理。

    使用時機:
	欲使RDD內的所有元素都套用在同個function，請注意，foreach並不會回傳任何東西，可配合accumulator使用。

    Example:
    ```python
      accum = sc.accumulator(0)
      def func(x)**:
            global accum
            accum += x
      test = sc.parallelize([1, 2, 3, 4])
      test.foreach(func)
      accum.value #10	
    ```

23. **foreachPartition (self, func)**:
依據RDD設定的partition，讓partition內所有的元素都經過function的處理。

    使用時機:
	將資料拆開，分別給function處理。

    Example:
    ```python
      def f(iter)**:
		      a = []
		      for x in iter:
			     a.append(x)
		      print a
		      yield None (請記得回傳一個空的iterator，不然會噴錯)
      rdd = sc.parallelize([1, 2, 3, 4], 2)
      rdd.foreachPartition(f) #執行後，能在Finished task ID訊息後面找到[1, 2]和[3, 4]，順序不一樣，是因為同步處理，所以先執行完function的會先出現。
    ```

24. **collect (self)**:
回傳RDD內所有的元素。

    使用時機:
	欲從RDD中取出所有元素。

    Example:
    ```python
      test = parallelize([‘a’, ‘b’, ‘c’ ])
      test #ParallelCollectionRDD[108] at parallelize at PythonRDD.scala:286 
(資料存RDD內，需透過collect才可獲得)
      test.collect() #[‘a’, ‘b’, ‘c’]
    ```

25. **reduce (self, func)**:
針對RDD內的元素，使用一種特定交替組合的二進位運算，並透過自訂的function運作。下面會有例子，能讓大家更清楚reduce的大概形式。

    使用時機:
	將資料以自訂的方式整合起來。

    Example:
    ```python
    #清楚reduce的大概形式
      test = sc.parallelize(range(10))
      test.reduce(lambda x, y: (x, y))
    ((((1, 0), (3, 2)), (5, 4)), (9, (8, (7, 6))))  
    #從這結果可得知，x和y的分配不是按照順序，所以做add的operator是OK的，若要做sub
    可能就行不通了。

      from operator import add
      test.reduce(add) 	OR        test.reduce(lambda x, y: x + y) #45  
    ```

26. **fold (self, zeroValue, op)**:
把所有的partition都聚集在一起，然後再經過一個自訂的function和一個共用的zeroValue處理。(Example會呈現fold處理的結構)

    使用時機:
	需將所有partition一起使用自訂function處理時。

    Example:
    ```python
      test = sc.parallelize(range(1,6))
      test.fold(0, lambda x, y: (x, y)) #(0, (5, (4, (3, (2, (1, 0)))))), zeroValue會在開始和結束出現
      test.fold(0, lambda x, y: x+y) #15
      test.fold(1, lambda x, y: x+y) #17, 因為zeroValue前後出現的關係，造成最後結果+2
    ```

27. **aggregate (self, zeroValue, seqOp, combOp)**:
把所有的partition都聚集在一起，然後再經過第一個自訂的function(seqOp)和一個共用的zeroValue處理。最後，再經過第二個自訂function(combOp)與zeroValue處理。
(Example會呈現aggregate處理的架構)
    
    使用時機:
	需將所有partition一起使用自訂function處理時，然後再經過一個function merge起來。

    Example:
    ```python
      test = sc.parallelize(range(1, 5))
      combOp = (lambda x, y: (x, y))
      seqOp = (lambda x, y: (x, y))
      test.aggregate((0, 0), seqOp, combOp) #((0, 0), ((((((0, 0), 1), 2), 3), 4), (0, 0)))  
    #最裡面那層為seqOp ((0, 0) 為zeroValue)，處理完後，再經過combOp處理，外層的兩個(0, 0)也為zeroValue。

      test.aggregate((2, 2), seqOp, combOp) #((2, 2), ((((((2, 2), 1), 2), 3), 4), (2, 2)))
      seqOp = (lambda x, y: (x[0] + y, x[1] + 1))  
      test.aggregate((0, 0), seqOp, combOp)	#((0, 0), ((10, 4), (0, 0)))            
    #最外層會被兩個zeroValue包住
      combOp =  (lambda x, y: (x[0] + y[0], x[1] + y[1]))
      test.aggregate((0, 0), seqOp, combOp) #(10, 4) 
    ```

28. **max (self)**:
找出RDD內的最大值。
    
    使用時機:
	欲找出RDD內最大值。

    Example:
    ```python
      test = sc.parallelize([1, 3, 10, 100])
      test.max() #100
    ```

29. **min (self)**:
找出RDD內的最小值。

    使用時機:
	欲找出RDD內最小值。	

    Example:
    ```python
      test = sc.parallelize([1, 3, 10, 100])
      test.min() #1
    ```

30. **sum (self)**:
將RDD的元素全部總和起來。

    使用時機:
	欲取得RDD所有元素的總和。

    Example:
    ```python
      test = sc.parallelize(range(5))
      test.sum() #10
    ```

31. **count (self)**:
計算出RDD內元素的數量。
    
    使用時機:
	欲求得RDD內元素的數量。

    Example:
    ```python
      test = sc.parallelize([1, 3, 10, 100])
      test.count() #4
    ```

32. **stats (self)**:
印出該RDD的所有state(count、mean、stdev、max、min)。

    使用時機:
	可以用來檢查結果，也可透過他取得值(Ex: test.stats().max())。

    Example:
    ```python
      test = sc.parallelize(range(5))
      test.stats() #(count: 5, mean: 2.0, stdev: 1.41421356237, max: 4, min: 0)	
    ```

33. **mean (self)**:
求出RDD內元素的平均值。
    
    使用時機:
	欲算出RDD內元素的平均值。
    
    Example:
    ```python
      test = sc.parallelize(range(5))
      test.mean() #2.0
    ```
34. **variance (self)**:
求出RDD內元素的變異量。

    使用時機:
	欲算出RDD內元素的變異量。

    Example:
    ```python
      test = sc.parallelize(range(5))
      test.variance() #2.0
    ```

35. **stdev (self)**:
求出RDD內元素的標準差。

    使用時機:
	欲算出RDD內元素的標準差。

    Example:
    ```python
      test = sc.parallelize(range(5))
      test.stdev() #1.4142135623730951
    ```

36. **sampleStdev (self)**:
求出RDD內元素的樣本標準差 (sample和一般的差別在於 /N-1 & /N)。

    使用時機:
	欲算出RDD內元素的樣本標準差。

    Example:
    ```python
      test = sc.parallelize(range(5))
      test.sampleStdev() #1.5811388300841898
    ```

37. **sampleVariance (self)**:
求出RDD內元素的樣本變異量 (sample和一般的差別在於 /N-1 & /N )。

    使用時機:
	欲算出RDD內元素的樣本變異量。

    Example:
    ```python
      test = sc.parallelize(range(5))
      test.sampleVariance() #2.5
    ```

38. **countByValue (self)**:
將RDD每種元素做統計，回傳一個(value, count)的格式，有如dictionary的格式。

    使用時機:
	統計RDD內個別元素的總數。

    Example:
    ```python
      test = sc.parallelize([‘a’, ‘b’, ‘a’, ‘a’, ‘c’])
      test.countByValue() #defaultdict(<type 'int'>, {'a': 2, 'c': 1, 'b': 2}) 
(若要單純取出結果，可使用items)
      test.countByValue().items() #[('a', 2), ('c', 1), ('b', 2)]
    ```

39. **top (self, num)**:
RDD經過降冪排列後，取得最上面N個元素。

    使用時機:
	可以用來取得RDD內前幾大的元素，字母的話，則是按照ASCII排列。

    Example:
    ```python
    #字母
      test = sc.parallelize([‘a’, ‘d’, ‘c’, ‘e’, ‘d’])
      test.top(3) #['e', 'd', 'c']
    #數字
      test = sc.parallelize([4, 30, 100, 2])
      test.top(2) #[100, 30]
    ```

40. **takeOrdered (self, num, key=None)**:
RDD經過升冪排列後，取得最上面N個元素，或是，透過key function來決定排列的方式。

    使用時機:
	依自訂的形式排列後，可取得前N項的元素。

    Example:
    ```python
      test = sc.parallelize([1, 5, 32, 45, 3])
      test.takeOrdered(3) #[1, 3, 5]
      test.takeOrdered(3, key=lambda x: -x) #設定為降冪 ,[45, 32, 5]
      test.takeOrdered(3, key=lambda x: x) #設定為升冪, [1, 3, 5]
    ```

41. **take (self, num)**:
從RDD取出N個元素，取的順序依照原始資料，不會經過排列。

    使用時機:
	從RDD的第一個元素開始，取出N個元素。

    Example:
    ```python
      test = sc.parallelize([1, 5, 32, 45, 3])
      test.take(3) #[1, 5, 32]
    ```

42. **first (self)**:
取出RDD的第一個元素。

    使用時機:
	欲取出RDD內第一個元素時，通常也會用來測試，是否有讀入檔案。

    ```python
    Example:
      test = sc.parallelize(["How", "are", "you"])
      test.first() #‘How’
    ```

43. **saveAsTextFile (self, path)**:
將該RDD以text file的格式儲存在指定的路徑裡，每份資料會獨立儲存在一個text file內。通同會儲存在hdfs內(hdfs://host:port/)，不過也可以儲存在local，若要儲存在local僅需給予一個檔案名稱即可，spark會自動將檔案儲存在你當下執行程式的位置。

    使用時機:
	將結果以text file的格式做儲存。

    Example:
    ```python
      test = sc.parallelize(["How", "are", "you"])
      test.saveAsTextFile("save_test”)		    
    ```

44. **collectAsMap (self)**:
將key-value格式的RDD，並轉換成dictionary的格式。

    使用時機:
    若RDD內為key-value的格式，可透過collectAsMap將其轉換成dictionary，讓操作更方便。

    Example:
    ```python
      test = sc.parallelize([("Name","Terry"), ("Age", 20)])
      result = test.collectAsMap()
      result #{'Age': 20, 'Name': 'Terry'}
      result['Name'] #‘Terry’
    ```

45. **keys (self)**:
key-value格式的RDD，取出key的部份。 

    使用時機:
	只想取得key的部份時。可用來查表用。

    Example:
    ```python
      test = sc.parallelize([("Name","Terry"), ("Age", 20)])
      key = test.keys()
      key #['Age', 'Name']
    ```

46. **values (self)**:
key-value格式的RDD，取出value的部份。

    使用時機:
	只想取得value的部份時。可用來蒐集資料。

    Example:
    ```python
      test = sc.parallelize([("Name","Terry"), ("Age", 20)])
      value = test.values()
      value #[20, 'Terry']	
    ```

47. **reduceByKey (self, func, numPartitions=None)**:
將相同的key，以自訂的function處理後，並merge起來。
Output的部份，若有設定numPartitions，處理的時候將會用hash的方式來分包partition，若沒有設定，則會用原始設定的parallelism level來處理。

    使用時機:
	資料已經map成key-value的格式，並且要將資料merge起來時，通常都會搭配add來做統計的動作。

    Example:
    ```python
      from operator import add
      test = sc.parallelize([("a", 1), ("a", 1), ("b", 1), ("b", 1), ("b", 1)])
      test.reduceByKey(add).collect()  #[('a', 2), ('b', 3)]
    #OR...
      test.reduceByKey(lambda x, y: x + y).collect()  #[('a', 2), ('b', 3)]
    ```

48. **reduceByKeyLocally (self, func)**:
將相同的key，以自訂的function處理後，並merge起來。
與reduceByKey不同的地方，就是回傳的格式是dictionary，並非RDD。

    使用時機:
	欲將資料merge成dictionary的格式時。

    Example:
    ```python
      from operator import add
      test = sc.parallelize([("a", 1), ("a", 1), ("b", 1), ("b", 1), ("b", 1)])
      result = test.reduceByKey(add).items() 
      result #{'a': 2, 'b': 3}
    ```

49. **countByKey (self)**:
計算Key出現的數量，value的值不影響結果。最後以dictionary的格式回傳。

    使用時機:
	欲取得key出現的數量。

    Example:
    ```python
      test = sc.parallelize([("a", 10), ("a", 15), ("b", 12), ("b", 31), ("b", 1)])	
      result = test.countByKey()
      result.items() #[('a', 2), ('b', 3)]
      result[‘a’] #2
    ```

50. **join (self, other, numPartitions=None)**:
回傳一個RDD，內容是依據key然後將兩RDD join在一起。	

    使用時機:
	欲將兩個RDD依據key做join的動作。

    Example:
    ```python
      x = sc.parallelize([("a", 1), ("b", 2), ("d", 5)])
      y = sc.parallelize([("a", 3), ("b", 4), ("c", 2)])
      x.join(y).collect()
      [('a', (1, 3)), ('b', (2, 4))]
    ```

51. **leftOuterJoin (self, other, numPartitions=None)**:
回傳一個RDD，內容是依據key然後將兩RDD left outer join在一起。
(Ex: x.leftOuterJoin(y)) x為left方。

    使用時機:
	欲將兩個RDD依據key做left outer join的動作。請注意，呼叫function的RDD為left。

    Example:
    ```python
      x = sc.parallelize([("a", 1), ("b", 2), ("d", 5)])
      y = sc.parallelize([("a", 3), ("b", 4), ("c", 2)])
      x.leftOuterJoin(y).collect() #[('a', (1, 3)), ('b', (2, 4)), ('d', (5, None))]
    ```

52. **rightOuterJoin (self, other, numPartitions=None)**:
回傳一個RDD，內容是依據key然後將兩RDD right outer join在一起。
(Ex: x.rightOuterJoin(y)) y為right方。

    使用時機:
	欲將兩個RDD依據key做right outer join的動作。請注意，呼叫function的RDD為left。

    Example:
    ```python
      x = sc.parallelize([("a", 1), ("b", 2), ("d", 5)])
      y = sc.parallelize([("a", 3), ("b", 4), ("c", 2)])
      x.leftOuterJoin(y).collect() #[('a', (1, 3)), ('c', (None, 2)), ('b', (2, 4))]
    ```

53. **partitionBy (self, numPartitions, partitionFunc=hash)**:
將資料依據指定的partitioner或原始設定的hash做分割。

    使用時機:
	欲將資料以自訂的方式或使用內建的hash將資料分割成需要的格式。

    Example:
    ```python
      test = sc.parallelize(["a","b","c","d","a","b"]).map(lambda x: (x,1))
      test.partitionBy(3).glom().collect() #[[], [('b', 1), ('d', 1), ('b', 1)], [('a', 1), ('c', 1), ('a', 1)]]
    ```

54. **combineByKey (self, createCombiner, mergeValue, mergeCombiners, numPartitions=None)**:
先將Value經過createCombiner轉換為新的格式((K, V) -> (K, C))，再針對各partition做個別的mergeValue(也就是用來處理各partition的function)，再經過mergeCombines這個自訂的function將各partition merge在一起。

    使用時機:
	需將個partition經過自訂function merge在一起時。

    Example:
    ```python
      x = sc.parallelize([("a", 1), ("a", 1), ("c", 1), ("b", 1), ("b", 1), ("a", 1)], 3)
      def foo1(x, y)**: return (x, y)
      def foo2(x, y)**: return ("test", x, y)
      x.combineByKey(int, foo1, foo1).collect() #[('a', ((1, 1), 1)), ('c', 1), ('b', (1, 1))]
    #看a部份，(1, 1) 是經過mergeValue處理後的結果，((1, 1), 1) 是經過mergeCombiners處理後的結果，(1, 1) = x ，1 = y 。

       x.combineByKey(int, foo1, foo2).collect() #[('a', ('test', (1, 1), 1)), ('c', 1), ('b', ('test', 1, 1))]
    #再次驗證上面例子所證明結構是正確的。
    #c為何沒有’test’ ?  因為c沒有其他的partition，所以不會經過mergeCombiners處理。
      def foo1(x, y)**: return x+y
      x.combineByKey(int, foo1, foo1).collect() #[('a', 3), ('c', 1), ('b', 2)]
    ```

55. **foldByKey (self, zeroValue, func, numPartitions=None)**:
回傳一個RDD，依據key將value merge在一起，然後再將value經過自訂function處理。

    使用時機:
	需將key的value一起透過function處理時，zeroValue是所有value可使用的共用值，可自行設定。

    Example:
    ```python
      from operator import add
      test = sc.parallelize([(“a”, 1), (“b”, 1), (“a”, 1)])
      test.foldByKey(0, add).collect() #[('a', 2), ('b', 1)]
      def f(x,y)**:
    	  	word = ""
    		word = str(x) + str(y)
    	return word
      test.foldByKey(“word: ”, f).collect() #[('a', 'word: 11'), ('b', 'word: 1')]
    ```

56. **groupByKey (self, numPartitions=None)**:
回傳一個RDD，內容為依據key，將value都merge起來。

    使用時機:
    整合key的內容時。value的格式為resultIterable，如同groupBy回傳的格式，可透過(sorted、max、min、list 目前已知)取出值來。

    Example:
    ```python
      test = sc.parallelize([("Name", "Terry"), ("Age", 20), ("Name", "Jhow"), ("Age", 21)]) 	
      result = test.groupByKey.collect()
      result #[('Age', <pyspark.resultiterable.ResultIterable at 0x180cd90>),('Name', <pyspark.resultiterable.ResultIterable at 0x180cfd0>)]
      [(x, list(y)) for (x, y) in result] #[('Age', [20, 21]), ('Name', ['Terry', 'Jhow'])]
    ```

57. **flatMapValues (self, func)**:
將[key1, [value1, value2, value3]]的資料格式拆開為 [(key1, value1), (key1, value2), (key1, value3)]的格式，傳入flatMapValues的values會自動轉為iterator，然後再切割。

    使用時機:
	需將資料切割為個別獨立的(key, value)時。

    Example:
    ```python
      test = sc.parallelize([("a", [1,2,3]), ("b", [4, 5])]
      test.flatMapValues(lambda x: x).collect() # x為iterator的格式, [('a', 1), ('a', 2), ('a', 3), ('b', 4), ('b', 5)]
    ```
58. **mapValues (self, func)**:
將valus透過function處理後，再map為key-value的格式，values為經過fucntion處理過後。
    
    使用時機:
	欲將value重新處理，然後map為新的key-value格式時。

    Example:
    ```python
      test = sc.parallelize([("a", [1,2,3]), ("b", [4, 5])]
      def f(x)**:
	       word = “”
	       for i in x:
		      word += str(i)
	       return word
      test.mapValues(f).collect() #[('a', '123'), ('b', '45')]
    ```

59. **groupWith (self, other)**:
回傳一個RDD，內容為依據key將兩RDD merge為一個新的key-value格式，value的部份為個別的list(格式為resultIterator)。

    使用時機:
	欲將兩RDD依據key，merge成一個新的key-value格式時，但value仍需透過(sorted、max、min、list 目前已知)取出值來。

    Example:
    ```python
      rdd1 = sc.parallelize([("a",1), ("a", 3), ("b",4) ])
      rdd2 = sc.parallelize([("a",2), ("b",5) ])
      result = rdd1.groupWith(rdd2).collect()
      result
    #[('a',(<pyspark.resultiterable.ResultIterable at 0x17ccc90>, <pyspark.resultiterable.ResultIterable at 0x17ccfd0>)), ('b', (<pyspark.resultiterable.ResultIterable at 0x17cca50>, <pyspark.resultiterable.ResultIterable at 0x17ccd50>))]
      [(x, list(y[0]), list(y[1])) for (x, y) in result] #[('a', [1, 3], [2]), ('b', [4], [5])]   
    ```

60. **cogroup (self, other, numPartitions=None)**:
與groupWith完全相同。

    使用時機:
	與groupWith完全相同。

    Example:
    ```python
      rdd1 = sc.parallelize([("a",1), ("a", 3), ("b",4) ])
      rdd2 = sc.parallelize([("a",2), ("b",5) ])
      result = rdd1.cogroup(rdd2).collect()
      result
    #[('a', (<pyspark.resultiterable.ResultIterable at 0x17ccc90>, <pyspark.resultiterable.ResultIterable at 0x17ccfd0>)),('b', (<pyspark.resultiterable.ResultIterable at 0x17cca50>, <pyspark.resultiterable.ResultIterable at 0x17ccd50>))]
      [(x, list(y[0]), list(y[1])) for (x, y) in result] #[('a', [1, 3], [2]), ('b', [4], [5])] 
    ```

61. **subtract (self, other, numPartitions=None)**:
回傳一個RDD，內容為另一個RDD沒有的資料。

    使用時機:
	欲篩選出與另一個RDD不重複的資料。

    Example:
    ```python
      rdd1 = sc.parallelize([1, 2, 3, 4]) 
      rdd2 = sc.parallelize([2, 3])
      rdd1.subtract(rdd2).collect() #[1, 4]
    ```

62. **subtractByKey (self, other, numPartitions=None)**:
回傳一個RDD，內容為依據key，而另一個RDD沒有的資料。
    
    使用時機:
	欲篩選出不重複的key。
    
    Example:
    ```python
      rdd1 = sc.parallelize([("a",2), ("c", 5), ("d", 3)])
      rdd2 = sc.parallelize([("a",5), ("b", 5), ("e", 9)])
      rdd1.subtractByKey(rdd2).collect() #[('c', 5), ('d', 3)]
    ```

63. **keyBy (self, f)**:
創造一個經過自訂function產生的key，然後再map成key-value的格式。
    
    使用時機:
	欲使用自訂的function產生key時。

    Example:
    ```python
      test = sc.parallelize(range(5))
      def f(x)**:                           
   	    	word = ""
    	    word = "data" + str(x)
  		    return word
      test.keyBy(f).collect() #[('data0', 0), ('data1', 1), ('data2', 2), ('data3', 3), ('data4', 4)]
    ```

64. **repartition (self, numPartitions)**:
變更分割(partition)的數量，然後處理資料。
    
    使用時機:
	當partition要設定較原先高時，使用repartition較OK，若要設定成較低，則建議使用coalesce，就能避免多做shuffle，但切割的結果並不是平均切割，而會切割成很多空的。(看Example就可瞭解)

    Example:
    ```python
      test = sc.parallelize(range(10), 2)
      test.glom().collect() #[[0, 1, 2, 3, 4], [5, 6, 7, 8, 9]]
      test.repartition(4).glom.collect() #[[], [], [], [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]]
    ```

65. **coalesce (self, numPartitions, shuffle=False)**:
設定一個新的partition，但需比原本的低，切割效果比repartition好很多。

    使用時機:
	需將partition的數量向下修時，由於效果比repartition好，所以建議初始化的partition能設高一點，之後再用coalesce調整。

    Example:
    ```python
      test = sc.parallelize(range(10), 5) #[[0, 1], [2, 3], [4, 5], [6, 7], [8, 9]]
      test.coalesce(2).glom().collect() #[[0, 1, 2, 3], [4, 5, 6, 7, 8, 9]]	
    ```

66. **zip (self, other)**:
使兩RDD的元素進行配對，配對的數量依最少的RDD為主。

    使用時機:
	進行key-value的配對時。

    Example:
    **Ex1**
    ```python
      rdd1 = sc.parallelize(range(10))
      rdd2 = sc.parallelize(range(5))
      rdd1.zip(rdd2).collect() #[(0, 0), (1, 1), (2, 2), (3, 3), (4, 4)]
    ```

    **Ex2**
    ```python
      x = sc.parallelize(["data1", "data2", "data3"])
      y = sc.parallelize(range(1,4))
      x.zip(y).collect() #[('data1', 1), ('data2', 2), ('data3', 3)]
    ```

67. **name (self)**:
回傳該RDD的Name。 

    使用時機:
	需取得該RDD的Name，可用於辨別RDD，通常配合setName，因為初始化不會有Name。
    
    Example:
    ```python
      test = sc.parallelize(range(5))
      test.name()
      test.setName(“Terry”)
      test.name() #‘Terry ‘
    ```

68. **setName (self, name)**:
設定該RDD的Name。	
    使用時機:
	給予該RDD一個特別的Name，可有辨識功用。
    Example:
    ```python
      test = sc.parallelize(range(5))
      test.name()
      test.setName(“Terry”)
      test.name() #‘Terry ‘	
    ```

69. **toDebugString (self)**:
印出該RDD的資訊。

    使用時機:
	欲得知該RDD的詳細資訊時。

    Example:
    ```python
      test = sc.parallelize(range(5), 5)
      test.setName(“Terry”)	
      test.toDebugString() #'Terry ParallelCollectionRDD[549] at parallelize at PythonRDD.scala:286 (5 partitions)'
    ```

70. **getStorageLevel (self)：
取得該RDD的存取狀態。

    使用時機:
	欲取得該RDD的存取狀態。

    Example:
    ```python
      test = sc.parallelize(range(5), 5)
      test.getStorageLevel() #StorageLevel(False, False, False, False, 1)
      test.cache()	
      test.getStorageLevel() #StorageLevel(False, True, False, True, 1)
    ```
