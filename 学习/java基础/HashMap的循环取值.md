# HashMap的循环
#### forEach()

```

    HashMap<K,V> map = new HashMap<>();
    map.put(K,V);
    ...
    
    map.forEach((K,V)->{
        System.out.println("键->" + K);
        System.out.println("值->" + V);
    });
    
```
#### entrySet()
```
    1)
    for(Map.Entry<K,V> entry:map.entrySet<K,V>){
        System.out.println("键->" + entry.getKey());
        System.out.println("值->" + entry.getValue());
    }
    
    2)
    Iterator iter = map.entrySet<K,V>.iterator();
    while(iter.hasNext()){
        Entry<K,V> entry = iter.next();
        System.out.println("键->" + entry.getKey());
        System.out.println("值->" + entry.getValue());
    }
```
    
#### keySet()
 ``` 
    Iterator iter = map.keySet().iterator();
    while(iter.hasNext()){
        Object key = iter.next();
        Object value = map.get(key);
        System.out.println("键->" + key);
        System.out.println("值->" + value);
    }
```