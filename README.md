# `sniper`

[GoDoc](https://godoc.org/github.com/recoilme/sniper)

A simple and efficient thread-safe key/value store for Go.

> NOTES: 
> Original sniper repositroy not support Bucket (sortedset) reindex. If database closed, bucket indexes pufff.
> This repository supports:
> - Bucket reindexing after database opened.
> - Add new Index methods: Remove, PutIndex, RemoveIndex, HasIndex -> Look at the tests file how to use it


> TODO: Bucket Items count method is missing!!!

# Getting Started

## Features

* Store hundreds of millions of entries
* Fast. High concurrent. Thread-safe. Scales on multi-core CPUs
* Extremly low memory usage
* Zero GC overhead
* Simple, pure Go implementation

## Installing

To start using `sniper`, install Go and run `go get`:

```sh
$ go get -u github.com/uretgec/sniper
```

This will retrieve the library.

## Usage

The `Sniper` includes this methods:
`Set`, `Get`, `Incr`, `Decr`, `Delete`, `Count`, `Open`, `Close`, `FileSize`, `Backup`.

```go
s, _ := sniper.Open(sniper.Dir("1"))
s.Set([]byte("hello"), []byte("go"))
res, _ = s.Get([]byte("hello"))
fmt.Println(res)
s.Close()
// Output:
// go
```

with Bucket includes this methods: (this methods can use only this repository)
`CreateBucketIndexIfNotExists`,`Put`,`Remove`,`Keys`,`PutIndex`,`RemoveIndex`,`HasIndex`.

```go
s, _ := sniper.Open(sniper.Dir("2"),CreateBucketIndexIfNotExists([]string{"users"}))

users, _ := s.Bucket("users")
s.Set([]byte("users01")), []byte("rob"), 0) // Set users01 -> rob
s.PutIndex(users, []byte("01")) // users01 put into users bucket index

keys := users.Keys(0, 0)
fmt.Printf("%v#\n", keys) // []string{"01"}

has := s.HasIndex(users, []byte("01"))
fmt.Println(has) // true

deleted := s.RemoveIndex(users, []byte("02"))
fmt.Println(deleted) // false

res, _ = s.Get([]byte("users01"))
fmt.Println(res)

s.Close()
// Output:
// go
```

## Performance

```
MacBook Pro 2019 (Quad-Core Intel Core i7 2,8 GHz, 16 ГБ, APPLE SSD AP0512M)

go version go1.14 darwin/amd64

     number of cpus: 8
     number of keys: 10000000
            keysize: 10
        random seed: 1570109110136449000

-- sniper --
set: 10,000,000 ops over 8 threads in 63159ms, 158,331/sec, 6315 ns/op, 644.3 MB, 67 bytes/op
get: 10,000,000 ops over 8 threads in 4455ms, 2,244,629/sec, 445 ns/op, 305.5 MB, 32 bytes/op
del: 10,000,000 ops over 8 threads in 37568ms, 266,182/sec, 3756 ns/op, 122.8 MB, 12 bytes/op

With fsync

set: 10,000,000 ops over 8 threads in 85088ms, 117,524/sec, 8508 ns/op, 644.4 MB, 67 bytes/op
get: 10,000,000 ops over 8 threads in 5623ms, 1,778,268/sec, 562 ns/op, 305.5 MB, 32 bytes/op

```

## How it is done

* Sniper database is sharded on 250+ chunks. Each chunk has its own lock (RW), so it supports high concurrent access on multi-core CPUs.
* Each chunk store `hash(key) -> (value addr, value size)`, map. 
* Hash is very short, and has collisions. Sniper has collisions resolver.
* Efficient space reuse alghorithm. Every packet has power of 2 size, for inplace rewrite on value update and map of deleted entrys, for reusing space.

## Limitations

* 512 Kb - maximum entry size `len(key) + len(value)`
* ~1 Tb - maximum database size

## Mac OS tip

[How to Change Open Files Limit on OS X and macOS](https://gist.github.com/tombigel/d503800a282fcadbee14b537735d202c)

## Contact

Vadim Kulibaba [@recoilme](https://github.com/recoilme)

## License

`sniper` source code is available under the MIT [License](/LICENSE).