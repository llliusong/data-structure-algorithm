MySQL
---



* MySQL的引擎:
    -   分为Innodb和Myisam




# 索引

* 什么是索引:索引是指数据库管理系统中的一个排序的数据结构,索引的两大类型有b tree,hash
    -   hash索引: 基于hash的特性,检索效率非常的高,一次定位,不需要像b tree一样多次io
        -   缺点: 
            -   仅仅只能满足"=","IN",和"<=>"查询,`不能使用范围查询`,`因为hash当数据变更之后并不能保证变更后的hash与变更前的一致`
            -   `Hash索引无法用来排序`,原因一样,hash一旦数据变更之后就不能保证前后一致
            -   `Hash索引不能利用部分查询`,当通过组合索引的时候,hash索引会将索引和相加,而不是单独计算,组合索引也就无效了
            -   `无法避免全局扫描`,因为可能不同的值具有相同的hash索引
            -   `大量hash碰撞后,并不能保证效率比b tree 高`
    -   b tree索引(是一种k,v结构的树): b tree 是一种多路平衡二叉树,k,v在同一个节点上
        -   具有如下特征:对于`m阶的b tree`
            -   根节点的孩子数目为[ceil(m/2),m],关键字的数目为[ceil(m/2)-1,m-1] 
            -   **非叶节点的根节点至少有2个孩子节点**
            ![](https://img-blog.csdnimg.cn/20190212005404642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NvZGVyX0pva2Vy,size_16,color_FFFFFF,t_70)
            btree中叶子节点和根节点以及分支节点都是同一种类型(class)
    -   b+ tree索引(也是一种k,v结构的树): b+ tree 也是一种多路平衡二叉树,但是与b tree的不同点在于,非叶子节点不存放值v,只存放键k****
        -   具有如下的特征:对于`m阶的b+ tree`
            -   根节点的孩子数目为:[ceil(m/2),m],关键字的数目为[ceil(m/2)-1,m]
            -   根节点和分支节点只用来保存关键字(索引),数据地址都存放在叶子节点上
            ![](https://img-blog.csdnimg.cn/20190212005018445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NvZGVyX0pva2Vy,size_16,color_FFFFFF,t_70)     
            叶子节点中的5,8,9分别就是数据库中的记录,因此**b+tree中叶子节点与根节点和分支节点不是同一种类型(class)**
    -   为什么b+tree 更适合作为索引:`因为btree只是提高了磁盘io的性能,但是并没有解决元素遍历效率低下的问题`
        -   在mysql中每次读取数据都是以页为单位,而每页的数据由操作系统而定,4k或者8k或者16k,但是**磁盘io是昂贵的操作,因而操作系统会预读,既多读相连的几页数据**
        -   b+tree因为数据都在叶子节点,且叶子节点都是有序的,这也就导致了范围查询的时候b+tree比btree要快太多了,也因为如此使得数据页中的数据量尽可能的多,从而减少了io次数
    -   b+tree 索引可以分为聚集索引和辅助索引,`聚集索引的b+tree存放的是行记录数据,而辅助索引存放的是,当通过辅助索引的时候先通过辅助索引找到聚集索引键,然后聚集索引键在聚集索引中找到数据`    
    -   索引设置的疑问点:
        -   为什么`索引要尽可能的小`:因为io次数与b+tree的高度有关,原因在于`每个磁盘页的大小是一定的`,数据项占的空间越小->从而每页占据的数据量也就越多->树的高度也就会越低,这同时也是为什么将数据值都放在叶子节点的缘故
        -   `索引的最左匹配原则`: b+树的key项是复合索引结构,是按照从左到右建立搜索树的,如(a,b,c)索引,优先比较a,然后b,最后c
* 索引的分类:
    -   普通索引(name): 仅用户加速查找
    -   唯一索引: 
        -   主键索引: primary key(id) `加速查找且约束:不为空且唯一`
        -   唯一索引: unique(uuid)    `加速查找且约束:唯一`
    -   联合索引:
        -   联合主键索引:(id,name)
        -   联合唯一索引:(uuid,name)
        -   联合普通索引:(name,asd)
    -   全文索引: 用于搜索一篇很长的文章的时候
    
    -   `聚集索引`:表中各行的物理顺序与物理磁盘键值的逻辑(索引)顺序一致,`表中只能包含一个聚集索引,主键列默认为聚集索引`
        -   如果`主键被定义了`: 则这个主键就是聚集索引
        -   若主键未定义,则这个表的`唯一非空索引作为聚集索引`
        -   若没有主键也没有唯一非空索引,则会自动生成一个`隐藏的主键,会自动递增`
    TODO sql编写涉及索引的原则,最左匹配有点糊涂了
    -   `非聚集索引,也可以称为辅助索引`:  与物理磁盘的顺序不一致,叶子节点上存放的是指向聚集索引的指针
    
    
# MySQL的存储引擎

* InnoDB

* MyISAM

* Memory