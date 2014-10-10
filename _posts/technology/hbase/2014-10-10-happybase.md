---
layout: post
title:  "happybase"
date:   2014-10-10 12:00:00
categories: hbase
---

### happybase 快速入门
#### 连接数据库

```python
connection = happybase.Connection('thrift 服务器地址', autoconnect=False)      #不自动连接
connection.open()

# 或者

connection = happybase.Connection('thrift 服务器地址')     #自动连接

# 或者

connection = happybase.Connection('thrift 服务器地址', table_prefix='myproject') 
# 这样的连接，以后所涉及的表实际都是以这个 table_prefix + '_' 开头的，后面再访问表的时候，就不需要再加 table_profux + '_' 了
```

#### 表操作

```python
# 获取所有表格列表
connection.tables()

# 创建表格
connection.create_table(
    'mytable',
    { 'cf1': dict(max_versions=10),
      'cf2': dict(max_versions=1, block_cache_enabled=False),
      'cf3': dict(), # use defaults
    })      
# 第二个参数是个 map<string, map> 的结构，外层的 map 的 key 是列族名，对应的 value 是一个 map 用于设置对应的列族的参数配置，可以带上一些参数如：max_version、block_cache_enables

# 获取表对象
table = connection.table('mytable')
# 这个方法不会检查表格是否存在，也就是在用 table 的表的某些方法的时候才会出现异常
```

#### 读数据

```python
# 获取单行数据
row = table.row('row-key')    # 通过 row key 获取一行的信息
print row['cf1:col1']               # 获取指定 family:colum 的值

# 获取多行数据：
rows = table.rows(['row-key-1', 'row-key-2'])
for key, data in rows:
    print key, data   # 通过指定多个 key 的值，可以返回对应的 key 和 value 的组合，注意这里可不是指定一定范围的 key，用于返回一堆
    rows_as_dict = dict(table.rows(['row-key-1', 'row-key-2']))

# 获取多行有序map：
from collections import OrderedDict
    rows_as_ordered_dict = OrderedDict(table.rows(['row-key-1', 'row-key-2'])   # 将 rows 方法返回的 key、value 存入 map 或者放入有序 map

# 获取指定列的数据（默认情况下。row 或者 rows 方法会获取行的所有列（而大多数的列，在这些相应的业务处理的时候都是用不到的，所以可以指定获取那些 colum，减少传输数据的大小）：
row = table.row('row-key', columns=['cf1:col1', 'cf1:col2'])
print row['cf1:col1']
print row['cf1:col2']

row  = table.row('row-key', columns=['cf1'])                # 也可以只是指定列族
row  = table.row('row-key', timestamp=123456789)   # 同时可以指定时间片信息，但是请注意一定得是数字(>=0)，不可以是字符串
row  = table.row('row-key', columns=['cf1:col1'], include_timestamp=True)
value, timestamp = row['cf1:col1']
rows = table.rows(['h1', 'h5', 'h3'], include_timestamp=True)   # 对于 include_timestamp 也可以用于 rows 方法
 
for key, value in rows:
    print key, value   # 返回的结果类似一个 map，key 为相应的 row_key，value 也为 map，不过其值不光是 value 还有 value 对应的时间片，组成了一个数组。

# 获取多版本的信息
values = table.cells('row-key', 'cf1:col1', versions=2)
for value in values:
    print "Cell data: %s" % value

cells = table.cells('row-key', 'cf1:col1', versions=3, include_timestamp=True)
for value, timestamp in cells:
    print "Cell data at %d: %s" % (timestamp, value)
# 一般都这格式，value 带时间片的，都是一个先 value，再时间片的数组
```

#### 表格遍历

```python
for key, value in table.scan():
    print key, value

# 这里 key 就是 row_key，value 为一整行的数据，都是 map，key 为 famliy:colname 形式，和 row 方法或者 row 方法返回的 value 值是一样的，可以在 scan 参数中指定 row_start='aaa' 或者 row_end='bbb'，用于限定范围，同事支持 row_key 的前端匹配，通过 row_prefix，详细的还是看 scan 的 api 吧。功能很强大
```

