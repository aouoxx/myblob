

### kafka 中关注点信息

#### _kafka在zk中的节点_

```xml
consumers
    |--console-consumer-35193(consumer-group)
        |--ids(consumerId)
            |--console-consumer-35193_ssgao-1531791473956-90d1d19f (临时节点,每创建一个Consumer实例就会在ids目录下创建一个consumerId节点)
        |--offsets
            |--topic-0
                |-0 (partition 索引号:持久znode,partition编号,编号0分区,被该消费者消费消息的offset偏移量,该offset含义为消费了多少条消息或记录)
                |-n (partition 索引号)
            |--topic-n
        |--owners
            |--topic-0
                |--0 (partition 索引号:临时节点,用来标识partition被哪个consumer(线程)消费,当有consumer加入时创建,存储内容为线程标识)
                |--n                                
*) consumerId节点的内容
   {"version":1,"subscription":{"ssgao":1},"pattern":"white_list","timestamp":"1531791474058"}
```



### _kafka再平衡_

```xml

```

