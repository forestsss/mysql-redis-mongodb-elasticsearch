# mysql-redis-mongodb-elasticsearch

## Mysql数据库的个人理解和总结

### 常用的数据类型
整数数据类型、浮点数数据类型、精确小数类型、二进制数据类型、日期/时间数据类型、字符串数据类型。

### MySQL中的存储引擎myisam和innodb
（1）InnoDB存储引擎支持事务，而MyISAM不支持事务；  
（2）InnoDB支持行级锁，而MyISAM只支持表级锁；  
（3）InnoDB支持外键，而MyISAM不支持外键；  
（4）InnoDB不保存数据库表中表的具体行数，而MyISAM会保存；  

### MySQL中的事务，以及事务的特征
事务是一种操作序列，一系列的操作要么都执行，要么都不执行，它们是一个整体。每个事务结束时，都保持着数据一致性。事务有四大特征：原子性、一致性、隔离性、持久性。  

### 数据库中的乐观锁和悲观锁
乐观锁：假设不会发生并发冲突，只在提交的时候检查是否发生并发冲突。可以使用版本号机制和CAS算法实现。

版本号机制：一般在数据表中加一个数据版本号version字段，表示数据被修改的次数，当数据被修改时version值加一。  
当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若当前读取到的version值与第一次读取到的数据库version值相等时才更新，否则重试更新操作，直到更新成功。  

### CAS机制：即compare and swap（比较与交换），无锁编程，在不使用锁的情况下实现多线程之间的变量同步，也就是在没有线程被阻塞的情况下实现变量的同步，因此也叫非阻塞同步。

悲观锁：假定会发生并发冲突，在读取的时候就对数据进行加锁， 在该用户读取数据的期间，其他任何用户都不能来修改该数据，但是其他用户是可以读取该数据的， 只有当自己读取完毕才释放锁。

　在数据库中可以使用Repeatable Read的隔离级别（可重复读）来实现悲观锁，它完全满足悲观锁的要求（加锁）。Java中synchronized和ReentrantLock等独占锁就是悲观锁思想的实现。

两种锁的使用场景：
乐观锁性能好，但是无法解决脏读问题
悲观锁能解决脏读问题，开销较大，而且加锁时间较长，对于并发的访问性支持不好。
　　如果冲突很少，或者冲突的后果不会很严重，那么通常情况下应该选择乐观锁，因为它能得到更好的并发性；
　　如果冲突太多或者冲突的结果对于用户来说痛苦的，那么就需要使用悲观策略，它能避免冲突的发生。

一般乐观锁适用于写比较少的情况下（多读场景），即冲突真的很少发生的时候；悲观锁适用于多写的情况，多写的情况一般会经常产生冲突。

### 共享锁与排它锁
共享锁和排它锁是具体的锁，是数据库机制上的锁。

共享锁（读锁）： 在同一个时间段内，多个用户可以读取同一个资源，读取的过程中数据不会发生任何变化。读锁之间相互不阻塞， 多个用户可以同时读，但是不能允许有人修改。 
排它锁（写锁）： 在任何时候只能有一个用户写入资源，当进行写锁时会阻塞其他的读锁或者写锁操作，只能由这一个用户来写，其他用户既不能读也不能写。
加锁会有粒度问题，从粒度上从大到小可以划分为 ：

表锁：开销较小，一旦有用户访问这个表就会加锁，其他用户就不能对这个表操作了，应用程序的访问请求遇到锁等待的可能性比较高。
页锁：是MySQL中比较独特的一种锁定级别，锁定颗粒度介于行级锁定与表级锁之间，所以获取锁定所需要的资源开销，以及所能提供的并发处理能力也同样是介于上面二者之间。  
另外，页级锁定和行级锁定一样，会发生死锁。
行锁：开销较大，能具体的锁定到表中的某一行数据，但是能更好的支持并发处理， 会发生死锁。

### MySQL数据库的基本的索引类型
索引是对数据库表中一或多个列的值进行排序的结构，利用索引可快速访问数据库表的特定信息。

普通索引、唯一索引、主键索引、联合索引、全文索引。

