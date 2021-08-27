# Path and Movement

----

Path finding and moving is an indispensable part of almost all bots, and a CPU consuming one. If not taken care of,
wrong path finding result can prevent creeps from doing their jobs, and may eat CPU bucket pretty quickly and lead to
catastrophic failure in the end.

The path finding and moving are separated in the clarkok bot, which means we don't use the default `moveTo` in most of
cases. When a creep or a squad need to go somewhere, it will first do a path search, cache the result in the memory if
succeeded, then follow the path and go toward the destination. Unlike the default behavior of the `moveTo` that re-path
every time the creep cross a room boundary, in the clarkok bot the creeps will only search the path at the very
beginning of the movement, and use that path until they reach their destinations, or get blocked.

Also given the advantage of using the generator based thread, a handful of helper functions are added for most of the
normal cases. For example, one hauler can just use below snippet to go fetch the resource, and put it back to storage:

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

For most of the inter-rooms path finding, we can't have visibility for all the rooms in the path, but we can't assume
all the rooms are safe and clear for the creeps to pass by. So the clarkok bot will collect and cache intel from
surrounding rooms proactively and refresh them from time to time to keep them up to date.  To support path finding, the
clarkok bot collect a few kinds of information as listed below:

1. Room Type - Each room in the map will be categorized in one of the RoomType including Owned (owned by the bot
   itself), Hostile(owned by other players), Plain(unowned rooms or rooms reserved by the bot), Reserved(by other
   players), SourceKeepr, HighWay, and etc.
2. Owner Username - If the room is owned by other player or is occupied by the invader stronghold, we record the owner's
   username
3. Blockage - A compressed bitmap storing all the walkable tile in the room. As the size can be large we only store this
   field when the bot find it is necessary
4. A few points of interests like InvaderLair

## Path Finding

The path finding is done in 2 steps, first we do a `Game.map.findRoute` call to get a list of rooms, and then a
`PathFinder.search` call to only search paths in those rooms returned by `findRoute`. We are doing so to reduce the path
finding cost during long range path finding, especially in those cases where multiple hostile rooms lies in between the
source and the destination.

As a general rule, the clarkok bot will not path through any rooms owned by other players, except the owner is a friend.
So in the `findRoute` step we will rule those hostile owned rooms out, together with rooms with different `roomStatus`
with the source room, to prevent the path go across regions like respawn zones. Also, we will give those rooms with
`blockage` field higher cost, to prefer rooms without blockage. It works well most of time but this solution did cause
some trouble in the Season 2 :(

When in the `PathFinder.search` step, we always ignore creeps, and will resolve conflict in the moving phase

## Moving


