# Local Redis Cluster with Docker Compose

Sets up a local Redis v6 cluster with 3 master nodes and no replica.

To add a new replica, just modify the `docker-compose.yml` file and add some more redis instance with different `ipv4_address` subnet. Then, add that new subnet into the `command` field on `redis-cluster-init`, and add the service on the `depends_on` field so the initialization service will only execute if everything's running.

To run it:
```sh
docker compose up -d
```

There is no user. The password is `foobared`. Change the password to anything in `redis.conf` file.

Don't use this configuration in production.

To connect with Go:
```go
package main

import (
    "context"
    "time"

	"github.com/go-redis/redis/v8"
)

func main() {
    cache := redis.NewClusterClient(&redis.ClusterOptions{
		Addrs:        []string{"localhost:6379", "localhost:6380", "localhost:6381"},
		ReadOnly:     true,
		Username:     "",
		Password:     "foobared",
		DialTimeout:  time.Second * 30,
		WriteTimeout: time.Second * 5,
		ReadTimeout:  time.Second * 5,
		IdleTimeout:  time.Second * 30,
	})
    defer cache.Close()

    ctx, cancel := context.WithTimeout(context.Background(), time.Second * 60)
    defer cancel()

    err := cache.SetEX(ctx, "hello", "world").Err()
    if err != nil {
        log.Printf(err)
        return
    }

    res, err := cache.Get(ctx, "hello").Result()
    if err != nil {
        log.Printf(err)
        return
    }

    log.Println(res)
}
```