唯一索引：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。
主键索引：是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值。 为表定义主键将自动创建主键索引。（数据库表某列或列组合，其值唯一标识表中的每一行。该列称为表的主键。）  
联合索引：指对表上的多个列做索引。只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循最左前缀原则。
全文索引：主要用来查找文本中的关键字，而不是直接与索引中的值相比较。目前只有char、varchar，text 列上可以创建全文索引。

### 数据表建立索引的原则：
在最频繁使用的、用以缩小查询范围的字段上建立索引。
在频繁使用的、需要排序的字段上建立索引。

### B+树索引与哈希索引
Hash索引和B+树索引的特点：

Hash索引结构的特殊性，其检索效率非常高，索引的检索可以一次定位;

B+树索引需要从根节点到枝节点，最后才能访问到页节点这样多次的IO访问。

### Hash索引与B+树索引区别？

如果是等值查询，那么哈希索引明显有绝对优势，因为只需要经过一次算法即可找到相应的键值；当然了，这个前提是，键值都是唯一的。如果键值不是唯一的，就需要先找到该键所在位置，然后再根据链表往后扫描，直到找到相应的数据；
从示意图中也能看到，如果是范围查询检索，这时候哈希索引就毫无用武之地了，因为原先是有序的键值，经过哈希算法后，有可能变成不连续的了，就没办法再利用索引完成范围查询检索；
同理，哈希索引也没办法利用索引完成排序，以及like ‘xxx%’ 这样的部分模糊查询（这种部分模糊查询，其实本质上也是范围查询）；
哈希索引也不支持多列联合索引的最左匹配规则；
B+树索引的关键字检索效率比较平均，不像B树那样波动幅度大，在有大量重复键值情况下，哈希索引的效率也是极低的，因为存在所谓的哈希碰撞问题。

### B树和B+树
B+树是一种平衡查找树。在B+树中，所有记录节点都是按键值的大小顺序存放在同一层的叶节点中，各叶结点指针进行连接。

（平衡二叉树AVL：首先符合二叉查找树的定义（最结点的值比根节点小，右结点的值比根结点大），其次必须满足任何节点的左右两个子树的高度最大差为1。）

B树  ：每个节点都存储key和data，所有节点组成这棵树，并且叶子节点指针为nul，叶子结点不包含任何关键字信息。
B+树：所有的叶子结点中包含了全部关键字的信息，及指向含有这些关键字记录的指针，且叶子结点本身依关键字的大小自小而大的顺序链接，所有的非终端结点可以看成是索引部分。

B+比B树更适合实际应用中操作系统的文件索引和数据库索引
（1）B+的磁盘读写代价更低

　　B+的内部结点并没有指向关键字具体信息的指针。因此其内部结点相对B树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说IO读写次数也就降低了。

（2）B+tree的查询效率更加稳定
　由于非终结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。

### 聚集索引和非聚集索引

数据库中的B+索引可以分为聚集索引和辅助聚集索引。不管是聚集索引还是非聚集的索引，其内部都是B+树的，即高度平衡的，叶节点存放着所有的数据，聚集索引与非聚集索引不同的是，叶节点存放的是否是一整行的信息。

聚集索引(clustered index)：
　　聚集索引就是按照每张表的主键构造一颗B+树，并且叶节点中存放着整张表的行记录数据，因此也让聚集索引的叶节点成为数据页。聚集索引的这个特性决定了索引组织表中数据也是索引的一部分。由于实际的数据页只能按照一颗B+树进行排序，因此每张表只能拥有一个聚集索引。

　　聚集索引表记录的排列顺序和索引的排列顺序一致，所以查询效率快，只要找到第一个索引值记录，其余就连续性的记录在物理也一样连续存放。聚集索引对应的缺点就是修改慢，因为为了保证表中记录的物理和索引顺序一致，在记录插入的时候，会对数据页重新排序。

非聚集索引(nonclustered index)（也叫辅助索引）：
　　对于辅助索引(非聚集索引)，叶级别不包含行的全部数据。聚集索引键来告诉InnoDB存储引擎，哪里可以找到与索引相对应的行数据。辅助索引的存在并不影响数据在聚集索引中的组织，因此每张表上可以有多个辅助索引。通过辅助索引来寻找数据时，InnoDB存储引擎会遍历辅助索引并通过叶级别的指针获得指向主键索引的主键，然后再通过主键索引来找到一个完整的行记录。

