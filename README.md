# chefzero-benchmark

[Chef Zero](https://github.com/opscode/chef-zero)

```
$ rake -vT
rake bench:disk  # disk_store benchmack
rake bench:mem   # memory_store benchmack
rake default     # run benchmark both mem_store and disk_store
```

This benchmark runs below.

1. Creates chef zero server with file store which is set memory or local file system.
2. Register fake nodes which is made by [fauxhai](https://github.com/customink/fauxhai).
3. Pick up one node.
4. Search node with simple query.
5. Search node with wildcard query.


## Result example

Nodes: `[50, 100, 500, 1000, 5000]`.

```
MacBook Pro Retina, 13-inch, Late 2013
Processor: 2.8 GHz Intel Core i7
Memory: 16 GB 1600 MHz DDR3

------

```

```