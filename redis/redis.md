![在这里插入图片描述](https://img-blog.csdnimg.cn/1058c7a72d3643ad89504863ef4165a2.png)


# 一、Ubuntu环境

## 1. 安装：
		redis：基于内存的高效非关系型数据库
		redis-py：用redis-py库与redis交互
		redis-dump：用于redis数据导入/导出的工具，基于ruby实现，因此需提前安装ruby
```python
	# redis安装
	$ sudo apt-get -y install redis-server
	
	# redis-py安装
	$ pip3 install redis
	
	# redis-dump安装(需提前安装ruby)
	$ gem install redis-dump
```
## 2. 验证： 
```python
	# redis安装验证 
	$ redis-cli # 进入Redis命令行模式
	127.0.0.1:6379> set 'name' 'dafei'
    OK
    127.0.0.1:6379> get 'name'  
    'dafei'
    
    # redis-py验证
    $ python3
    >>> import redis
    >>> redis.VERSION
    (2, 10, 5)
    >>>
	
	# redis-dump
	# redis-load 
```
## 3. 远程连接：
#### 3.1 修改配置文件：/etc/redis/redis.config
		
	1 注释该行：
```python 
	bind 127.0.0.1
```
	2 设置密码：​foobared即当前密码
```python
	requirepass foobared
```
#### 3.2 使用Redis服务
```python
	$ sudo /etc/init.d/redis-server restart # 重启
	
	# redis-cli 本地连接：redis-cli 、auth password
	(base) root@huzi-pc:~# redis-cli
	127.0.0.1:6379> get name
	(error) NOAUTH Authentication required.
	127.0.0.1:6379> auth 9522
	OK
	127.0.0.1:6379>
	
	# redis-cli远程连接：redis-cli -h ip -p 6379 -a password
	(base) root@huzi-pc:~# redis-cli -h 164.92.141.135 -p 6379 -a 8888
	Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
	164.92.141.136:6379> get name
	(nil)
	(1.09s)
											
	$ sudo /etc/init.d/redis-server stop # 停止
	$ sudo /etc/init.d/redis-server start # 启动
```
#### 3.3 python的redis操作
###### 1. 关于Redis和StrictRedis:
	   redis-py库提供两个类Redis和StrictRedis进行Redis命令操作.推荐用StrictRedis
	   
###### 2. 连接Redis
```python
	from redis import StrictRedis
	redis = StrictRedis(host='localhost',port=6379,db=0,password='foobared')
	redis.set('name', 'Bob')
	print(redis.get('name'))
```
		还可以用ConnectionPool来连接，如下：
```python
	from redis import StrictRedis, ConnectionPool
	pool = ConnectionPool(host='localhost',port=6379,db=0,password='foobared')
	redis = StrictRedis(connection_pool=pool)
```
		另外，ConnectionPool 还支持通过 URL 来构建。URL 的格式支持有如下 3 种：
```python
	redis://[:password]@host:port/db  # RedisTCP连接
	rediss://[:password]@host:port/db  # RedisTCP+SSL连接
	unix://[:password]@/path/to/socket.sock?db=db # Redis UNIX socket连接
```
		URL连接示例
```python
	url = 'redis://:foobared@localhost:6379/0'  
	pool = ConnectionPool.from_url(url)  
	redis = StrictRedis(connection_pool=pool)
```	 
###### 3. 键操作

###### 4. 字符串操作

###### 5. 列表操作

###### 6 集合操作
###### 7. 有序集合操作

###### 8. 散列操作
###### 9. RedisDump
	数据导出：redis-dump
	数据导入：redis-load
	
	先看redis-dump数据导出，如下：
```python		
$ redis-dump -h
Usage: redis-dump [global options] COMMAND [command options]   
    -u, --uri=S                      Redis URI (e.g. redis://hostname[:port])  
    -d, --database=S                 Redis database (e.g. -d 15)  
    -s, --sleep=S                    Sleep for S seconds after dumping (for debugging)  
    -c, --count=S                    Chunk size (default: 10000)  
    -f, --filter=S                   Filter selected keys (passed directly to redis' KEYS command)  
    -O, --without_optimizations      Disable run time optimizations  
    -V, --version                    Display version  
    -D, --debug  
        --nosafe
```
	注：- u 代表 Redis 连接字符串，-d 代表数据库代号，-s 代表导出之后的休眠时间，-c 代表分块大小，默认是 10000，
	-f 代表导出时的过滤器，-O 代表禁用运行时优化，-V 用于显示版本，-D 表示开启调试。

	
```python
	redis-dump -u :foobared@localhost:6379 # 无密码则为 redis-dump -u localhost:6379
	{"db":0,"key":"name2","ttl":-1,"type":"string","value":"Durant","size":6}  
	{"db":0,"key":"name3","ttl":-1,"type":"string","value":"Durant","size":6}  
	{"db":0,"key":"name4","ttl":-1,"type":"string","value":"HelloWorld","size":10}
	# 每条数据都包含 6 个字段，其中 db 即数据库代号，key 即键名，ttl 即该键值对的有效时间，type 即键值类型，
	# value 即内容，size 即占用空间。
	redis-dump -u :foobared@localhost:6379 &gt; ./redis_data.jl # 输出为 JSON 行文件
	redis-dump -u :foobared@localhost:6379 -d 1 &gt; ./redis.data.jl # 导出 1 号数据库的内容
	redis-dump -u :foobared@localhost:6379 -f adsl:* &gt; ./redis.data.jl 
	#  -f 即 Redis 的 keys 命令的参数,导出以 adsl 开头的数据，-f 参数用来过滤,可以写一些过滤规则
```
	再看redis-load，数据导入：
```python
	redis-load -h
	redis-load --help  
  	  Try: redis-load [global options] COMMAND [command options]   
	    -u, --uri=S                      Redis URI (e.g. redis://hostname[:port])  
	    -d, --database=S                 Redis database (e.g. -d 15)  
	    -s, --sleep=S                    Sleep for S seconds after dumping (for debugging)  
	    -n, --no_check_utf8  
	    -V, --version                    Display version  
	    -D, --debug  
	        --nosafe
```
	注： -u 代表 Redis 连接字符串，-d 代表数据库代号，默认是全部，-s 代表导出之后的休眠时间，
		-n 代表不检测 UTF-8 编码，-V 表示显示版本，-D 表示开启调试	
```python
	< redis_data.json redis-load -u :foobared@localhost:6379 #  JSON 行文件导入到 Redis 数据库
	cat redis_data.json | redis-load -u :foobared@localhost:6379 # JSON 行文件导入到 Redis 数据库

```
