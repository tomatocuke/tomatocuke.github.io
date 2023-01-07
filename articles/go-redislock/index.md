---
title: "go+redis分布式锁"
date: "2022-11-23"
---

1. Redis怎么做分布式锁是老生常谈，但是之前没有注意一些细节，存在小隐患。
	- 可重入。必须使用脚本原子性执行，先通过key获取，如果没有正常加锁`（set + ex + nx)`，如果存在，则刷新`ttl`。
	- 误删问题。超时是不可预知的，试想，当A获取到锁，进行业务逻辑，再解锁，但是业务逻辑超时了，锁已经自动解除，并且被B获得，那么A解锁就会把B的锁删掉。   所以，需要一个id来保证只解除自己的锁。
2. 使用Go来实现。
   - 重试机制。网上能找到的例子多是不带重试机制的，即尝试加锁，失败直接返回。
   - 超时控制。用go写第一个项目的时候，我在重试里使用for循环，给timeout参数。但是其实有个问题的，A加锁后处理业务时出现了不可控的问题，之后的BCD..仍然会重新加锁尝试，这大概是不必要的。 所以这版使用了`context.Context`来进行协程控制，A出现问题，直接调用`cancel()` 去掉不必要的重试。


<!--more-->

### 代码
```go
package rds

import (
	"context"
	"errors"
	"math/rand"
	"strconv"
	"time"

	"github.com/go-redis/redis"
)

const (

	// 先get获取，如果有就刷新ttl，没有再set。 这种是可重入锁，防止在同一线程中多次获取锁而导致死锁发生。
	lockCommand = `if redis.call("GET", KEYS[1]) == ARGV[1] then
	redis.call("SET", KEYS[1], ARGV[1], "PX", ARGV[2])
	return "OK"
else
	return redis.call("SET", KEYS[1], ARGV[1], "NX", "PX", ARGV[2])
end`

	// 删除。必须先匹配id值，防止A超时后，B马上获取到锁，A的解锁把B的锁删了
	delCommand = `if redis.call("GET", KEYS[1]) == ARGV[1] then
	return redis.call("DEL", KEYS[1])
else
	return 0
end`

	letters   = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
	randomLen = 10
)

var (
	redisDb *redis.Client
	// 默认超时时间
	defaultTimeout = 500 * time.Millisecond
	// 重试间隔
	retryInterval = 10 * time.Millisecond
	// 上下文取消
	ErrContextCancel = errors.New("context cancel")
)

func init() {
	rand.Seed(time.Now().UnixNano())
	if err := initClient(); err != nil {
		panic(err)
	}
}

// 根据redis配置初始化一个客户端
func initClient() (err error) {
	redisDb = redis.NewClient(&redis.Options{
		Addr:     "localhost:6379", // redis地址
		Password: "",               // redis密码，没有则留空
		DB:       0,                // 默认数据库，默认是0
	})

	_, err = redisDb.Ping().Result()
	return err
}

type RedisLock struct {
	ctx       context.Context
	timeoutMs int
	key       string
	id        string
}

func NewRedisLock(ctx context.Context, key string) *RedisLock {
	timeout := defaultTimeout
	if deadline, ok := ctx.Deadline(); ok {
		timeout = deadline.Sub(time.Now())
	}
	rl := &RedisLock{
		ctx:       ctx,
		timeoutMs: int(timeout.Milliseconds()),
		key:       key,
		id:        randomStr(randomLen),
	}

	return rl
}

func (rl *RedisLock) TryLock() (bool, error) {
	t := strconv.Itoa(rl.timeoutMs)
	resp, err := redisDb.Eval(lockCommand, []string{rl.key}, []string{rl.id, t}).Result()
	if err != nil || resp == nil {
		return false, nil
	}

	reply, ok := resp.(string)
	return ok && reply == "OK", nil
}

func (rl *RedisLock) Lock() error {
	for {
		select {
		case <-rl.ctx.Done():
			return ErrContextCancel
		default:
			b, err := rl.TryLock()
			if err != nil {
				return err
			}
			if b {
				return nil
			}
			time.Sleep(retryInterval)
		}
	}
}

func (rl *RedisLock) Unlock() {
	redisDb.Eval(delCommand, []string{rl.key}, []string{rl.id}).Result()
}

func randomStr(n int) string {
	b := make([]byte, n)
	for i := range b {
		b[i] = letters[rand.Intn(len(letters))]
	}
	return string(b)
}

```


### 测试

```go
package rds

import (
	"context"
	"fmt"
	"sync"
	"sync/atomic"
	"testing"
	"time"
)

// 测试
func BenchmarkRedisLock1(b *testing.B) {
	x := b.N
	wg := sync.WaitGroup{}
	wg.Add(x)
	now := time.Now()

	var n int64
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)
	defer cancel()
	for i := 0; i < x; i++ {
		go func() {
			defer wg.Done()
			rl := NewRedisLock(ctx, "testLock")
			if err := rl.Lock(); err != nil {
				cancel() // 这里cancel，可以在第一个超时或者发生错误后，后边的不再尝试加锁。
				return
			}
			defer rl.Unlock()
			atomic.AddInt64(&n, 1)
		}()
	}
	wg.Wait()
	fmt.Printf("测试数:%d\t成功数:%d\t结果:%t\t耗时： %d \n", x, n, x == int(n), time.Since(now).Milliseconds())
}
```
