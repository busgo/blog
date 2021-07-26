+++

title = "etcd 快速使用"
date = "2021-07-26"
description = "etcd 快速使用入门"
featured = false
categories = [
  "go"
]
tags = [
  "etcd"
]
images = [

"https://github.com/etcd-io/etcd/raw/main/logos/etcd-horizontal-color.svg"

]

+++

#### 概述

*etcd* 是 CoreOS 团队在2013年06月发起的开源项目，它的目标是构建一个高可用的分布式键值 *KV* 数据库。*etcd* 内部采用 *raft* 协议作为一致性协议算法，*etcd* 基于 *Go* 语言实现。*etcd* 对开发者使用上有以下特点。

* 简单：安装配置简单(二进制安装包)，提供了 HTTP API 进行交互，使用上非常简单。
* 安全：支持 *SSL* 证书验证。
* 快速：单台实例支持每秒 1w+读写操作。
* 可靠：基于 *CAP* 理论中的 *CP* 实现 *raft* 分布式一致性算法，实现分布式系统数据的一致性 *A* 和可用性 *P* 。

#### 应用场景

*etcd* 目前有很多应用场景是用于微服务注册/发现、分布式锁、选主以及限流工具。比较常用的一般我们在微服务中采用 *etcd* 介质来构建一个高性能、强一直性的微服务注册/发现中心。在同一个分布式系统中应用如何将服务信息注册能够通知到服务的订阅，服务上下线如何让当前的服务订阅者能够快速感知服务的状态。那么 *etcd* 是很好的一个 *CP* 选择,服务提供者通过 *etcd* 将自己的服务元数据注册在 *etcd* 中，服务订阅者将自己感兴趣的服务通过 *etcd* 发起订阅，当服务提供者上下线状态变更的时候则 *etcd* 能够进行通知到相关服务的订阅者进行感知，简单的称为 服务注册/发现流程。

#### 安装