#### 写数据

```python
# 传入给 hbase 的数据都是原始的字节字符串（翻译来的）
table.put('row-key', {'cf:col1': 'value1','cf:col2': 'value2'})               # 跟 shell 的差不多，key 然后是个 colum 与 value 的 map
table.put('row-key', {'cf:col1': 'value1'}, timestamp=123456789)   # 可以手动加入时间片，注意一定是 integer 的

# 批量写
b = table.batch(timestamp=123456789)
b.put( ...)   # 这里的 timestamp 可以不指定，也可以在 put 的时候指定
b.send()

# 或者是与 with 集合，能起到即使有异常抛出，也会进行 b.send() 的操作的目的，不过从某种角度，这就不是一个完整的 transation 了
with table.batch() as b:
    b.put('row-key-1', {'cf:col1': 'value1', 'cf:col2': 'value2'})
    b.put('row-key-2', {'cf:col2': 'value2', 'cf:col3': 'value3'})
    b.put('row-key-2', {'cf:col2': 'value2', 'cf:col3': 'value3'})
    b.put('row-key-3', {'cf:col3': 'value3', 'cf:col4': 'value4'})
# 注意 put 和 delete 操作可以放在一个 batch 里头，但是对同一行执行 put 和 delete 操作，放到一个 batch 里头其表现是不可预测

# 当然因为需要插入多条数据或者数据本身很大，导致数据量会很大，这时候就需要设置 batch 的大小，以做优化
with table.batch(batch_size=1000) as b:
    for i in range(1200):
        b.put('row-%04d' % i, {'cf1:col1': 'v1', 'cf1:col2': 'v2',})
# 这里也是可以 put 多个列的，用 map。特别需要注意的是在 batch 里头 transaction=True, batch_size=1000，这两个属性是不能同时使用的
```

#### 删除数据

```python
table.delete('row-key')
table.delete('row-key', columns=['cf1:col1', 'cf1:col2'])
```

#### 计数器

计数器这东西如果想要增减或者减少的话，最好不要通过 set 方法，应该用 increment 和 decrease 这些原子方法

#### 连接池

```python
pool = happybase.ConnectionPool(size=3, host='...', table_prefix='myproject')     
# 连接池会先发布一个真正的连接，所以针对连接池的情况下，能很早的发现主机名等配置的错误，而不用特殊的配置，当然这个还没强大到会检查这个 table_prefix 是否真的存在。
# 当然上面的 host 配置成...，铁定是要出问题的，会报错：thrift.transport.TTransport.TTransportException: Could not connect to ...:9090，也就是说默认其实是连接 9090，这接口居然真的将'...'和9090拼到了一起。

with pool.connection() as connection:
    print connection.tables()    # 这方法很犀利，with 块过了，连接就返回给了连接池，所以过了 with 块，就别想着复用这个 connection 了，需要用再另外申请打报告吧

with pool.connection() as connection:
    table = connection.table('table-name')
    row = table.row('row-key')
    process_data(row)    # 既然你使用连接池，就说明，连接池很宝贵，所以尽量少占用抢过来的连接所独占的时间，数据获取到了就行，处理逻辑放到with块外面，别一边获取，一边处理
    pool = happybase.ConnectionPool(size=1, host='localhost', table_prefix='ml')

with pool.connection() as connection:
    table = connection.table('one_family')
    table.put('row_xxoo_1', {'fam1:col1':'row_xxoo_1_value'})

with pool.connection() as connection:
    table = connection.table('one_family')
    table.put('row_xxoo_2', {'fam1:col1':'row_xxoo_2_value'})

with pool.connection() as connection:
    table = connection.table('one_family')
    table.put('row_xxoo_3', {'fam1:col1':'row_xxoo_3_value'})     # 对于同一个线程，获取的 connection 是同一个的，所以3个都成功了，而不是卡住了什么的

# 连接这东西，不是获取了就啥错也不会报，如果连接出了问题，如超过租用时间等等，该有错还是会报，虽然连接池在收回这个连接的时候，会修复，但是实际接收异常、处理异常还是要自己编码完成的。
```

