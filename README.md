# cdocker

An easy way to run commands across a cluster of Docker Engines without needing a Swarm.

## Example

```
cdocker run -d -e "constraint:node==enode{1..10}" --name "nginx_{1..10}" -p 80:80 nginx
```

It looks like a regular Docker run command but with a few smart additions:

- Prefix your command with `cdocker` instead of `docker`
- Use classic Swarm mode constraints
- Use *brace expansion* in constraint and name parameters to create arrays of containers

This command will create an array of 10 containers, 1 on each node, named `nginx_1` on node `enode1`, `nginx_2` on node `enode2` and so on.

In addition to brace expansions, you can use globbing:

```
cdocker kill "nginx_*"
```

will kill containers on all nodes with names that match the `nginx_*` pattern.

You can narrow this down by specifying which nodes, again similar to how they are represented in classic Swarm:

```
cdocker kill "enode{1..5}/nginx_*"
```

will kill the matching containers only on `enode1` to `enode5`.

## Installation

It is written in Node.js, so install using `npm`:

```
sudo npm install -g cdocker
```

and add your nodes like so

```
cdocker node add 192.168.0.2
```

The default port we will try and find your Docker Engines on is `2375` but that can be changed with the `--port` option.

Of course, you can also use brace expansion here

```
cdocker node add "192.168.0.{11..20}"
```

When you add or remove nodes, this list will be stored in the `~/.cdocker.json` file.

## Scheduling

Scheduling arrays of containers to nodes is achieved by first finding nodes that match the specified constraints.

> At the moment we only support name and label constraints

Then, one of 3 situations will be reached:

- There are less nodes matched than there are containers you want to start (nodes < containers)
- There are exactly the same number of nodes to containers you want to start (nodes == containers)
- There are more nodes matched than there are containers you want to start (nodes > containers)

### Less nodes than containers you want to start

In this instance, `cdocker` will use a round robin strategy to distribute the containers among the matched nodes.

For example, if you want to start 10 containers on 5 nodes, each node will have 2 containers on it.

### Exactly the same number of nodes as containers you want to start

In this instance, `cdocker` will create a *straight up* mapping of 1 container to 1 node.

### More nodes than containers you want to start

In this instance, you can decide between different strategies on how to utilise the available nodes that matched your constraints.

- `unique-random` will schedule the containers to random nodes, never using the same node twice
- `least-running` will order the nodes based on how many containers are already running on them and use those that are the least loaded

You can specify the strategy in the `run` command as an environment variable, just like the constraints:

```
-e strategy:unique-random
```

If you don't specify a strategy, `least-running` will be used by default. However, this requires enumeration of all nodes which may not be performant with very large clusters.