安装 *etcd* 非常简单在  [下载页面](https://github.com/etcd-io/etcd/releases) 可以看到各个平台的二进制安装包以及 *docker* 安装方法，更多可以参考 [安装指南](http://play.etcd.io/)  和 [操作指南](https://etcd.io/docs/latest/op-guide) 。如果在生产环境进行使用 *etcd* 则推荐使用 *3-5* 个 *etcd* 节点为宜不需要太多节点，如果节点太多则数据同步复制会消耗 *etcd* 的性能；如果节点少于3台则会不会保证 *etcd* 集群的高可用性。因为在 *raft* 选举中超过半数以上的节点 ( *n/2+1*) 才能成为 *Leader* 否则整个集群将会一个 脑裂选举的状态。

为了方便演示使用 *docker* 进行单机节点进行部署方式，*docker* 拉取镜像

```shell
# https://gcr.io/etcd-development/etcd
docker pull gcr.io/etcd-development/etcd:v3.5.0
```

启动容器

```shell
docker run \
  -p 2379:2379 \
  -p 2380:2380 \
  --mount type=bind,source=/tmp/etcd-data.tmp,destination=/etcd-data \
  --name etcd-gcr-v3.5.0 \
  gcr.io/etcd-development/etcd:v3.5.0 \
  /usr/local/bin/etcd \
  --name s1 \
  --data-dir /etcd-data \
  --listen-client-urls http://0.0.0.0:2379 \
  --advertise-client-urls http://0.0.0.0:2379 \
  --listen-peer-urls http://0.0.0.0:2380 \
  --initial-advertise-peer-urls http://0.0.0.0:2380 \
  --initial-cluster s1=http://0.0.0.0:2380 
```

控制台输出

```shell
Digest: sha256:28759af54acd6924b2191dc1a1d096e2fa2e219717a21b9d8edf89717db3631b
Status: Downloaded newer image for gcr.io/etcd-development/etcd:v3.5.0
{"level":"info","ts":"2021-07-26T02:03:56.314Z","caller":"etcdmain/etcd.go:72","msg":"Running: ","args":["/usr/local/bin/etcd","--name","s1","--data-dir","/etcd-data","--listen-client-urls","http://0.0.0.0:2379","--advertise-client-urls","http://0.0.0.0:2379","--listen-peer-urls","http://0.0.0.0:2380","--initial-advertise-peer-urls","http://0.0.0.0:2380","--initial-cluster","s1=http://0.0.0.0:2380"]}
{"level":"info","ts":"2021-07-26T02:03:56.314Z","caller":"embed/etcd.go:131","msg":"configuring peer listeners","listen-peer-urls":["http://0.0.0.0:2380"]}
{"level":"info","ts":"2021-07-26T02:03:56.314Z","caller":"embed/etcd.go:139","msg":"configuring client listeners","listen-client-urls":["http://0.0.0.0:2379"]}
{"level":"info","ts":"2021-07-26T02:03:56.314Z","caller":"embed/etcd.go:307","msg":"starting an etcd server","etcd-version":"3.5.0","git-sha":"946a5a6f2","go-version":"go1.16.3","go-os":"linux","go-arch":"amd64","max-cpu-set":4,"max-cpu-available":4,"member-initialized":false,"name":"s1","data-dir":"/etcd-data","wal-dir":"","wal-dir-dedicated":"","member-dir":"/etcd-data/member","force-new-cluster":false,"heartbeat-interval":"100ms","election-timeout":"1s","initial-election-tick-advance":true,"snapshot-count":100000,"snapshot-catchup-entries":5000,"initial-advertise-peer-urls":["http://0.0.0.0:2380"],"listen-peer-urls":["http://0.0.0.0:2380"],"advertise-client-urls":["http://0.0.0.0:2379"],"listen-client-urls":["http://0.0.0.0:2379"],"listen-metrics-urls":[],"cors":["*"],"host-whitelist":["*"],"initial-cluster":"s1=http://0.0.0.0:2380","initial-cluster-state":"new","initial-cluster-token":"etcd-cluster","quota-size-bytes":2147483648,"pre-vote":true,"initial-corrupt-check":false,"corrupt-check-time-interval":"0s","auto-compaction-mode":"periodic","auto-compaction-retention":"0s","auto-compaction-interval":"0s","discovery-url":"","discovery-proxy":"","downgrade-check-interval":"5s"}
{"level":"info","ts":"2021-07-26T02:03:56.316Z","caller":"etcdserver/backend.go:81","msg":"opened backend db","path":"/etcd-data/member/snap/db","took":"1.070978ms"}
{"level":"info","ts":"2021-07-26T02:03:56.318Z","caller":"etcdserver/raft.go:448","msg":"starting local member","local-member-id":"1c70f9bbb41018f","cluster-id":"a0d2de0531db7884"}
{"level":"info","ts":"2021-07-26T02:03:56.318Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f switched to configuration voters=()"}
{"level":"info","ts":"2021-07-26T02:03:56.318Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f became follower at term 0"}
{"level":"info","ts":"2021-07-26T02:03:56.318Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"newRaft 1c70f9bbb41018f [peers: [], term: 0, commit: 0, applied: 0, lastindex: 0, lastterm: 0]"}
{"level":"info","ts":"2021-07-26T02:03:56.318Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f became follower at term 1"}
{"level":"info","ts":"2021-07-26T02:03:56.318Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f switched to configuration voters=(128088275939295631)"}
{"level":"warn","ts":"2021-07-26T02:03:56.320Z","caller":"auth/store.go:1220","msg":"simple token is not cryptographically signed"}
{"level":"info","ts":"2021-07-26T02:03:56.321Z","caller":"mvcc/kvstore.go:415","msg":"kvstore restored","current-rev":1}
{"level":"info","ts":"2021-07-26T02:03:56.322Z","caller":"etcdserver/quota.go:94","msg":"enabled backend quota with default value","quota-name":"v3-applier","quota-size-bytes":2147483648,"quota-size":"2.1 GB"}
{"level":"info","ts":"2021-07-26T02:03:56.322Z","caller":"etcdserver/server.go:843","msg":"starting etcd server","local-member-id":"1c70f9bbb41018f","local-server-version":"3.5.0","cluster-version":"to_be_decided"}
{"level":"info","ts":"2021-07-26T02:03:56.323Z","caller":"etcdserver/server.go:728","msg":"started as single-node; fast-forwarding election ticks","local-member-id":"1c70f9bbb41018f","forward-ticks":9,"forward-duration":"900ms","election-ticks":10,"election-timeout":"1s"}
{"level":"info","ts":"2021-07-26T02:03:56.324Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f switched to configuration voters=(128088275939295631)"}
{"level":"info","ts":"2021-07-26T02:03:56.324Z","caller":"membership/cluster.go:393","msg":"added member","cluster-id":"a0d2de0531db7884","local-member-id":"1c70f9bbb41018f","added-peer-id":"1c70f9bbb41018f","added-peer-peer-urls":["http://0.0.0.0:2380"]}
{"level":"info","ts":"2021-07-26T02:03:56.326Z","caller":"embed/etcd.go:276","msg":"now serving peer/client/metrics","local-member-id":"1c70f9bbb41018f","initial-advertise-peer-urls":["http://0.0.0.0:2380"],"listen-peer-urls":["http://0.0.0.0:2380"],"advertise-client-urls":["http://0.0.0.0:2379"],"listen-client-urls":["http://0.0.0.0:2379"],"listen-metrics-urls":[]}
{"level":"info","ts":"2021-07-26T02:03:56.326Z","caller":"embed/etcd.go:580","msg":"serving peer traffic","address":"[::]:2380"}
{"level":"info","ts":"2021-07-26T02:03:56.326Z","caller":"embed/etcd.go:552","msg":"cmux::serve","address":"[::]:2380"}
{"level":"info","ts":"2021-07-26T02:03:57.319Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f is starting a new election at term 1"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f became pre-candidate at term 1"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f received MsgPreVoteResp from 1c70f9bbb41018f at term 1"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f became candidate at term 2"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f received MsgVoteResp from 1c70f9bbb41018f at term 2"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f became leader at term 2"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"raft.node: 1c70f9bbb41018f elected leader 1c70f9bbb41018f at term 2"}
{"level":"info","ts":"2021-07-26T02:03:57.321Z","caller":"etcdserver/server.go:2476","msg":"setting up initial cluster version using v2 API","cluster-version":"3.5"}
{"level":"info","ts":"2021-07-26T02:03:57.323Z","caller":"membership/cluster.go:531","msg":"set initial cluster version","cluster-id":"a0d2de0531db7884","local-member-id":"1c70f9bbb41018f","cluster-version":"3.5"}
{"level":"info","ts":"2021-07-26T02:03:57.323Z","caller":"etcdserver/server.go:2027","msg":"published local member to cluster through raft","local-member-id":"1c70f9bbb41018f","local-member-attributes":"{Name:s1 ClientURLs:[http://0.0.0.0:2379]}","request-path":"/0/members/1c70f9bbb41018f/attributes","cluster-id":"a0d2de0531db7884","publish-timeout":"7s"}
{"level":"info","ts":"2021-07-26T02:03:57.323Z","caller":"embed/serve.go:98","msg":"ready to serve client requests"}
{"level":"info","ts":"2021-07-26T02:03:57.323Z","caller":"api/capability.go:75","msg":"enabled capabilities for version","cluster-version":"3.5"}
{"level":"info","ts":"2021-07-26T02:03:57.323Z","caller":"etcdserver/server.go:2500","msg":"cluster version is updated","cluster-version":"3.5"}
{"level":"info","ts":"2021-07-26T02:03:57.323Z","caller":"etcdmain/main.go:47","msg":"notifying init daemon"}
{"level":"info","ts":"2021-07-26T02:03:57.324Z","caller":"etcdmain/main.go:53","msg":"successfully notified init daemon"}
{"level":"info","ts":"2021-07-26T02:03:57.324Z","caller":"embed/serve.go:140","msg":"serving client traffic insecurely; this is strongly discouraged!","address":"[::]:2379"}
```

从控制台输出日志我们可以看出一些关键信息

* name  节点名称
* data-dir 数据保存目录
* http://localhost:2378 与其他 *peer* 节点进行通信地址。
* http://localhost:2379 与客户端进行 HTTP API 交互的通信地址。
* heartbeat-interval 心跳间隔 用于 *leader* 向 *follower* 发起一次心跳时间间隔。
* election-timeout 选举超时时间  如果 *follower* 在1秒内没有收到 *leader* 的心跳 则会发起重新选举请求到其他的 *peer* 节点。
* snapshot-count 多少事务被提交则会触发截取快照到磁盘。
* 选举过程

```shell
{"level":"info","ts":"2021-07-26T02:03:57.319Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f is starting a new election at term 1"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f became pre-candidate at term 1"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f received MsgPreVoteResp from 1c70f9bbb41018f at term 1"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f became candidate at term 2"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f received MsgVoteResp from 1c70f9bbb41018f at term 2"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"1c70f9bbb41018f became leader at term 2"}
{"level":"info","ts":"2021-07-26T02:03:57.320Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"raft.node: 1c70f9bbb41018f elected leader 1c70f9bbb41018f at term 2"}
```



* 在第一任期号创建预投票选举请求。
* 1c70f9bbb41018f 晋升为预候选人。
* 1c70f9bbb41018f 接受预投票响应。
* 1c70f9bbb41018f 在任期号为2 的时候升级为候选人身份并发起投票选举请求。
* 1c70f9bbb41018f 接受投票选举响应。
* 超过半数以上投票 1c70f9bbb41018f 在第二任期晋升为 *leader* 。



#### etcd  Go 客户端基本使用

官方自带的 [Go 客户端](https://github.com/etcd-io/etcd/tree/main/client/v3) 提供给开发者直接跟 *etcd* 打交道，操作上非常简洁、方便。

安装 *etcd* 依赖包

```shell
go get go.etcd.io/etcd/client/v3
```

创建并初始化 *etcd* 客户端,使用 *clientv3.New*  方法进行创建一个 *etcd* 客户端对象

```shell
// new etcd cli
func NewEtcdCli(config *CliConfig) (*Cli, error) {

	c, err := clientv3.New(clientv3.Config{
		Endpoints:   config.Endpoints,
		Username:    config.UserName,
		Password:    config.Password,
		DialTimeout: config.DialTimeout,
	})

	if err != nil {
		return nil, err
	}

	return &Cli{
		c:         c,
		kv:        clientv3.NewKV(c),
		lease:     clientv3.NewLease(c),
		elections: make(map[string]*concurrency.Election),
	}, err
}

```

由于 etcd v3 版使用 *gRPC*  方式进行通信，在 *etcd* 操作的时候要指定客户端 *context.WithTimeout* 请求超时时间。

*Get* 通过 *Get* 方法可以获取指定key 或者指定的key值前缀的kv集合信息。

```go

// get with key 
func (cli *Cli)Get(ctx context.Context,key string) (string,error) {

	resp,err:= cli.kv.Get(ctx,key)
	if err != nil {
		return "", err
	}

	if len(resp.Kvs) ==0{
		return "", nil
	}
	return string(resp.Kvs[0].Value), err

}


// get with prefix 
func (cli *Cli)GetWithPrefix(ctx context.Context,prefix string) ([]string ,[]string,error) {

	resp,err:= cli.kv.Get(ctx,prefix,clientv3.WithPrefix())
	if err != nil {
		return make([]string,0),make([]string,0), err
	}

	if len(resp.Kvs) ==0{
		return make([]string,0),make([]string,0), nil
	}

	keys:= make([]string,0)
	values:=make([]string,0)
	for _,kv:=range resp.Kvs{

		keys = append(keys,string(kv.Key))
		values = append(values,string(kv.Value))
	}

	return keys, values, err
}
```





*PUT* 

在 *etcd* 中设置kv ,如果有错误则直接返回相应的异常即可。

```go
// put a key
func (cli *Cli) Put(ctx context.Context, key, value string) error {

	_, err := cli.kv.Put(ctx, key, value)
	return err
}
```



PUT withTTL 

类似 *zookeeper* 中的临时节点 在 *etcd* 中没有临时节点的概念，我们只能通过 *etcd* 的 *租约* 来实现临时节点的功能。

```go
// put a key with ttl
func (cli *Cli) PutWithTTL(ctx context.Context, key, value string, ttl int64) (int64, error) {

	leaseResponse, err := cli.lease.Grant(ctx, ttl)
	if err != nil {
		return 0, err
	}
	_, err = cli.kv.Put(ctx, key, value, clientv3.WithLease(leaseResponse.ID))
	return int64(leaseResponse.ID), err
}
```

注意 TTL 单位为秒数



PUT withNotExist

*etcd* 支持事务方式我们可以不存在则设置，存在则直接返回

```go
func (cli *Cli) PutWithNotExist(ctx context.Context, key, value string) error {
	_, err := cli.c.Txn(ctx).If(clientv3.Compare(clientv3.Value(key), "=", "")).
		Then(clientv3.OpPut(key, value)).
		Commit()
	return err
}
```

PUT withNotExistTTL

*etcd* 支持事务方式我们可以不存在则设置，存在则直接返回

```go
func (cli *Cli) PutWithNotExistTTL(ctx context.Context, key, value string, ttl int64) (int64, error) {
	leaseResponse, err := cli.lease.Grant(ctx, ttl)
	if err != nil {
		return 0, err
	}
	_, err = cli.c.Txn(ctx).If(clientv3.Compare(clientv3.Value(key), "=", "")).
		Then(clientv3.OpPut(key, value, clientv3.WithLease(leaseResponse.ID))).
		Commit()
	return int64(leaseResponse.ID), nil
}
```

Revoke 给定的租约id 进行废除

```go
func (cli *Cli) Revoke(ctx context.Context, leaseId int64) error {

	if leaseId <= 0 {
		return nil
	}
	_, err := cli.lease.Revoke(ctx, clientv3.LeaseID(leaseId))
	return err
}
```

Keepalive etcd 指定key 进行定期自动续租操作,注意定期续租默认 *channel* 会存储16个 我们需要进行 起一个 *goroutine* 进行消费即可。

```go
func (cli *Cli) Keepalive(ctx context.Context, key, value string, ttl int64) error {
	resp, err := cli.lease.Grant(ctx, ttl)
	if err != nil {
		return err
	}
	_, err = cli.kv.Put(ctx, key, value, clientv3.WithLease(resp.ID))
	if err != nil {
		return err
	}

	// the key 'foo' will be kept forever
	ch, err := cli.lease.KeepAlive(context.Background(), resp.ID)
	if err != nil {
		return err
	}
	go keepaliveHandle(key, ch)
	return nil
}


// handle keep alive
func keepaliveHandle(key string, ch <-chan *clientv3.LeaseKeepAliveResponse) {

	for {
		select {
		case c := <-ch:
			if c == nil {
				log.Printf("the keep alive key:%s has closed", key)
				return
			}
			log.Printf("keep alive for key:%s .................%+v", key, c)

		}
	}
}
```

Watch 监听指定的key 或者指定的 key 前缀变化事件，我们可以通过自定义事件+ channel 方式通知到订阅者。

```go

func (cli *Cli) Watch(key string) (*WatchKeyResponse, error) {

	watcher := clientv3.NewWatcher(cli.c)
	watchChan := watcher.Watch(context.Background(), key)
	keyChangeCh := make(chan *KeyChange, defaultKeyChangeSize)

	// start watch
	go keyChangeHandle(key, watchChan, keyChangeCh)
	return &WatchKeyResponse{
		Watcher:     watcher,
		KeyChangeCh: keyChangeCh,
	}, nil

}



func (cli *Cli) WatchWithPrefix(prefix string) (*WatchKeyResponse, error) {

	watcher := clientv3.NewWatcher(cli.c)
	watchChan := watcher.Watch(context.Background(), prefix, clientv3.WithPrefix())

	keyChangeCh := make(chan *KeyChange, defaultKeyChangeSize)

	// start watch
	go keyChangeHandle(prefix, watchChan, keyChangeCh)
	return &WatchKeyResponse{
		Watcher:     watcher,
		KeyChangeCh: keyChangeCh,
	}, nil

}



func keyChangeHandle(prefix string, watchChan clientv3.WatchChan, keyChangeCh chan *KeyChange) {

	for {
		select {
		case ch, ok := <-watchChan:
			if !ok {
				log.Printf("the watch prefix key:%s has cancel", prefix)
				keyChangeCh <- &KeyChange{
					Event: KeyCancelChangeEvent,
					Key:   prefix,
				}
				return
			}
			for _, event := range ch.Events {
				keyChangeEventHandle(event, keyChangeCh)
			}
		}

	}
}

func keyChangeEventHandle(event *clientv3.Event, ch chan *KeyChange) {

	c := &KeyChange{
		Key:   string(event.Kv.Key),
		Value: "",
	}
	switch event.Type {
	case mvccpb.PUT:
		c.Value = string(event.Kv.Value)
		c.Event = KeyCreateChangeEvent
		if event.IsModify() {
			c.Event = KeyUpdateChangeEvent
		}
	case mvccpb.DELETE:
		c.Event = KeyDeleteChangeEvent
	}
	ch <- c
}
```

Compaign 选举

```go
// campaign become leader
func (cli *Cli) Campaign(ctx context.Context, id, prefix string, ttl int64) error {

	// create a session
	session, err := concurrency.NewSession(cli.c, concurrency.WithTTL(int(ttl)))
	if err != nil {
		log.Printf("new session fail,id:%s,prefix:%s,%+v", id, prefix, err)
		return err
	}

	election := concurrency.NewElection(session, prefix)
	cli.elections[prefix] = election
	return election.Campaign(ctx, id)
}
```

Leader 查找当前 *Leader* 信息

```go

func (cli *Cli) getElection(prefix string) (*concurrency.Election, error) {

	election := cli.elections[prefix]
	if election != nil {
		return election, nil
	}
	// create a session
	session, err := concurrency.NewSession(cli.c)
	if err != nil {
		log.Printf("new session fail,prefix:%s,%+v", prefix, err)
		return nil, err
	}
	election = concurrency.NewElection(session, prefix)
	cli.elections[prefix] = election
	return election, nil
}

// find leader
func (cli *Cli) Leader(ctx context.Context, prefix string) (id string, err error) {

	election, err := cli.getElection(prefix)
	if err != nil {
		return
	}

	resp, err := election.Leader(ctx)
	if err != nil {
		return
	}
	return string(resp.Kvs[0].Value), nil

}
```

完整 *etcd* 客户端操作 *Go* 代码

```go
package etcd

import (
	"context"
	"go.etcd.io/etcd/api/v3/mvccpb"
	clientv3 "go.etcd.io/etcd/client/v3"
	"go.etcd.io/etcd/client/v3/concurrency"
	"log"
	"sync"
	"time"
)

type KeyChangeEvent int32

type KeyChangeChan <-chan *KeyChange

const (
	KeyCreateChangeEvent = iota + 1 //  create event
	KeyUpdateChangeEvent            //  update event
	KeyDeleteChangeEvent            //  delete event
	KeyCancelChangeEvent            //  cancel event
	defaultKeyChangeSize = 32
)

//  etcd cli
type Cli struct {
	c         *clientv3.Client
	kv        clientv3.KV
	lease     clientv3.Lease
	elections map[string]*concurrency.Election
	sync.RWMutex
}

// etcd cli config
type CliConfig struct {
	Endpoints   []string
	UserName    string
	Password    string
	DialTimeout time.Duration
}

type WatchKeyResponse struct {
	Watcher     clientv3.Watcher
	KeyChangeCh chan *KeyChange
}

type KeyChange struct {
	Event KeyChangeEvent
	Key   string
	Value string
}

// new etcd cli
func NewEtcdCli(config *CliConfig) (*Cli, error) {

	c, err := clientv3.New(clientv3.Config{
		Endpoints:   config.Endpoints,
		Username:    config.UserName,
		Password:    config.Password,
		DialTimeout: config.DialTimeout,
	})

	if err != nil {
		return nil, err
	}

	return &Cli{
		c:         c,
		kv:        clientv3.NewKV(c),
		lease:     clientv3.NewLease(c),
		elections: make(map[string]*concurrency.Election),
	}, err
}


// get with key
func (cli *Cli)Get(ctx context.Context,key string) (string,error) {

	resp,err:= cli.kv.Get(ctx,key)
	if err != nil {
		return "", err
	}

	if len(resp.Kvs) ==0{
		return "", nil
	}
	return string(resp.Kvs[0].Value), err

}


// get with prefix
func (cli *Cli)GetWithPrefix(ctx context.Context,prefix string) ([]string ,[]string,error) {

	resp,err:= cli.kv.Get(ctx,prefix,clientv3.WithPrefix())
	if err != nil {
		return make([]string,0),make([]string,0), err
	}

	if len(resp.Kvs) ==0{
		return make([]string,0),make([]string,0), nil
	}

	keys:= make([]string,0)
	values:=make([]string,0)
	for _,kv:=range resp.Kvs{

		keys = append(keys,string(kv.Key))
		values = append(values,string(kv.Value))
	}

	return keys, values, err
}


// put a key
func (cli *Cli) Put(ctx context.Context, key, value string) error {

	_, err := cli.kv.Put(ctx, key, value)
	return err
}

// put a key with ttl
func (cli *Cli) PutWithTTL(ctx context.Context, key, value string, ttl int64) (int64, error) {

	leaseResponse, err := cli.lease.Grant(ctx, ttl)
	if err != nil {
		return 0, err
	}
	_, err = cli.kv.Put(ctx, key, value, clientv3.WithLease(leaseResponse.ID))
	return int64(leaseResponse.ID), err
}

func (cli *Cli) PutWithNotExist(ctx context.Context, key, value string) error {
	_, err := cli.c.Txn(ctx).If(clientv3.Compare(clientv3.Value(key), "=", "")).
		Then(clientv3.OpPut(key, value)).
		Commit()
	return err
}

func (cli *Cli) PutWithNotExistTTL(ctx context.Context, key, value string, ttl int64) (int64, error) {
	leaseResponse, err := cli.lease.Grant(ctx, ttl)
	if err != nil {
		return 0, err
	}
	_, err = cli.c.Txn(ctx).If(clientv3.Compare(clientv3.Value(key), "=", "")).
		Then(clientv3.OpPut(key, value, clientv3.WithLease(leaseResponse.ID))).
		Commit()
	return int64(leaseResponse.ID), nil
}

func (cli *Cli) Revoke(ctx context.Context, leaseId int64) error {

	if leaseId <= 0 {
		return nil
	}
	_, err := cli.lease.Revoke(ctx, clientv3.LeaseID(leaseId))
	return err
}

func (cli *Cli) Keepalive(ctx context.Context, key, value string, ttl int64) error {
	resp, err := cli.lease.Grant(ctx, ttl)
	if err != nil {
		return err
	}
	_, err = cli.kv.Put(ctx, key, value, clientv3.WithLease(resp.ID))
	if err != nil {
		return err
	}

	// the key 'foo' will be kept forever
	ch, err := cli.lease.KeepAlive(context.Background(), resp.ID)
	if err != nil {
		return err
	}
	go keepaliveHandle(key, ch)
	return nil
}

// handle keep alive
func keepaliveHandle(key string, ch <-chan *clientv3.LeaseKeepAliveResponse) {

	for {
		select {
		case c := <-ch:
			if c == nil {
				log.Printf("the keep alive key:%s has closed", key)
				return
			}
			log.Printf("keep alive for key:%s .................%+v", key, c)

		}
	}
}

func (cli *Cli) Watch(key string) (*WatchKeyResponse, error) {

	watcher := clientv3.NewWatcher(cli.c)
	watchChan := watcher.Watch(context.Background(), key)
	keyChangeCh := make(chan *KeyChange, defaultKeyChangeSize)

	// start watch
	go keyChangeHandle(key, watchChan, keyChangeCh)
	return &WatchKeyResponse{
		Watcher:     watcher,
		KeyChangeCh: keyChangeCh,
	}, nil

}

func (cli *Cli) WatchWithPrefix(prefix string) (*WatchKeyResponse, error) {

	watcher := clientv3.NewWatcher(cli.c)
	watchChan := watcher.Watch(context.Background(), prefix, clientv3.WithPrefix())

	keyChangeCh := make(chan *KeyChange, defaultKeyChangeSize)

	// start watch
	go keyChangeHandle(prefix, watchChan, keyChangeCh)
	return &WatchKeyResponse{
		Watcher:     watcher,
		KeyChangeCh: keyChangeCh,
	}, nil

}

func keyChangeHandle(prefix string, watchChan clientv3.WatchChan, keyChangeCh chan *KeyChange) {

	for {
		select {
		case ch, ok := <-watchChan:
			if !ok {
				log.Printf("the watch prefix key:%s has cancel", prefix)
				keyChangeCh <- &KeyChange{
					Event: KeyCancelChangeEvent,
					Key:   prefix,
				}
				return
			}
			for _, event := range ch.Events {
				keyChangeEventHandle(event, keyChangeCh)
			}
		}

	}
}

func keyChangeEventHandle(event *clientv3.Event, ch chan *KeyChange) {

	c := &KeyChange{
		Key:   string(event.Kv.Key),
		Value: "",
	}
	switch event.Type {
	case mvccpb.PUT:
		c.Value = string(event.Kv.Value)
		c.Event = KeyCreateChangeEvent
		if event.IsModify() {
			c.Event = KeyUpdateChangeEvent
		}
	case mvccpb.DELETE:
		c.Event = KeyDeleteChangeEvent
	}
	ch <- c
}

// campaign become leader
func (cli *Cli) Campaign(ctx context.Context, id, prefix string, ttl int64) error {

	// create a session
	session, err := concurrency.NewSession(cli.c, concurrency.WithTTL(int(ttl)))
	if err != nil {
		log.Printf("new session fail,id:%s,prefix:%s,%+v", id, prefix, err)
		return err
	}

	election := concurrency.NewElection(session, prefix)
	cli.elections[prefix] = election
	return election.Campaign(ctx, id)
}

func (cli *Cli) getElection(prefix string) (*concurrency.Election, error) {

	election := cli.elections[prefix]
	if election != nil {
		return election, nil
	}
	// create a session
	session, err := concurrency.NewSession(cli.c)
	if err != nil {
		log.Printf("new session fail,prefix:%s,%+v", prefix, err)
		return nil, err
	}
	election = concurrency.NewElection(session, prefix)
	cli.elections[prefix] = election
	return election, nil
}

// find leader
func (cli *Cli) Leader(ctx context.Context, prefix string) (id string, err error) {

	election, err := cli.getElection(prefix)
	if err != nil {
		return
	}

	resp, err := election.Leader(ctx)
	if err != nil {
		return
	}
	return string(resp.Kvs[0].Value), nil

}
```









