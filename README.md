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

This command will create 10 containers, 1 on each node, named `nginx_1` on node `enode1`, `nginx_2` on node `enode2` and so on.

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
