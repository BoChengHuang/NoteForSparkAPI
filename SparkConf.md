#SparkConf
SparkConf也就是指**Spark Configuration**，主要是使用來設定Spark重要的參數和性質。  
值得一提的是，SparkConf可支援chaining 
(Ex: conf.setMaster(“local”).setAppName(“Test”).setSparkHome(“UserName”))。
還有一點要注意，當SparkConf設定好並傳到Spark後，將會被clone住且不能再被user調整。
而SparkConf都是與SparkContext配合使用 
Example:
```python 
    conf = SparkConf()
    conf.setMaster(“local”).setAppName(“Test”)
    sc = SparkContext(conf = conf)
```


##Setting Parameter:


1. **loadDefaults**: 
設定是否從Java系統的properties讀取value(Default = True)

###Class SparkConf:


1. **set (self, key, value)**:
設定configuration的properties。set可以設定絕大多數的value，就連底下會介紹到的
executorEnv也可透過set設定，set讀進key時，會先去搜尋是否有該參數，然後
加以更改，若沒有，會新增該參數。

    使用時機:
	  需設定programmer想要的properties時，可對指定的參數設定或修改value。

    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()
      conf.set(“spark.master”, “local”)
      conf = setExecutorEnv(“master”, “local”) #executorEnv
      conf.getAll() #(u'spark.executorEnv.master', u'local')
      conf.set(“spark.executorEnv.master”, “local[2]”)
      conf.getAll() #(u'spark.executorEnv.master', u'local[2]')
    ```
    
2. **setMaster (self, value)**:
設定master的位置

    使用時機:
	  設定指定的master(Ex: local、yarn-client)

    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()
      conf.setMaster(“local”)
    ```

3. **setAppName (self, value)**:
設定Application的名稱。

    使用時機:
	  需設定自行定義的Application時。

    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()
      conf.setAppName(“Spark Testing”)
    ```

4. **setSparkHome (self, value)**:
設定安裝Spark節點的路徑	

    使用時機:  
	  設定spark home到指定的路徑，spark都會自行設定好，所以通常不設定。

    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()
      conf.setSparkHome(“/home/usr/spark/”)
    ```

5. **setExecutorEnv (self, key=None, value=None, pairs=None)**:
設定傳給excutor的環境變數，使用pairs可一次設定較多個。

    使用時機:
  	可設定或新增環境的key與其相對應的value

    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()
      conf.setExecutorEnv(“master”, “local”)
      conf.setExecutorEnv(pairs = [(“home”, “user”), (“spark”, “name”)])
      conf.get(“spark.executorEnv.master”)
    u’local ‘
      conf.get(“spark.executorEnv.home”)
    u’spark ‘
    ```

6. **setAll (self, pairs)**:
設定多個參數，每組參數都是key-value的格式，設定方法與setExecutorEnv使用pairs設定雷同。

    使用時機:
	  需設定多個Spark的參數時。

    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()	
      conf.setAll(pairs = [(“master”, “appName”), (“local”, “Test”)])
      conf.get(“master”)
    u’local ‘
      conf.get(“appName”)
    u’Test ‘
    ```
7. **get(self, key, defaultValue=None)**:
取得指定參數名稱的value, 若key不存在則回傳None或指定的default值

    使用時機:
	  需取得指定參數名稱的value時。

    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()	
      conf = getAll()
    [(u'spark.submit.pyFiles', u''),
    (u'spark.app.name', u'pyspark-shell'),
    (u'spark.master', u'local[*]')]
      conf.get(“spark.master”)
    u'local[*] '
      conf.get(“spark.home”)
    (None)
      conf.get(“spark.home”, “This parameter does not exist.”)
    u'This parameter does not exists. '
    ```
8. **getAll (self)**:
取得一個內容為所有key與其對應value的list。

    使用時機:
   	需取得conf內所有的key與value時。

    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()	
      conf = getAll()
    [(u'spark.submit.pyFiles', u''),
    (u'spark.app.name', u'pyspark-shell'),
    (u'spark.master', u'local[*]')]	
    ```
9. **contains (self, key)**:
找尋指定的key是否存在於configuration。

    使用時機:
	  驗證一指定key是否存在於configuration。

    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()	
      conf.contains(“spark.master”) #True
      conf.contains(“Test”) #False	
    ```

10. **toDebugString (self)**:
回傳一個printable的值，內容為key與其相對的value。

    使用時機:
	  可以直接將資料印出來，且較清楚。
    
    Example:
    ```python
      from pyspark import SparkConf
      conf = SparkConf()	
      conf = getAll()
    [(u'spark.submit.pyFiles', u''),
    (u'spark.app.name', u'pyspark-shell'),
    (u'spark.master', u'local[*]')]	
      printf conf.toDebugString()
    spark.app.name=pyspark-shell
    spark.master=local[*]
    spark.submit.pyFiles=
    ```
