# RedisRaft
### Strongly-Consistent Redis Deployments

RedisRaft is a Redis module that implements the [Raft Consensus
Algorithm](https://raft.github.io/), making it possible to create strongly-consistent clusters of Redis servers.

The Raft algorithm is provided by a [standalone Raft
library](https://github.com/willemt/raft) by Willem-Hendrik Thiart.

## Main Features

* Strong consistency (in the language of [CAP](https://en.wikipedia.org/wiki/CAP_theorem), this system prioritizes consistency and partition-tolerance).
* Support for most Redis data types and commands
* Dynamic cluster configuration (adding / removing nodes)
* Snapshots for log compaction
* Configurable quorum or fast reads

## Getting Started

### Building

The module is mostly self-contained and comes with its dependencies as git
submodules under `deps`.

To compile the module, you will need:
* Build essentials (a compiler, GNU make, etc.)
* CMake
* GNU autotools (autoconf, automake, libtool)
* libbsd-dev (on Debian/Ubuntu) or an equivalent for `bsd/sys/queue.h`.

To build, simply run:

    git submodule init
    git submodule update
    make

### Creating a RedisRaft Cluster

Note: Make sure you're using a recent Redis 6.0 release candidate or a private build from the `unstable` branch. RedisRaft depends on Module API capabilities not available versions of Redis < 6.0.

To create a three node cluster, start the first node and initialize the
cluster:

    redis-server \
        --port 5001 --dbfilename raft1.rdb \
        --loadmodule <path-to>/redisraft.so \
            raft-log-filename=raftlog1.db addr=localhost:5001
    redis-cli -p 5001 raft.cluster init

Then start the second node, and run the `RAFT.CLUSTER JOIN` command to join it to the cluster:

    redis-server \
        --port 5002 --dbfilename raft2.rdb \
        --loadmodule <path-to>/redisraft.so \
            raft-log-filename=raftlog2.db addr=localhost:5002
    redis-cli -p 5002 RAFT.CLUSTER JOIN localhost:5001

Now add the third node in the same way:

    redis-server \
        --port 5003 --dbfilename raft3.rdb \
        --loadmodule <path-to>/redisraft.so \
            raft-log-filename=raftlog3.db addr=localhost:5003
    redis-cli -p 5003 RAFT.CLUSTER JOIN localhost:5001

To query the cluster state, run the `RAFT.INFO` command:

    redis-cli --raw -p 5001 RAFT.INFO

Now you can start using this RedisRaft cluster. All [supported Redis commands](docs/Using.md) will be executed in a strongly-consistent manner using the Raft protocol.

## Documentation

Please consult the [documentation](docs/TOC.md) for more information.

## License

RedisRaft is dual licensed under the [GNU Affero General Public License (AGPL) Version 3](LICENSE.agpl) or the [Redis Source Available License (RSAL)](LICENSE.rsal).
