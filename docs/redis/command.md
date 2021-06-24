# 操作符


```markdown
- command
  - str
    - set key value [expiration EX seconds|PX milliseconds] [NX|XX]
    - get key
    - getset key value
    - setnx key value
    - setex key seconds value
    - psetex key milliseconds value
    - mset key value [key value ...]
    - mget key [key ...]
    - msetnx key value [key value ...]
    - append key value
    - incr key
    - incrby key increment
    - incrbyfloat key increment
    - decr key 
    - decrby key decrement
    - strlen key
    - setrange key offset value
    - getrange key start end
  - hash
    - hset key field value
    - hsetnx key field value
    - hget key field
    - hlen key
    - hstrlen key field
    - hkeys key
    - hvals key
    - hgetall key
    - hincrby key field increment
    - hincrbyfloat key field increment
    - hmset key field value [field value ...]
    - hmget key field [field ...]
    - hscan key cursor [MATCH pattern] [COUNT count]
    - hexists key field
    - hdel key field [field ...]
  - list
    - lpush key value [value ...]
    - lpop key 
    - lpushx key value 
    - rpushx key value 
    - llen key
    - rpush key value [value ...]
    - rpop key
    - rpoplpush source destination
    - blpop key [key ...] timeout
    - brpop key [key ...] timeout
    - brpoplpush source destination timeout
    - lrange key start stop
    - ltrim key start stop
    - lindex ken index
    - linsert key BEFORE|AFTER pivot value
    - lrem key count value

  - sets 集合
    - sadd key member [member ...]
    - spop key [count]
    - smove source destination member
    - sismember key member
    - smembers key
    - sdiff key [key ...]
    - sdiffstore destination key [key ...]
    - sunion key [key ...]
    - sunionstore destination key [key ...]
    - sinter key [key ...]
    - sinterstore destination key [key ...]
    - srandmember key [count]
    - scard key
    - srem key member [member ...]

  - zsets 有序集合
  
```