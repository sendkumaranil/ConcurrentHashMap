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
Constructs a new ConcurrentHashMap, Its initial capacity is 16. And load factor is 0.75 (We’ll
discuss it later in post) <br>

        Map<Integer,String> concurrentHashMap=new ConcurrentHashMap<Integer,String>();
Defining ConcurrentHashMap<Integer,String> means key can of Integer type and value can be<br>
String type only, using any other type will cause compilation error.
<br><br>
Concurrency level tells how many threads can access ConcurrentHashMap concurrently, 
default concurrency level of ConcurrentHashMap is 16.
        new ConcurrentHashMap()
Creates a new ConcurrentHashMap with concurrency level of 16.
