# Development documentation

Not much here yet, just a command for Meltano how to lock the dependencies
after adding/editing component definitions.

```shell
meltano lock --update --all
```

In most cases, `install` needs to be executed afterwards.

```shell
meltano install
```
