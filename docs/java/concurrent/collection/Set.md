# Set

## 子类

### HashSet

无序不重复集合，实现依赖于 HashMap 或 LinkedHashMap；

当使用构造方法 `HashSet(int initialCapacity, float loadFactor, boolean dummy)` 是会使用 LinkedHashMap 创建有序集合，但是此构造为 protect ，是为 LinkedHashSet 提供的；

### LinkedHashSet

HashSet 的子类，提供有序不重复的集合；初始化时如上所示，会采用LinkedHashMap创建有序集合；

### TreeSet

依赖于 TreeMap

### CopyOnWriteArraySet

依赖于 CopyOnWriteArrayList 

### ConcurrentSkipListSet

依赖于 ConcurrentSkipListMap