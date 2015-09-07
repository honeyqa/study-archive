# 작성중.....


# Title
## Intro.
etcd를 이용한 죽지 않는 서버 만들기

어떻게?

## [etcd](https://coreos.com/etcd/docs/latest/getting-started-with-etcd.html)
### etcd란

1. simple interface
2. key/value starage
3. watch for changes

### Install

***OSX***

    $ curl -L  https://github.com/coreos/etcd/releases/download/v2.1.3/etcd-v2.1.3-darwin-amd64.zip -o etcd-v2.1.3-darwin-amd64.zip
    $ unzip etcd-v2.1.3-darwin-amd64.zip
    $ cd etcd-v2.1.3-darwin-amd64 
    $ ./etcd
Open another terminal:

    $ ./etcdctl set mykey "this is awesome"
    this is awesome
    $ ./etcdctl get mykey
    this is awesome

***OSX Homebrew***

    $ brew install etcd
    $ etcd
    
Open another terminal:

    $ etcdctl set mykey "this is awesome"
    this is awesome
    $ etcdctl get mykey
    this is awesome


### etcdctl
ummm...

## [node-etcd](https://www.npmjs.com/package/node-etcd)

### Install

For nodejs >= 0.10 and iojs:

    $ npm install node-etcd

For nodejs == 0.8:

    $ npm install node-etcd@3.0.2

### Methods
**Basic usage**

    Etcd = require('node-etcd');
    etcd = new Etcd();
    etcd.set('key', 'value');
    etcd.get('key', console.log);
    etcd.del('key', console.log);

**.create(path, value, [options], [callback])**

Atomically create in-order keys.

    etcd.create('queue', 'first');
    etcd.create('queue', 'next', console.log);
    
**.watcher(key, [index], [options])**

Returns an eventemitter for watching for changes on a key

    watcher = etcd.watcher("key");
    watcher.on("change", console.log); // Triggers on all changes
    watcher.on("set", console.log);    // Triggers on specific changes (set ops)
    watcher.on("delete", console.log); // Triggers on delete.
    watcher2 = etcd.watcher("key", null, {recursive: true});
    watcher2.on("error", console.log);

You can cancel a watcher by calling ```.stop()```.

Signals:

* ```change``` - emitted on value change
* ```reconnect``` - emitted on reconnect
* ```error``` - emitted on invalid content
* ```<etcd action>``` - the etcd action that triggered the watcher (ex: set, delete).
* ```stop``` - watcher was canceled.
* ```resync``` - watcher lost sync (server cleared and outdated the index).

Use the ```.watch()``` command in you need more direct control.

## Node.js and MySQL for Auto-Recovery System

3. get 
4. watcher instance
5. etcd 초기화(setup, notify, connect)
6. watcher 작성
7. test
8. result

## REFERENCE

* [coreos-etcd-api](https://coreos.com/etcd/docs/latest/api.html)

* [coreos-etcd-git](https://github.com/coreos/etcd/tree/master/etcdctl)

* [etcd-tutorials](https://www.digitalocean.com/community/tutorials/how-to-use-etcdctl-and-etcd-coreos-s-distributed-key-value-store)

* [soma6th-git](https://github.com/swmaestro6th-crashreport/nodejs-etcd-mongodb)

* [soma6th-slideshare](http://www.slideshare.net/parkdainel/etcd-db-watcher?qid=cd557011-f6f6-474c-b5d0-69fecc711b87&v=default&b=&from_search=1)
