# Rollback

At its core, Borger is a rollback framework. This is the idea that an older [**simulation tick ID**](./simulation-and-presentation.md#simulation) can rerun, when information pertaining to the past finally arrives over the slow internet connection.

Let's say the [**server**](./server-and-client.md#server) is chugging along as usual:

```
Tick ID 100
         ↓
Tick ID 101
         ↓
Tick ID 101
         ↓
        etc.
```

And a [**client**](./server-and-client.md#client) is connected to that server, chugging alongside it in perfect sync:

```
Tick ID 100
         ↓
Tick ID 101
         ↓
Tick ID 101
         ↓
        etc.
```
