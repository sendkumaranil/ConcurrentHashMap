# ConcurrentHashMap
How does concurrent hash map works internally?

-----------------------------------------------------------------
<ul><li>java.util.ConcurrentHashMap is implementation of the java.util.Map interface in java.</li>
<li>java.util.ConcurrentHashMap enables us to store data in keyvalue pair form. </li>
<li>Insertion order of keyvalue pairs is not maintained. </li>
<li>ConcurrentHashMap is synchronized in java.</li>


Hierarchy of ConcurrentHashMap in java:<br>
                java.lang.Object <br>
            +java.util.AbstractMap <br>
            +java.util.concurrent.ConcurrentHashMap
<br><br>
Constructs a new ConcurrentHashMap, Its initial capacity is <b>16</b>. And load factor is <b>0.75</b> <br>

        Map<Integer,String> concurrentHashMap=new ConcurrentHashMap<Integer,String>();
Defining ConcurrentHashMap<Integer,String> means key can of Integer type and value can be<br>
String type only, using any other type will cause compilation error.
<br><br>
Concurrency level tells how many threads can access ConcurrentHashMap concurrently, 
<b>default concurrency level of ConcurrentHashMap is 16 .</b><br>
  <b>new ConcurrentHashMap()</b> -> Creates a new ConcurrentHashMap with concurrency level of 16.
<br><br>
ConcurrentHashMap is divided into different <b>segments</b> based on concurrency level. So different
threads can access different <b>segments</b> concurrently in java.
<br><br>
When thread locks one segment for updation it does not block it for retrieval (done by get
method) hence some other thread can read the segment (by get method), but it will be able to read
the data before locking.
<br>
#Segments in ConcurrentHashMap with diagram in java
![alt tag](https://github.com/sendkumaranil/ConcurrentHashMap/blob/master/ConcurrentHashMap.png)