　　非聚集索引指定了表中记录的逻辑顺序，但是记录的物理和索引不一定一致，两种索引都采用B+树结构，非聚集索引的叶子层并不和实际数据页相重叠，而采用叶子层包含一个指向表中的记录在数据页中的指针方式。非聚集索引层次多，不会造成数据重排。

根本区别：

聚集索引和非聚集索引的根本区别是表记录的排列顺序和与索引的排列顺序是否一致。


## Redis个人理解和总结


### redis 是Nosql数据库， key-value存储系统。 基于内存运行，性能高效。
### 低延迟的读写速度。应用广泛：可以做缓存系统（缓存热点数据），计数器，排行榜等。 

### 有五种自有数据类型：string,hash,list,set,zset.
string 类型 二进制安全的字符串，可以存储字符串，图片，视频等，最大长度512M，做一些复杂的计数功能的缓存。
hash类型，存放的是结构化对象，方便于操作其中的字段，可以用在单点登录上，将cookieId作为key,用户信息存放在value中。
list：插入顺序排序的字符串元素集合，基于双链表。
set:无顺序集合，元素唯一,底层hash表，使用交并差集，可以做共同爱好，个性化喜好等功能。
zset带权重参数的集合 底层hash表，可以做排行榜等功能。

### redis的持久化机制：
持久化机制有两种，1.RDB(redis默认使用)，通过时间周期（通过配置文件中的save参数定义快照周期）把内存中的数据以快照的形式保存到磁盘中，产生dump.rdb的数据文件。
2.AOF,redis会将每个收到的写命令都通过write函数追加到文件最后。同时开启两种机制，数据恢复redis会优先选择AOF恢复。

### Redis的缓存击穿，缓存雪崩，缓存预热：

缓存击穿：是指用户查询的数据在数据库中没有，并且大量查询该数据，导致缓存中不存在该数据，数据库中也没有，增加了很多无效的查询，增大了数据库的访问压力，特别是一些恶意攻击。
解决该问题的方法是采用布隆过滤器，它的核心思想是利用多个hash函数完成对数据的判重，当数据通过所有hash函数判断数据存在于集合，则存在，没有则直接返回空数据。
缓存雪崩：缓存雪崩是指某个时间，大量缓存失效了，大量的访问直接查询数据库，导致数据库压力很大，甚至超出数据库的承载压力。应对方案可以是：采用多redis的方式，多redis之间设置有间隔的数据过期时间。
缓存预热:对于一部分热点数据，在服务开启时将热点数据加载进缓存中，例如秒杀活动，可以在活动开始前五分钟，将活动信息都加载进缓存中，可以是定时刷新缓存。

### Redis的过期策略以及内存淘汰机制：

Redis采用定期删除+惰性删除策略，定期删除是指，默认在每100ms随机抽查是否有过期的key,如果有则删除。惰性删除是指，当用户获取某个key时，redis会检查是否过期，过期了则删除。

.内存淘汰策略有：（在redis.conf中有一行配置 maxmemory-policy）
volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰  
volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰  
volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰  
allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰  
allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰。  

redis实现分布式锁
    Setnx lock-key value1  
    Setnx lock-key value2  
    Get lock-key  
 
### redis在springboot中的使用：

导入依赖

<!--redis-->  
       <dependency>  
           <groupId>org.springframework.boot</groupId>  
           <artifactId>spring-boot-starter-data-redis</artifactId>  
       </dependency>

配置连接信息

#### Redis数据库索引（默认为0）  
spring.redis.database=0  
#### Redis服务器地址  
spring.redis.host=192.168.0.24  
#### Redis服务器连接端口  
spring.redis.port=6379  
#### Redis服务器连接密码（默认为空）  
spring.redis.password=  
#### 连接池最大连接数（使用负值表示没有限制）  
spring.redis.pool.max-active=200  
#### 连接池最大阻塞等待时间（使用负值表示没有限制）  
spring.redis.pool.max-wait=-1  
#### 连接池中的最大空闲连接  
spring.redis.pool.max-idle=10 
#### 连接池中的最小空闲连接  
spring.redis.pool.min-idle=0  
#### 连接超时时间（毫秒）  
spring.redis.timeout=1000 

