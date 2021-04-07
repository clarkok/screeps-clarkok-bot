# Path and Movement

====

Path finding and moving is an indispensable part of almost all of the bots, but yet a CPU consuming one. If not taken
care of, not only the creeps won't get their job done, higher CPU consumption can kill the bucket and eventually leads
to catastrophic failures.

The path finding and moving are separated in the clarkok bot, which means we don't use the default `moveTo` in most of
cases. When a creep or a squad need to go somewhere, it will first do a path search, cache the result in the memory if
succeeded, then follow the path and get to the destination. Unlike the default behavior of the `moveTo` that re-path
every time the creep cross a room boundary, in the clarkok bot the creeps will only search the path at the beginning of
the movement, and use that path until they got blocked, or reached their destinations. 

Also given the advantage of using the generator based thread, a handful of helper functions are added for most of the
normal cases. So in the clarkok bot, one hauler can just use below snippet to go fetch the resource, and put it back to
storage:

``` typescript

// c: a wrapper object to reference a creep across ticks, c.creep will return the actual creep object at that tick or
//    null if the creep is dead

yield* FetchResource(c, task.resource);

if (!c.creep?.store.getUsedCapacity()) {
    // if the creep is dead of it does't have anything in the store, return from the task generator
    return;
}

yield* MoveToPos(c, 1 /* range */, colony.storage.pos /* assumeing the storage exists */);

while (c.creep?.store.getUsedCapacity()) {
    c.creep.transfer(colony.storage, Object.keys(c.creep.store)[0]);
    yield false;    // false means we don't need to resume the generator in the same tick
}


```

## Path Finding
