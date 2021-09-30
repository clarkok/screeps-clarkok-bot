# Path and Movement

----

Path finding and moving is an indispensable part of almost all of the bots, and also a CPU consuming one. If not taken care of.Wrong path finding result can prevent creeps from doing their jobs, and may eat CPU bucket quickly and lead to
catastrophic failure as the results.

The path finding and moving are separated in the clarkok bot, which means we don't use the default `moveTo` in most of
cases. When a creep or a squad need to go somewhere, it will first do a path search, cache the result in the memory if
succeeded, then follow the path and go toward the destination. Unlike the default behavior of the `moveTo` that re-path
every time the creep cross a room boundary, in the clarkok bot the creeps will only search the path at the verybeginning of the movement, then use that path until they reach their destinations, or get blocked.

Also given the advantage of using the generator based threads, a handful of helper functions are added for common tasks. For example, one hauler can just use below snippet to go fetch the resource, and put it back to storage:

``` typescript

// c: a wrapper object to reference a creep across ticks, c.creep will return the actual creep object at that tick or
//    null if the creep is dead

yield* FetchResource(c, task.resource);

if (!c.creep?.store.getUsedCapacity()) {
    // if the creep is dead of it does't have anything in the store, return from the task generator
    return;
}

// convient helper function to 1) find a path, 2) cache the path and 3) move to that pos
yield* MoveToPos(c, 1 /* range */, colony.storage.pos /* assumeing the storage exists */);

while (c.creep?.store.getUsedCapacity()) {
    c.creep.transfer(colony.storage, Object.keys(c.creep.store)[0]);
    yield false;    // false means we don't need to resume the generator in the same tick
}


```

## Intel to Support Path Finding

For most of the inter-room path finding, we don't have visibility for all the rooms in the path, and we can't assume
all the rooms are safe and clear for the creeps to enter. So the clarkok bot will collect and cache intel ofsurrounding rooms proactively and refresh it from time to time to keep it up to date.  To support path finding, the
clarkok bot collect a few kinds of information as listed below:

1. Room Type - Each room in the map will be categorized in one of the RoomType including Owned (owned by the botitself), Hostile(owned by other players), Plain(unowned rooms or rooms reserved by the bot), Reserved(by otherplayers), SourceKeepr, HighWay, and etc.
2. Owner Username - To determine the owner of the room is friendly or not
3. Blockage - A compressed bitmap storing all the walkable tile in the room. As the size can be large we only store thisfield when the bot think it is necessary
4. A few points of interest like the invader lairs

## Path Finding

The path finding is done in a layered manner. The full searching involves 3 layers, 1) the portal layer, 2) the room layer and 3) the tile layer.

The topmost layer will search for a set of portals to go through between the source and the destination. It will only be enabled when the source and the destination are more than 15 rooms away, or the second layer (aka. the rooms layer) failed. The portal information can be retrieved from cached info, or the failed attempt second layer.

The second layer will use a so called "exit based find routine" method, to search for a set of room exits that form a path from the source to the destination, or to portals yielded from the first layer. The exits used here are generated from the nature exits, with some other blockage information. For example, some sk room lairs are too close to the room exit, and some or even all of the tiles on the exits are exposed to the source keepers. In those case we will shrink the exit, cut the exit into multiple parts, or just ignore the exit. Besides the exits, this layer also filter out some rooms which is not appropriate for creeps to enter, like the hostile owned rooms. This layer will only be enabled when the source and the destination are more than 3 rooms away, or the first attempt of third layer search failed.

And the third layer, will do a `PathFinder.seach` to do the actual walkable tile path search. This search will only consider rooms returned by the second layer, to lower the over all cost. The cost matrices used in this layer is also cached and shared between most of the tasks.

When in the `PathFinder.search` step, we always ignore creeps, and will resolve conflict in the moving phase.

## Moving

After we getting a valid path, the movement related of methods will cache it in the task memory so it can survive global resets. And the unit will follow the cached path all along the trip, until it is blocked (which means the path is no longer valid, and we will refresh Intel / cost matrix cache, and do the path search again), or there are hostile creeps with attaching body parts in the same room.

### Avoid Hostile Creeps

Units in the clarkok bot will try to avoid hostile creeps with attacking body parts to keep themselves from being harmed. When such hostile creeps appear in the room, our units will stop following the precalculated path, instead they will do a single room path finding every tick with a CM generated marking tiles within hostile creeps attack range unwalkable. And the destinations of this kind of path searching will be the very last room position in this room in the previous precalculated path. So once our units leave the dangerous room, they can continue their original path.

### Resolve Traffic Jams

During movement the units in the clarkok bot always ignore other creeps. As a result sometimes the units will block each other when they are waiting on different conditions. For example a builder may stay on a tile for sometime and try building something, while a hauler trying to go through the same tile to bring some resources to somewhere. Without proper handling the hauler may need to wait long enough and the overall efficiency would be harmed.

To solve the traffic jams, the moving unit can ask the blocking creep to move aside after a few ticks waiting. And we prefer 90 degree directions if the blocking creep has nothing to do (a free creep), or towards next tile in its path if it is moving. But if the preferred directions are not feasible, all directions will be considered.