### happybase API
#### 表管理（Connection 类）
***class happybase.Connection( host='localhost' , port=9090 , timeout=None , autoconnect=True ,table_prefix=None, table_prefix_separator='_' , compat='0.96' , transport='buffered' )***
> timeout 单位为毫秒，这个超时的属性，貌似不是在调用这个 Connection 方法的时候有效的，而是调用这个方法返回的连接对象所对应的一些需要去服务器请求信息的操作时候用到的。
> table_prefix 和 table_prefix_separator 你看就明白，就像定义了一个明明空间，通过这个 connection 弄出的操作的表，实际上都是以 table_prefix + table_prefix_separator开头的，这个分隔符居然能改，很人性化啊 
> autoconnect 这个属性也许有点用，如果你急着建立连接对象，但是又不想让占用太多连接资源，那就设置成 false，真正用的时候，在 open 一下 
> compat 这个属性，初看，会把他误认为是 compact，也就是 hbase 在 regionserver 上面进行的文件合并操作，实际上这个词是。。compatibility，兼容性，没查到具体解释，个人认为这个数字指定的应该是 hbase 的版本，注意这个是个字符串，也就是表达这个客户端的协议与哪个版本的 hbase 的相关协议兼容。而且必须是 0.90, 0.92, 0.94，这几个中的一个，虽然默认是 0.96，但是这个值你不能显示的指定。
> transport 这个没太看明白，跟 thrift 有点关系，英文说明如下： The optional transport argument specifies the Thrift transport mode to use. Supported values for this argument are buffered (the default) and framed. Make sure to choose the right one, since otherwise you might see non-obvious connection errors or program hangs when making a connection. HBase versions before 0.94 always use the buffered transport. Starting with HBase 0.94, the Thrift server optionally uses a framed transport, depending on the argument passed to the hbase-daemon.sh start thrift command. The default -threadpool mode uses the buffered transport; the -hsha, -nonblocking, and -threadedselector modes use the framed transport.

***close( )***
> 平常一般都不 close，不过 close 之后，这个 connction 对象就不能用了，很好理解，tcp 连接断了

***compact_table( name , major=False )***
> compact table 的操作懂吧？就是将 region_server 上的 hfile 文件合并，一种是普通的合并，没太多特殊操作，一种是 major 合并，这时候一些多余的数据，根据逻辑会被删掉（例如有删除标记的，一些老的数据就会被删掉，或者是根据版本保存的个数显示，进行相应的数据丢弃），需要注意的是，这可是一个异步调用，很快就会返回，相应的服务器会进行compact 操作，耗费时间