### 使用redisTempate

public class Test_1{
    @Autowired
    private RedisTemplate<String,String>redisTemplate;

    @Test
    public void set(){
        redisTemplate.opsForValue().set("myKey","myValue");
        System.out.println(redisTemplate.opsForValue().get("myKey"));
    }
}

### 创建一个 RedisTemplate<String,Object>的类

@Configuration
public class RedisConfig {
   
    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {  
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();  
        template.setConnectionFactory(factory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);  
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}

### 操作redis的五种数据类型：
opsForValue()：操作字符串。
opsForList()：操作列表。
opsForHash()：操作哈希。
opsForSet()：操作集合。
opsForZSet()：操作有序集合。

### string:
redisTemplate.opsForValue().set("name","linqz",3, TimeUnit.SECONDS);  
获取旧值：getAngSet  追加字符串：.append。

### list: 双向列表：leftPushAll 左侧插入，rightPushAll右侧插入  
redisTemplate.opsForList().rightPush("userInfo",1);  
redisTemplate.opsForList().index("userInfo",0)；  

redisTemplate.opsForList().rightPushAll("user1",user1);  
redisTemplate.opsForList().range("user1",0,-1)；  
查询列表长度  
redisTemplate.opsForList().size("user1")；  
删除列表中的元素
    redisTemplate.opsForList().remove("user1",1,"linqz");  
弹出元素
redisTemplate.opsForList().rightPop("user1")；  


### hash
在单点登录的时候存储用户信息，以cookie作为key,设置缓存时间30min.  
redisTemplate.opsForHash().putAll("userHash",userMap);  


redisTemplate.opsForHash().put("userHash","userName","linqz");  
获取特定key的value。
redisTemplate.opsForHash().get("userHash","userName");  
获取特定hash的所有value值
redisTemplate.opsForHash().values("userHash");  
获取特定hash的所有key
redisTemplate.opsForHash().keys("userHash");  
删除特定hash下的特定key
redisTemplate.opsForHash().delete("userHash","userName");  


### set集合：不存放重复值，无序。可以做去重，计算共同爱好，独有爱好，共同好友等。
redisTemplate.opsForSet().add("citySet",citys)  
redisTemplate.opsForSet().remove("citySet",citys)  
交集：
redisTemplate.opsForSet().intersect("citySet1","citySet2")；  
并集： 
redisTemplate.opsForSet().union("citySet1","citySet2")；  
差集：
redisTemplate.opsForSet().difference("citySet1","citySet2")；  

### sorted set 
redisTemplate.opsForZSet().add("zset2", "linqz", 9.6);  
redisTemplate.opsForZSet().range("zset2", 0, -1);  
redisTemplate.opsForZSet().count("zset2", 0, 8);  
redisTemplate.opsForZSet().size("zset2");  



## Mongodb的个人理解和总结



### Mongodb是一个基于分布式文件存储的数据库。
支持字段索引，优势在于查询功能强大，能存储海量数据，只支持单文档事务。

它面向集合存储，意思是数据是被分组存储在数据集合中的，每个集合在数据库中有唯一的标识名，集合的概念类似于关系型数据的表。
在集合中存储的是文档，被存储为键-值对的形式，键是每个文档的唯一标识，为字符串类型，值可以是各种的文件类型，这种存储形式为bson，文档的概念类似于数据库表中的一行数据。  

特点是：性能高，使用方便，存储数据方便。

### 使用场景，做服务器的日志记录，查找方便，导出也方便。  
存储监控数据，增删字段不需要修改表结构，  
存储大量的商家信息。  

### 与mysql的比较，  
mysql查询语句是传统的sql语句，拥有成熟的体系海量数据处理时效率显著变慢。  

mongodb：非关系型数据库，属于文档型数据库。（可以存放xml,json,bson类型的数据。存储方式：虚拟内存+持久化。  

### 数据类型：null,布尔，数值，字符串，日期，数组，内嵌文档，对象id。  
每个文档必须有一个_id键，可以是任意类型，默认是Objectid,在一个集合里有唯一的_id。  
  
### 数据库操作：
新增数据库：use db1(db1为数据库名) #有这个数据就使用这个，没有则创建（很方便简洁）。查询数据：show dbs。  
删除数据库：db.dropDatabase()

### 集合（表）的增删改：
新增：db.db1.info #db1.info是表名 或者直接插入文档，集合也会被创建。 db.table1.insert({'a',1})  

查询：show tables

删除集合：db.table1.drop()  


### 文档（表中一行记录）的增删改，

user1={
"name":"lin",
"age":25,
'hobbies':['music','read'],
'addr':{
'country':'China',
'city':'GZ'
}
}


新增一条数据：db.table1.insert(user1)  
新增多条记录：db.table1.insertMany([user1,user2])  

查询一条记录：db.table1.findOne()  
查询所有记录：db.table1.find()   
带条件查询：db.table1.find({"age":25})  
比较运算'!=' db.table1.find({"age":{"$en":25}})  
运算符'<' {"age":{"$lt":25}},'>'{"age":{"$gt":25}},'<='{"age":{"$lte":25}},'>='{"age":{"gte":25}}。  

逻辑and db.table1.find( {"age":{"$gte":24,"$lte":25},"name":"lin"})
逻辑or   db.table1.find({"or":[{"age":{"$gte":24,"$lte":25}},{"name":"lin"}]})


修改表中字段：db.table1.update({'age':25},{"name":"linqz"})
	       db.table1.update({"_id":1},obj)

 
### Mongodb在springboot中的使用，

#### 1.首先是在项目中导包，

       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>

#### 2配置文件中配置mongodb

spring.data.mongodb.host=120.67.195.135 //主机ip  
spring.data.mongodb.port=27017	//mongodb的端口号  
spring.data.mongodb.database=linDatabase 数据库名  
或者 
spring.data.mongodb.uri=mongodb://120.67.195.135 :27017/linDatabase  


#### 3使用 

@Autowired
    private MongoTemplate mongoTemplate;  
新增：mongoTemplate.save(user);  

根据条件查询:
 Query query = new Query(Criteria.where("name").is(name));  
 User user = mongoTemplate.findOne(query,User.class);  
	
根据条件更新:  
 Query query = new Query(Criteria.where("id").is(user.getId()));   
 Update update = new Update().set("name",user.getName()).set("password",user.getPassword());	  
 UpdateResult result = mongoTemplate.updateFirst(query,update,User.class);  

根据条件删除:
 Query query = new Query(Criteria.where("id").is(id));  
 mongoTemplate.remove(query,User.class)；  



	
# Elasticsearch的个人理解和总结

ElaticSearch，简称为ES， ES是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。ES也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的Restful Api来隐藏Lucene的复杂性，从而让全文搜索变得简单。

## ES核心概念：

#### (1) cluster（集群）：每个集群至少包含两个节点.
#### (2) node：集群中的每个节点，一个节点不代表一台服务器
#### (3) field：一个数据字段，与index和type一起，可以定位一个doc
#### (4) document：ES最小的数据单元 Json
#### (5)Field: 定义每个document应该有的字段。
#### (6) Index：索引 包含一堆有相似结构的文档数据
#### (7) shard：分片
index数据过大时，将index里面的数据，分为多个shard，分布式的存储在各个服务器上面。可以支持海量数据和高并发，提升性能和吞吐量，充分利用多台机器的cpu。
#### (8) replica：副本
在分布式环境下，任何一台机器都会随时宕机，如果宕机， index的一个分片没有，导致此index不能搜索。所以，为了保证数据的安全，我们会将每个index的分片经行备份，存储在另外的机器上。保证少数机器宕机es集群仍可以搜索。
能正常提供查询和插入的分片我们叫做主分片（primary shard），其余的我们就管他们叫做备份的分片（replica shard）。
es6默认新建索引时， 5分片， 2副本，也就是一主一备，共10个分片。所以， es集群最小规模为两台。

##  ElasticSearch与SpringBoot的整合

#### 导入依赖：
<dependency>  
	<groupId>org.springframework.boot</groupId>  
	<artifactId>spring-boot-starter-data-elasticsearch</artifactId>  
</dependency>  

#### 进行配置：
  data:  
    elasticsearch:  
      repositories:  
        enabled: true  
    cluster-nodes: 120.56.195.135:9300 # es的连接地址及端口号  
    cluster-name: elastic # es集群的名称
    
#### 创建一个查询实体：  
@Data  
@Builder  
@Document(indexName = "book", shards = 3, replicas = 0)  
public class Book {  
    @Id  
    private Long id;  

