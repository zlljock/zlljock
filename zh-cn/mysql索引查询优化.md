索引的本质

索引是帮助mysql高效获取数据的排好序的数据结构

索引结构在文件中

索引结构

HASH在查找范围的时候没有优势

B+Tree

非叶子节点不存储data，只存储索引，一颗放更多的索引，可以增大度 

叶子节点包含所有索引字段

叶子节点但用指针连接，提高区间访问的性能

data数据放在叶子节点上把，，，中间数据作为冗余字段

理解如果一个节点的大小值设置的是固定的话，在没有data的时候我们可以放入更多的索引元素，这样我们在存储同样多数的时候数的高度就会变少

B+Tree的指针有着范围查找的效果

一般来说mysql每个节点16K ，一个元素是8B，一个指针6B，第一行节点差不多是16000B/14B=1100,每个指针一个节点 1100*1100，叶子节点每个存1K的话可以存16个，这样算的话差不多是几千万的数据吧（如果深度为3）适用高度差不多是2-4

![image-20200518163836994](C:\Users\drew\AppData\Roaming\Typora\typora-user-images\image-20200518163836994.png)

索引底层数据结构与算法

B-Tree

度-节点的数据存储个数

叶节点具有相同的深度

叶节点的指针为空

节点中的数据key从左到右递增排列

数据库下的

.frm

表结构定义的数据data

.MYD

存储表的数据data

.MYI

这张表存储的索引的位置index

通过B+Tree查到到的数据然后就在MYI文件中去定位

.ibd

所以数据存储在一个文件中

#### InnoDB索引实现（聚集）存储引擎

 表数据文件本身就是按B+Tree组织的一个索引结构文件

聚集索引-叶节点包含了完整的数据记录

为什么InnoDB表必须有主键，并且推荐使用**整型**的自增主键？UUID是字符串存储的占位比较大所以不推荐用UUID类型的，自增是因为比大小比较容易，如果不是按自增的话插入原有的节点的是时候会分裂的

为什么非主键索引结构叶子节点存储的是主键值？（一致性和节省存储空间）如下图：

主键索引和数据在同一个文件

![image-20200518175012950](C:\Users\drew\AppData\Roaming\Typora\typora-user-images\image-20200518175012950.png)

#### 聚集索引整个索引是在一个文件里面，数据和索引是存储在一起的

#### 非聚集索引主键文件存储在MYI文件中，数据结构是在MYD文件中

