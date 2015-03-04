#SparkFiles
SparkFiles會配合SparkContext的addFile這個method使用，它能決定addFile的路徑，在之後若要再呼叫同一個檔案，只需要配合SparkFiles就能輕易呼叫。
使用時，不需要create多個SparkFiles，SparkFiles會自行根據addFile來讀取路徑。


###Setting Parameter:
(No parameter need to be set)

###Class SparkFiles: 
    
1. **get (cls, filename)**:
可取得透過addFile所加入的File絕對路徑。 
    
    使用時機:
	需呼叫addFile過的File時。

    Example:
    ```python
      from pyspark import SparkFiles as sf
      sc.addFile(“/home/usr/file/1.txt”)
      sf.get("1.txt") #u'/tmp/spark-988df13f-a5d4-4bbe-af95-b8e228ee25fb/1.txt’
      test1 = sc.textFile(sf.get(“1.txt”))
      test1.first() #u’Hello’
    ```
2. **getRootDirectory (cls)**:
可取得addFile暫存檔案的目錄。
    
    使用時機:
	需取得addFile暫存檔案的目錄時。

    Example:
    ```python
      from pyspark import SparkFiles as sf
      sc.addFile(“/home/usr/file/1.txt”)
      sf.getRootDirectory() #u'/tmp/spark-988df13f-a5d4-4bbe-af95-b8e228ee25fb'
	```

