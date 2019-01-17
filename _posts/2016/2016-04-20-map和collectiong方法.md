# map集合

## 1.双列集合map

​	双列集合是每个元素都有键与值两部分组成的集合，记录的是键值对对应关系。即通过键可以找值。 

常用子类:

​	HashMap:键唯一且无序，值可以重复 

​	用法:

​		put(key, value);放入对应的键和值

​		get(key);得到键所对应的值



## 2.map的使用

A：Map(HashMap)的使用：创建对象时加入两个泛型。

​	Map<k,v>

​	key - 此映射所维护的键的类型

​	value - 映射值的类型

B：常用方法：

​	public V put(K key,V value)          //加入元素，则新值会覆盖掉旧值

​	public V get(Object key)                  //根据键找值

​	****public Set<K> keySet()                    //返回所有键的集合****

​	****public Collection<V> values()       //返回所有值的集合****



## 3.map的遍历

第1种遍历方式是，使用Map集合的**keySet()方法**

A:思路：

   通过keySet()方法获取所有键的集合

   遍历键的集合，获取到每一个键

   根据键找值

​	 Map没有迭代器方法,最常用的遍历方法:先获取所有键的集合,迭代该集合,依次获取每一个键.通过键找值. 



第2种遍历方式是，使用Map集合的**entrySet()方法**

A: 思路：

   * 通过entrySet()方法获取所有键值对对象的集合

   * 遍历键值对对象的集合，获取到每一个键值对对象

   * 根据键值对对象找键和值

B: entrySet()方法解释

   Set<Map.Entry<K,V>> entrySet() 方法用于返回某个集合所有的键值对对象。

   Map.Entry说明Entry是Map的内部接口,将键和值封装成了Entry对象,并存储在Set集合中。可以从一个Entry对象中获取一个键值对的键与值。

C: Entry中的方法如下：

​      K getKey()      获取键

​      V getValue()    获取值



## 4.LinkedHashMap

A:LinkedHashMap:

​       * Linked链表结构,保证元素有顺序

​       * Hash结构保证元素唯一

​       * 以上约束对键起作用

B:LinkedHashMap的特点

       * 底层是链表实现的可以保证怎么存就怎么取



# Collections类方法

## 1. addall()

Collections中有一个方法可以一次加入多个元素

​       public static <T> boolean addAll(Collection<? super T> c,T... elements)

​       该方法使用到了可变参数，即定义时并不知道要传入多少个实际参数。此时定义成...的方式，此时可以在调用该方法时，一次传入多个参数。传入的多个数将被自动组织成数组，我们只要操作生成的数组即可。

​       注：可变参数只能放在最后定义。可变参数方法的参数本质是数组，所以不可以与数组类型参数重载。

## 2.shuffle

shuffle方法的作用：

​       打乱集合中元素顺序

public static void shuffle(List<?> list)            

## 3.sort

A：sort方法的作用：

​	对集合中元素排序

B：sort方法签名

 public static <T> void sort(List<T> list)

​	有顺序(有序)：第一个元素是多少,第二个元素是多少,第几个元素对应的是第几,顺序不变.

排序：不管是第几个放的,只要到集合中(以Integer集合为例),就按照一定的顺序重新排列了. 

## 4.bianrySearch

public static <T> int binarySearch(List<?>list,T key)   //查找元素索引

二分法查找:在一个集合当中,查找一个指定元素的索引是多少,如果不存在该元素,就返回负数索引 

二分法查询必须要求集合中的元素排好顺序 

## 5.asList(数组方法)

数组转集合使用的是，Arrays类中的asList方法

方法签名如下：

​       public static <T> List<T> asList(T... a)

解释：

​       静态方法，直接类名.方式调用

​       方法的形参是可变参数类型，可变参数本质也是数组，传入实际数组，将该数组转成集合返回

注意：

​       数组转成集合之后，该集合不支持添加或者删除操作，否则会抛出UnsupportedOperationException异常

## 6.toArray

集合ArrayList转数组使用的是，ArrayList中的toArray()方法。

该方法是重载的方法：

​       public Object[] toArray()

​       public <T> T[] toArray(T[] a)