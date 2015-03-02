#**SparkContext**  
##Concept:  
SparkContext 代表與cluster之間的connection，能使用SparkContext來create RDDs 也可放broadcast variables於cluster上  
於**1.0.0**版後，因為，SparkContext太常使用，且大多數設定SparkContext
(Ex: sc = SparkContext(master)) 為sc，所以spark為了方便，新增了sc這項method，master的部份會設定為local。
不過在寫程式時，為了安全起見，還是將SparkContext import進來(Ex: from pyspark import SparkContext)，然後再來設定SparkContext的參數。


Setting Parameter:

1. master:  設定此工作的master，(Ex: local、spark://host:port、mesos://host:port)，此參數一定要設定，在上述的內容提到的sc，就是將master設定為local。
2. appName:  設定此工作的Name，(Ex: appName = “PythonKmeans”)，此參數的default為none。
3. sparkHome:   設定cluster node的位置
4. environment: 可以給一個儲存環境變數的dictionary，來設定worker nodes。
5. batchSize:      default = 1024, 設定為1，可disable batching，若設定為-1，則batchSize
     為unlimited。
6. conf:	     設定Spark的性質，通常配合SparkConf使用
                          Ex: conf = SparkConf()
		           conf.setMaster(“local”).setAppName(“My App”)
		           sc = SparkContext(conf = conf)    

Class SparkContext: 

1. stop (self): 	  
停止SparkContext。
    使用時機:	  
當需要重新設定SparkContext時，請記得先stop，才能再重新設定SparkContext。

2. parallelize (self, c, numSlices=None): 
分散local的collection來形成RDD, c為使用者輸入的list(ex: sc.parallelize([1,2,3]))。
    使用時機:
	當需要讀入依任意list或資料。
    Example:
    >>>  test = sc.parallize([1, 2, 3])
3. textFile (self, name, minPartitions=None):
從HDFS、本機(local)、Hadoop-supported的URI讀取text file，並回傳RDD格式的
Strings。
    使用時機: 
處理text file相關data
    Example:
    >>>  test = sc.textFile(“README.md”)

4. wholeTextFiles (self, path, minPartitions=None):
	從指定路徑讀入所有text file，路徑可為HDFS、local、Hadoop-supported的URI，任
一個檔案都是單獨紀錄並會回傳一組key-value，key為檔案的path，value則為檔案的
內容。
    使用時機: 
	需要讀入大量text檔時，可利用此method。
    Example:
	假設在wholeTest內有兩個text file，分別為test1.txt和test2.txt，test1.txt的內容
為hello, test2.txt的內容為world
    >>>  dirPath = /usr/spark/wholeTest
    >>>  test = sc.wholeTextFiles(dirPath)

5. union (self, rdds):
	將多個或一個list的RDD，建立起union的關係
    使用時機:
	需要建立RDD間union的關係
    Example: 
    >>>  test1 = sc.parallelize( [“Hey”] )
    >>>  test2 = sc.parallelize( [“Hi”] )
    >>>  sc.union( [ test1, test2 ]).collect()
     [ 'Hey!', 'Hi!' ]

6. broadcast (self, value):
	可提供開發者一個cached在每台機器上read-only的variable。
    使用時機: 
	當處理多個檔案時，可利用broadcast當作識別的工具。
    Example:
    >>>  broad1 = sc.broadcast(“data1”)
    >>>  broad2 = sc.broadcast(“data2”)
    >>>  test1 = sc.parallelize([2, 3, 5, 6, 2]).map(lambda x: (x, broad1.value))
    >>>  test2 = sc.parallelize([6, 1, 7, 2, 4]).map(lambda x: (x, broad2.value))
    >>>  combine = sc.union([test1, test2]).collect()
    >>>  combine
[('data1', 2), ('data1', 3), ('data1', 5), ('data1', 6), ('data1', 2), ('data2', 6), ('data2', 1), ('data2', 7), ('data2', 2), ('data2', 4)]
7. accumulator (self, value, accum_param=None):
accumulator是一種只允許透過關聯操作來進行”add”(add、+)的variable，因此可在
平行時，被有效的support。spark原本就提供int和double的accumulator，而
programmer也可自行添加新的type。accumulator的是不允許讀取的，只有驅動程
式(driver program)可藉由value這個method來讀取值。
    使用時機:
	用來當counter或求和器(sum), accumulator有內建add的method可以使用，相當於
           accum += x。
    Example:
    (Sum)
    >>>  accum = sc.accumulator(0)
    >>>  sc.parallelize([1, 2, 3, 4]).foreach(lambda x: accum.add(x))
    >>>  accum
    Accumulator<id=0, value=10>
    
    (Counter)
    >>>  accum = sc.accumulator(0)
    >>>  sc.parallelize([1, 2, 3, 4]).foreach(lambda x: accum.add(1))
    >>>  accum
    Accumulator<id=0, value=4>
	
8. addFile (self, path):
	在Spark job可隨時增加File到所有node上，路徑可為local、HDFS、HTTP、FTP URL
    使用時機:
	需要增加一個新的file時，可配合SparkFiles使用
    Example:
	假設於/home/usr/file內有1.txt和2.txt，內容分別為Hello和World
    >>>  from pyspark import SparkFiles as sf
    >>>  sc.addFile(“/home/usr/file/1.txt”)
    >>>  sc.addFile(“/home/usr/file/2.txt”)
    >>>  test1 = sc.textFile(sf.get(“1.txt”))
    >>>  test2 = sc.textFile(sf.get(“2.txt”))
    >>>  test1.first()
    u’Hello’
    >>>  test2.first()
    u’World’

9. addPyFiles (self, path):
	可從指定路徑添加.py、.zip、.egg等文件到分佈式系統，路徑可為local、HDFS、
HTTP、FTP URL



10. setCheckpointDir (self, dirName):
	設定指定目錄為checkpoint，若工作執行在cluster上，checkpoint需設定在HDFS上。
    使用時機:
	設定checkpoint為programmer指定的目錄。
    Example:
    >>>  sc.setCheckpointDir(“/home/usr/spark/Checkpoint”)
	
11. setJobGroup (self, groupId, description, interruptOnCancel=False):
	設定一個Job，可給予ID和描述。
    使用時機:
	通常使用在多個工作執行時，可以在每個工作前，使用此method來定義工作ID和描
述。
    Example:
    >>>  import re
    >>>  from operator import add
    >>>  sc.setJobGroup(“wordCount”, “Count the word in article.”)
    >>>  line = sc.textFile(“README.md”)
    >>>  wordCount = line.flatMap(lambda line: re.findall(“[\w’]+”, line)).map(lambda word: (word, 1)).reduceByKey(add).collect()
    (Print the result…….)
  

12. setLocalProperty (self, key, value):
	類似於Hadoop的fair scheduler，可利用此方法來平分工作。
    使用時機:
	平分worker的工作。
    Example:
    >>>  sc.setLocalProperty(“spark.scheduler.pool”, “pool_1”)

13. getLocalProperty (self, key):
	可取得指定的local property。
    使用時機:
	取得指定的local property。
    Example:
    >>>  sc.getLocalProperty(“spark.scheduler.pool”)
    u’pool_1 ’

14. sparkUser (self):
	可取得正在使用SparkContext的user名稱。
    使用時機:
	取得正在使用SparkContext的user名稱。
    Example:
    >>>  sc = SparkContext("local", "terry")
    >>>  sc.sparkUser()
    (get your user name)
15. cancelJobGroup (self, groupId):
	取消指定ID的job。
    使用時機:
	需取消指定ID的job時。
    Example:
    >>>  sc.cancelJobGroup(“wordCount”)
	
16. cancelAllJobs (self):
	取消所有的job。
    使用時機:
	需取消所有的job時。
    Example:
    >>>  sc.cancelAllJobs()

	