    @Field(name = "author_name", type = FieldType.Keyword, store = true)  
    private String authorName;   

    //存储分词为ik_max_word, 查询分词为ik_smart  
    @Field(name = "book_name", store = true, analyzer = "ik_max_word", searchAnalyzer = "ik_smart")  
    private String bookName;   
  
    @Field(name = "desc", store = true, analyzer = "ik_max_word", searchAnalyzer = "ik_smart")  
    private String desc;  

    @Field(name = "pub_date", type = FieldType.Text, format = DateFormat.custom, pattern = "yyyy-MM-dd HH:mm:ss", store = true)  
    private Date pubDate;   

    @Override
    public String toString() {  
        return "Book{" +  
                "id=" + id +  
                ", authorName='" + authorName + '\'' +  
                ", bookName='" + bookName + '\'' +    
                ", desc='" + desc + '\'' +  
                ", pubDate=" + pubDate +  
                '}';  
    }  
}  


#### 索引操作：  
public class Index {  

    @Autowired
    private ElasticsearchRestTemplate restTemplate;  

    /**
     * 创建索引
     */
    @Test
    public void testCreateIndex() {  
        boolean isCreated = restTemplate.indexOps(Book.class).create();  
        log.info("test create index : {}", isCreated);  
    }  

    /**
     * 查询索引是否存在  
     */  
    @Test  
    public void testExistsIndex() {  
        boolean isExists = restTemplate.indexOps(Book.class).exists();  
        log.info("test exists index : {}", isExists);  
    }  

