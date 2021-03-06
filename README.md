## Finagle Consul

Service discovery for Finagle cluster with Consul. Multi DC, custom health checks.

### About

[Consul](https://www.consul.io/) is a tool for service discovery and configuration. Consul is distributed, highly available, and extremely scalable.

`finagle-consul` provides you Announcer and Resolver to work with low-level Consul HTTP API.

**Warning! This is still BETA.**

![Consul Web UI](https://s3.amazonaws.com/attendify-assets/code/Screen+Shot+2015-04-08+at+10.24.22+PM.png)

### QuickStart

Install and run Consul (see [docs](https://www.consul.io/intro/getting-started/install.html) for more information):

```shell
$ consul agent -server -bootstrap-expect 1 -data-dir /tmp/consul -ui-dir ~/Downloads/dist -log-level debug
```

Run one or more example servers (each one will be bound to the random socket):

```shell
$ sbt 'run-main com.twitter.finagle.consul.examples.Server'
[info] Running com.twitter.finagle.consul.examples.Server
Run server: sgN72ffoxX
Apr 08, 2015 10:45:58 PM com.twitter.finagle.consul.ConsulAnnouncer register
INFO: Register consul service ConsulService(3fc29cfa-75f6-4b59-8f2f-591c893a0f13,RandomNumber,192.168.0.104,57007)
```

You can now check list of registered services in [Consul Web UI](https://www.consul.io/intro/getting-started/ui.html).

Run example client:

```shell
$ sbt 'run-main com.twitter.finagle.consul.examples.Client'
[info] Running com.twitter.finagle.consul.examples.Client
Run CLIENT: 7spceT1R8f
Apr 08, 2015 10:48:12 PM com.twitter.finagle.consul.ConsulResolver$$anonfun$readCatalog$1 apply
INFO: Consul catalog lookup at localhost:8500 to look for RandomNumber: List(/192.168.0.104:57007, /192.168.0.104:57071)
Server:sgN72ffoxX; Client:7spceT1R8f; Req:50; Resp:p6Cli
Server:XmI4bk6khp; Client:7spceT1R8f; Req:47; Resp:p3lQb
Server:sgN72ffoxX; Client:7spceT1R8f; Req:42; Resp:ZJlzj
<...truncated output...>
```

### Consul path definition

To announce your service use following scheme:

```scala
consul!host1:port1,host2:port2,...!serviceName?additional=params
```

For example,

```scala
val server = Http.serveAndAnnounce("consul!127.0.0.1:8500!/RandomNumber")
val client = Http.newService("consul!127.0.0.1:8500!/RandomNumber")
```

Additional params:

```scala
scala> import com.twitter.finagle.consul.ConsulQuery
import com.twitter.finagle.consul.ConsulQuery

scala> ConsulQuery.decodeString("/RandomNumber?tag=prod&tag=tracing&dc=dc1&ttl=45")
res1: Option[com.twitter.finagle.consul.ConsulQuery] = Some(ConsulQuery(RandomNumber,Some(45.seconds),Set(prod, tracing, finagle),Some(dc1)))
```

Few notes:

1) You can specify name as URL, but all "/" will be replaced with ".":

```scala
scala> ConsulQuery.decodeString("/prod/cluster22/RandomNumber?ttl=45").get.name
res5: String = prod.cluster22.RandomNumber
```

2) TTL is defined in seconds. Default TTL value = 100000000 seconds (~3 years). Specifying TTL will register TTL-based health-check and will schedule periodical updates for health check status (now this is only one reasonable way to manipulate with service visibility in the cluster).

3) Tag "finagle" will be added automatically (so you can see all service registered from Finagle Consul in Consul Web UI).

4) Specifying datacenter that is not known yet to Consul cluster will lead to error (Consul will reject registeration request).

### TODO

- [x] TTL configuration
- [x] Custom tags for Announcer
- [x] Multi DC configuration
- [x] Tags filters for Resolver
- [ ] Package distribution
- [ ] Unit tests for consul query decode
- [ ] EndToEnd integration tests
- [ ] Debug level log messages for all Consul server queries
- [ ] Process Consul HTTP API errors/timeouts

(see also numerous "XXX" comments in code)

### Known issues

- "deregister" HTTP command works a bit unpredictably

- is any way to watch changes from HTTP API endpoint (?)

- registering service for unknown datacenter leads to silent error