***create_table( name , families )***
> families = { 'cf1': dict(max_versions=10), 'cf2': dict(max_versions=1, block_cache_enabled=False), 'cf3': dict(), # use defaults }
> connection.create_table('mytable', families) 
> 因为创建一个表，可能会有多个列族，每一个列族又会有多个属性的配置（属性有名称和值），用 map 表示列族的信息，再合适不过。这操作一般是由管理员进行。
> 命名风格：python 方法里头的参数名称都是 xx_xx_xx 的格式，也就是不用驼峰表示，都是小写单词以底杠 
> max_versions：这属性用于设置一个列的保存的版本数量 
> compression：压缩算法，我知道的 lzo，不过貌似需要在 region server 上安装跟这个压缩算法有关的一个东东，才能用 
> in_memory、bloom_filter_type、bloom_filter_vector_size、bloom_filter_nb_hashes、time_to_live：文档里没说 
> block_cache_enable：块缓存

***delete_table( name , disable=False )***
> 这个方法默认不会 disable 表的，如果表不事先 disable，或者这里 disable 没有设置为 True，会抛异常

***disable_table( name )、enable_table(name)***
> 这俩方法，功能跟方法名疑似差不多，就是废掉表和让表在此的工作

***is_table_enabled( name )***
> 判断表是否能工作，这方法需要注意的是，如果表名不存在，这方法也返回 true

***open()***
> 打开一个底层的 tcp 连接，因为在创建 connection 的时候，autoconnect 默认是 true 的，所以创建连接的时候，这个就打开了，不过如果 autoconnect=False，那就在使用的时候需要 open 一下

***table( name , use_prefix=True )***
> 首先，这方法不会去在底层检查一个表是否真的存在。但是当利用这个方法返回的 table 对象进行操作的时候，这表名如果不存在，就会有 ioerror。
> 其次，user_prefix 默认是 True，也就是传点的连接的时候的表明前缀的设置默认是会添加上的，如果想用一个非指定前缀的表，那就要设置为 False

***tables( )***
> 返回该连接对应的表名列表，是个数组，如果创建连接的时候指定指定了前缀，返回的就是这个前缀的对应的表名列表

#### 数据管理（Table 类）
***class happybase.Table( name , connection )***
> API 里说这个方法不能用于直接初始化，但是事实证明。。是可以的

***batch( timestamp=None, batch_size=None, transaction=False, wal=True)***
> 通过 table 对象，建立 batch 对象，timestamp 指定的话，这个 batch 里头的操作默认都用这个时间片（put、delete 操作）。
> bath_size 是一个 batch 最多能放下多少个操作，也就一种情况是操作所需的空间比较大，没到 bath_size 就给发出去了，一种是操作的数量凑够 batch_size 大小，然后 send 出去了。
> transaction 这东西是跟 with 语句配套使用的，并且不能与 batch_size 同时使用。
> wal 就是是否写 wal 的日志。

***cells(row, column, versions=None, timestamp=None, include_timestamp=False)***
> 返回一个包含存储的值的数组（如果 include_timestamp=False）
> versions 挺重要的，最好指定下，因为这个默认为 None 的 versions，却返回了所有的 version 
> timestamp 是对时间片的限定条件
> include_timestamp 为 True 在返回的 value 后会带上一个时间片，共同作为数组的一个元素的内容。

***counter_dec(row, column, value=1)***
> 对计数器进行减值，默认为1。需要注意的是，计数器跟平常存入的值不能混着来。计数器就是计数器，因为平常发发存入的值，都是字符串，而计数器是数值。value 可以为负值，相当于给计数器增加值。

***counter_inc(row, column, value=1)***
> 与 counter_dec 相反。

***counter_get(row, column)***
> 获取计数器的值（整形），column 不存在返回 0

***counter_set( row , column , value=0 )***
> 设置计数器的值，但是方法虽有，最好别用，有 inc、dec 方法，这俩方法是原子操作。

***delete( row , columns=None , timestamp=None , wal=True )***
> 删除行或者列族或者列，其中 colums 是个 list，里面的元素为字符串，字符串可以是列族名，或者是 "列族:列名"

***families( )***
> 这方法返回的一个 map，map 的 key 为列族的值，value 为列族配置的一堆 key，value 的配置

***put( row , data , timestamp=None , wal=True )***
> data 是个列族或者 “列族:列” 为 key 的 map，其中列族的 key 后面可以在接一个":"，而后面不放列

***regions( )***
> 获取 table 的 region 列表，列表的每一个元素都是关于一个 region 的一些属性信息的 map
> 例如: 
> { 'name': 'ml_oneday_baiduid_plus,,1395219468099.2f4b8037ff3c5155f96205a86bdb0a7a.', 'server_name': 'szjjh-serp62.szjjh01.baidu.com', 'port': 8620, 'end_key': '028f5c28', 'version': 1, 'start_key': '', 'id': 1395219468099 } 
> { 'name': 'ml_oneday_baiduid_plus,028f5c28,1395219468099.6851db9caa7a305f0bc00fb21c4c1593.', 'server_name': 'szjjh-serp57.szjjh01.baidu.com', 'port': 8620, 'end_key': '051eb850', 'version': 1, 'start_key': '028f5c28', 'id': 1395219468099 }

***row( row , columns=None , timestamp=None , include_timestamp=False )***
> 返回指定行的全部列或者部分列族或列
> colums 是个列表，元素为列族或者列，值得注意的是这个返回的数据是个 map，所以即使在 colums 指定的信息，会返回多个重复信息，因为结果是 map，所以是唯一的，会去重
> 结果如：
> {'fam1:': 'xxvaluekjajdljlfajsld', 'fam1:col2': 'xxvalue2'}
> 如果 timestamp=True 结果：
> {'fam1:': ('xxvaluekjajdljlfajsld', 1395619333514), 'fam1:col2': ('xxvalue2', 1395619732386)} 
> 值得注意的是，如果参数的名称指定错误的话，两种情况，一种呢会报错，一种不报错，但是空结果返回，所以名称一定要指定

***rows( rows , columns=None , timestamp=None , include_timestamp=False )***
> 其中 rows 为 row 的列表。返回值为 map，key 为 row_key，value 与 row返回的东西差不多

***scan( row_start=None, row_stop=None, row_prefix=None, columns=None, filter=None,timestamp=None, include_timestamp=False, batch_size=1000, scan_batching=None, limit=None,sorted_columns=False)***
> 跟 shell 的 scan 对应。返回的是个迭代器。返回的结果对应的 key 值 >= row_start，< row_stop
> row_start、row_stop 与 row_prefix 不能同时设置
> filter 是个过滤字符串，咋用呢，不知道啊？
> limit 限制返回的记录个数
> batch_size 对应的做多一次可以获取多少的记录（以行为单位）
> scan_batching 比 batch_size 计算的粒度更细，行会被拆分，对应每个单元格，这样会导致行被拆分
> sorted_columns 这个属性个人感觉没啥太大用处，就是返回的行的列 cell，按照列名会进行排序。。。 
> 还有就是 filter 这个过滤字串在 0.92 一下的版本的是不能用的，会有异常。

#### 批量数据管理（Batch 类）
***class happybase.Batch( table , timestamp=None , batch_size=None , transaction=False , wal=True )***
> 说是这个方法不能直接用，但是确实能用的，建议使用 Table.batch() 方法

***delete( row , columns=None , wal=None )***
> batch 里头的 delete 方法

***put( row , data , wal=None )***
> batch 里头的 put 方法，至于 wal 是干什么的。。真心没看懂，文档里是这么说的：The wal argument should normally not be used; its only use is to override the batch-wide value passed to Table.batch().

***send( )***
> 这个操作，如果你不用 with 的话，是必须要有的，不然 batch 真的就不会把操作发出去的。如果设置了 batch_size 的话，如果进行的操作数量凑够了 batch_size 的数量的话，那么send 会自动触发，当 batch_size 设置成1的话，数据丢不了，但是用 batch 就没啥意义了。

#### 线程安全，连接复用（ConnectionPool 类）
***class happybase.ConnectionPool( size , **kwargs )***
> 创建连接池
> size 为大小
> kwargs 为一些关键字参数，有 host、table_prefix（是否还有其他参数，以及是什么，文档里没有说）

***connection( *args , **kwds )***
> with pool.connection() as connection: pass # do something with the connection 
> 从连接池获取连接，可以设置 timeout 属性（单位为秒），timeout 设置时间内如果不能得到一个有效的连接，会抛出异常。如果不进行设置的话，就会无限的等待。

***class happybase.NoConnectionsAvailable***
> 获取有效连接超时，会抛出的异常

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

