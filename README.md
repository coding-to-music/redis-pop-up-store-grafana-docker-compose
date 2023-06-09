# redis-pop-up-store-grafana-docker-compose

# 🚀 Pop-up store demo using RedisTimeSeries, RedisGears, and Redis Data Source for Grafana 🚀

https://github.com/coding-to-music/redis-pop-up-store-grafana-docker-compose

From / By https://github.com/ajeetraina/redis-pop-up-store

https://redis.com/blog/3-real-life-apps-built-with-redis-data-source-for-grafana/

https://github.com/ajeetraina/awesome-compose


## Environment variables:

```java

```

## user interfaces:

- ReactApp http://localhost:3000

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/redis-pop-up-store-grafana-docker-compose.git
git push -u origin main
```

<h1 align="center">Pop-up store demo using RedisTimeSeries, RedisGears and Redis Data Source for Grafana</h1>

<div id="badges" align="center">

[![Grafana 7](https://img.shields.io/badge/Grafana-7-blue)](https://www.grafana.com)
[![RedisTimeSeries](https://img.shields.io/badge/RedisTimeSeries-inspired-yellowgreen)](https://oss.redislabs.com/redistimeseries/)
[![RedisGears](https://img.shields.io/badge/RedisGears-powered-orange)](https://oss.redislabs.com/redisgears/)
[![Grafana-Redis-Datasource](https://img.shields.io/badge/GrafanaRedisDatasource-integrated-yellow)](https://github.com/RedisTimeSeries/grafana-redis-datasource)

</div>

## Description

The Pop-up store demo is using [Redis Streams](https://redis.io/topics/streams-intro), [RedisTimeSeries](https://oss.redislabs.com/redistimeseries/), [RedisGears](https://oss.redislabs.com/redisgears/) and [Redis Datasource](https://github.com/RedisTimeSeries/grafana-redis-datasource) to visualize data pipeline in Grafana.

![Pop-up](https://github.com/RedisTimeSeries/redis-pop-up-store/blob/master/images/pop-up.gif)

## How it works

![Diagram](https://github.com/RedisTimeSeries/redis-pop-up-store/blob/master/images/pop-up.png)

- Node.js script adds random data to Customers and Orders streams
- RedisGears is using `StreamReader` to watch all `queue:` keys and adding Time-Series samples

```
# Add Time-Series
def tsAdd(x):
   xlen = execute('XLEN', x['key'])
   execute('TS.ADD', 'ts:len:'+x['key'], '*', xlen)
   execute('TS.ADD', 'ts:enqueue:' + x['key'], '*', x['value'])


# Stream Reader for any Queue
gb = GearsBuilder('StreamReader')
gb.countby(lambda x: x['key']).map(tsAdd)
gb.register(prefix='queue:*', duration=5000, batch=10000, trimStream=False)
```

- Another RedisGears script completes orders
  - adding data to `queue:complete` stream
  - deleting client's ordering
  - decreasing product amount
  - trimming Orders queue

```
# Complete order
def complete(x):
    execute('XADD', 'queue:complete', '*', 'order', x['id'],
            'customer', x['value']['customer'])
    execute('XDEL', 'queue:customers', x['value']['customer'])
    execute('DECR', 'product')


# Stream Reader for Orders queue
gb = GearsBuilder('StreamReader')
gb.map(complete)
gb.register(prefix='queue:orders', batch=3, trimStream=True)
```

- Grafana query streams and Time-Series keys every 5 seconds to display samples using Grafana Redis Datasource.

## What is displayed on Grafana dashboard

- `Product Available` - the value of `product` key
- `Customers Ordering` - length of `queue:customers`
- `Orders Processing` - length of `queue:orders`
- `Orders Completed` - length of `queue:complete`
- `Customers Overflow` - the difference between customer submitted orders and orders completed
- `Customers Ordering` - change of `queue:customers` length
- `Orders In Queue` - change of `queue:orders` length
- `Completed Flow` - how many orders processed

## Requirements

- [Docker](https://docker.com) to start Redis and Grafana.
- [Node.js](https://nodejs.org) to run simulation script.

## Start Redis with RedisTimeSeries and RedisGears modules installed and Grafana

For detailed instructions please take a look at [redismod - a Docker image with select Redis Labs modules](https://hub.docker.com/r/redislabs/redismod).

```
npm run start:docker
```

## Register [StreamReaders](https://oss.redislabs.com/redisgears/readers.html#streamreader)

Install Readers to add Time-Series and complete orders

```
npm run register:gears
```

## Install [ioredis](https://github.com/luin/ioredis) module and run simulation

Script `pop-up-store.js` will add customers to stream `queue:customers` and their orders to `queue:orders`.

```
npm run start:simulation
```

## Open Grafana Dashboard using browser http://localhost:3000

## Redis-cli

To start `redis-cli` and look at the keys please run

```
npm run redis-cli
```