    /**
     * 删除索引
     */
    @Test
    public void testDeleteIndex() {
        boolean isDeleted = restTemplate.indexOps(Book.class).delete();
        log.info("test delete index : {}", isDeleted);
    }
}

#### 新增文档：
public class Document {  

    @Autowired  
    private ElasticsearchRestTemplate restTemplate;  

    /**
     * 新增 or 全量更新  
     */
    @Test
    public void testAdd() {  
        Book book = Book.builder()  
                .id(1L)  
                .authorName("lin")  
                .bookName("ElasticSearch")  
                .desc("学编程使我快乐-1")  
                .pubDate(new Date())  
                .build();  

        IndexQuery indexQuery = new IndexQueryBuilder()  
                .withId(book.getId().toString())  
                .withObject(book)  
                .build();  
       
        IndexCoordinates indexCoordinates = restTemplate.getIndexCoordinatesFor(Book.class);  
        String documentId = restTemplate.index(indexQuery, indexCoordinates);  
        log.info("test index add content : {}", documentId);  
    }  


#### 查询数据：

public class SearchQuery {  

    @Autowired
    private ElasticsearchRestTemplate restTemplate;  

    /**
     * 简单查询
     */
    @Test
    public void search1() {  
        Criteria criteria = new Criteria()  
                .and("author_name").is("lin")  
                .and("book_name").contains("java");  
        Query query = new CriteriaQuery(criteria);  
        IndexCoordinates indexCoordinates = restTemplate.getIndexCoordinatesFor(Book.class);  
        SearchHits<Book> search = restTemplate.search(query, Book.class, indexCoordinates);  
        log.info("search1----{}",search.getTotalHits());  
    }  

    /**
     * 标准查询
     */
    @Test
    public void search2() {  
        Query query = new NativeSearchQueryBuilder()  
                //查询条件
                .withQuery(QueryBuilders.matchQuery("book_name","java"))  
                //分页  
                .withPageable(PageRequest.of(0, 5))  
                //排序  
                .withSort(SortBuilders.fieldSort("id").order(SortOrder.DESC))  
                //高亮字段显示 及自定义tag  
                .withHighlightFields(new HighlightBuilder.Field("book_name").preTags("<span>").postTags("</span>"))  
                .build();  
        IndexCoordinates indexCoordinates = restTemplate.getIndexCoordinatesFor(Book.class);  
        SearchHits<Book> search = restTemplate.search(query, Book.class, indexCoordinates);  
        log.info("search1----{}", search.getTotalHits());  
    }  

















     


