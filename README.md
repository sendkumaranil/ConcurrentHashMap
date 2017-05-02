# ConcurrentHashMap
<b>How does concurrent hash map works internally?</b>

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
<br>
<b>ConcurrentHashMap Question 1 : What will happen map.put(25,12) is called and some other</b><br>
<b>thread concurrently calls map.get(25)?</b><br>
<b>Answer :</b> When <b>map.put(25,12)</b> is called <b>segment 2</b> will be locked,
<b>key=25 also lies in segment 2</b>, When thread locks one segment for updation it does not block it for
retrieval hence some other thread can read the same segment, but it will be able to read the data
before locking (hence <b>map.get(25) will return 121</b>)
<br><br>
<b>ConcurrentHashMap Question 2 : What will happen map.put(25,12) is called and some other</b><br>
<b>thread concurrently calls map.get(33)?</b><br>
<b>Answer :</b> When <b>map.put(25,12)</b> is called <b>segment 2</b> will be locked,
<b>key=33 also lies in segment 2</b>, When thread locks one segment for updation it does not block it for
retrieval hence some other thread can read the same segment, but it will be able to read the data
before locking (hence <b>map.get(33) will return 15</b>)
<br><br>
<b>ConcurrentHashMap Question 3 : What will happen map.put(25,12) is called and some other</b><br>
<b>thread concurrently calls map.put(33,24)?</b><br>
<b>Answer :</b> When <b>map.put(25,12)</b> is called <b>segment 2</b> will be locked,
<b>key=33 also lies in segment 2</b>, When thread locks one segment for updation it does not allow any
other thread to perform updations in same segment until lock is not released on segment.
hence <b>map.put(33,24)</b> will have to wait for <b>map.put(25,12)</b> operation to release lock on segment
<br><br>
<b>ConcurrentHashMap Question 4 : What will happen map.put(25,12) is called and some other</b><br>
<b>thread concurrently calls map.put(30,29)?</b><br>
<b>Answer :</b> When <b>map.put(25,12)</b> is called <b>segment 2</b> will be locked,
but <b>key=30 lies in segment 3.</b>
Both the kays lies in different segments, hence both operations can be performed concurrently.
<br><br>

<b>ConcurrentHashMap Question 5 : What will happen updations (put/remove) are in process in</b><br>
<b>certain segments and new keypair have to be put/remove in same segment ?</b><br>
<b>Answer :</b> When updations are in process thread locks the segment and it does not allow any other
thread to perform updations (put/remove) in same segment until lock is not released on segment.

<br><br>
<b>ConcurrentHashMap putIfAbsent method in java:</b><br>
Definition of putIfAbsent method in java <br>
    <b>public V putIfAbsent(K key, V value)</b>
<br>
What do putIfAbsent method do:<br>
If map does not contain specified key, put specified key‐value pair in map and return null in java.<br>
If map already contains specified key, return value corresponding to specified key.<br>

    putIfAbsent method is equivalent to writing following code in java :
    synchronized (map){
    if (!map.containsKey(key))
    return map.put(key, value);
    else
    return map.get(key);
    }


<h4>Why concurrentHashMap does not allowed null as value or null as key?<h4>

<b>Why ConcurrentHashMap does not allow null value?</b>

Reasoning :

		The method get in Segment implementation of a ConcurrentHashMap takes a lock on the segment itself, if the value of a HashEntry turns out to be null. 
		This is illustrated in get method:

		1.  V get(Object key, int hash) {
		2.    if (count != 0) { // read-volatile
		3.      HashEntry e = getFirst(hash);
		4.      while (e != null) {
		5.        if (e.hash == hash && key.equals(e.key)) {
		6.          V v = e.value;
		7.            if (v != null)
		8.              return v;
		9.          return readValueUnderLock(e); // recheck //takes lock if the value is null
		10.       }
		11.       e = e.next;
		12.     }
		13.   }
		14.   return null;
		15. }

		Now if ConcurrentHashMap does not allows null values then how can the HashEntry have a null value 
		(value is marked volatile in HashEntry).Consider a scenario while a thread may be trying to put a new value on the HashEntry 
		(line 21 in put method) in the put method of the ConcurrentHashMap.The HashEntry object is created but not yet initialized, 
		so that value attribute in HashEntry does not reflects its actual value, but instead reflects null. 
		At this point a reader gets the HashEntry and reads a null for attribute value in HashEntry, 
		thus having a need to recheck with a lock (line 9. get method.).

Put method :

		1.  V put(K key, int hash, V value, boolean onlyIfAbsent) {
		2.    lock();
		3.    try {
		4.      int c = count;
		5.      if (c++ > threshold) // ensure capacity
		6.        rehash();
		7.      HashEntry[] tab = table;
		8.      int index = hash & (tab.length - 1);
		9.      HashEntry first = tab[index];
		10.     HashEntry e = first;
		11.     while (e != null && (e.hash != hash || !key.equals(e.key)))
		12.       e = e.next;
		13.     V oldValue;
		14.     if (e != null) {
		15.       oldValue = e.value;
		16.       if (!onlyIfAbsent)
		17.         e.value = value;
		18.       } else {
		19.         oldValue = null;
		20.         ++modCount;
		21.         tab[index] = new HashEntry(key, hash, first, value); // New value for HashEntry
		22.         count = c; // write-volatile
		23.       }
		24.     return oldValue;
		25.   } finally {
		26.   unlock();
		27. }
		28.  }

This extra check in Line 9 of get method is very costly (as we already see it in the test results above) and is avoided if a not null value of HashEntry is encountered. 
In case the null values are allowed this would require to have this lock acquired each time a null value is accessed, hence making the ConcurrentHashMap slower.

<b>Not allowing null key:</b>

The reason behind this implementation is due to ambiguity nature of the ConcurrentMap (ConcurrentHashMap) 
when it comes to threading environment. 
The main reason is that if map.get(key) returns null, we can't detect whether the key explicitly maps to null vs the key isn't mapped. 
In ConcurrentHashMap, It might be possible that key 'K' might be deleted in between the get(k) and containsKey(k) calls.

For example in below code of ConcurrentHashMap

		Line 1: if (concurrentHashMap.containsKey(k)) {     
		Line 2:      return concurrentHashMap.get(k);  
		Line 3: } else {
		Line 4:      throw new KeyNotPresentException();
		Line 5: }

The above example i am  explaining with two thread(1 & 2). 
In Line 2, after checking thread 1 with the key "K" in concurrentHashMap(Line 1 - contains some value for the key "K"),  
it will returns null (if thread 2 removed the key 'K' from concurrentHashMap). 
Instead of throwing KeyNotPresentException , it actually returns null.

The above scenario will not happen in case of non-concurrent map (HashMap), 
because there is no Concurrent access. 
So null key are allowed in HashMap by eliminating the ambiguity.
