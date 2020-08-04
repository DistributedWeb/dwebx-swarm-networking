# dwebx-swarm-networking
[![Build Status](https://travis-ci.com/distributedweb/dwebx-swarm-networking.svg?branch=master)](https://travis-ci.com/distributedweb/dwebx-swarm-networking)

A dwebx networking module that uses [dwswarm](https://github.com/dwswarm/network) to discovery peers. This module powers the networking portion of the [Hyperdrive daemon](https://github.com/distributedweb/dwebfs-daemon).

Calls to `seed` or `unseed` will not be persisted across restarts, so you'll need to use a separate database that maps discovery keys to network configurations. The Hyperdrive daemon uses [Level](https://github.com/level/level) for this.

Since dwebx has an all-to-all replication model (any shared cores between two peers will be automatically replicated), only one connection needs to be maintained per peer. If multiple connections are opened to a single peer as a result of that peer announcing many keys, then these connections will be automatically deduplicated by comparing NOISE keypairs.

### Installation
```
npm i dwebx-swarm-networking -g
```

### Usage
```js
const SwarmNetworker = require('dwebx-swarm-networking')
const Corestore = require('dwebx')
const ram = require('random-access-memory')

const store = new Corestore(ram)
await store.ready()

const networker = new SwarmNetworker(store)
networker.listen()

// Start announcing or lookup up a discovery key on the DHT.
await networker.seed(discoveryKey, { announce: true, lookup: true })

// Stop announcing or looking up a discovery key.
networker.unseed(discoveryKey)

// Shut down the swarm (and unnanounce all keys)
await networker.close()
```

### API

#### `const networker = new SwarmNetworker(dwebx, networkingOptions = {})`
Creates a new SwarmNetworker that will open replication streams on the `dwebx` instance argument.

`networkOpts` is an options map that can include all [dwswarm](https://github.com/dwswarm/dwswarm) options as well as:
```js
{
  id: crypto.randomBytes(32), // A randomly-generated peer ID,
  keyPair: HypercoreProtocol.keyPair(), // A NOISE keypair that's used across all connections.
}
```

### License
MIT
