# Resource Management and Logistics

====

The clarkok bot can make use of almost all the resources in the game, from the basic energy / minerals to the power and
commodities. While the solution is not as fancy as the Overmind one, it is at least simple to implement;

## In-room Energy Management

In peace time, there are 2 major energy income and 3 major sinks. Most of the energy comes from remote outposts, and the
in room sources. And it will mostly be spent on upgrading controller, spawning creeps and building structures /
ramparts. In clarkok bot, the whole energy flow is driven by sink side.

The colony level state machine will monitor the room status, and issue tasks when it thinks needed. Say if some
extensions are empty, the colony level state machine will issue refill task, or if there are construction sites, the
state machine will issue build task.

Once those task get assigned, the executor creep will ask the colony for available energy then pause itself when waiting
for the response. And in the middle of a tick, the colony will summarize the energy requirement by each task, and the
available energy in the room, like dropped energy, the energy in miners container, the energy in storage or in terminal.
And the colony will assign the available energy to each task, base on a few conditions like task priority, the distance
between energy and the creep. Once the energy requirement get fulfilled, the task will be unpaused and the creep will go
fetch that assigned energy and continue its work. The colony will keep some memory on the energy already assigned, so
we won't get in the situation where a creep went a long distance only to find empty containers / tombstones.

The refilling and building tasks tends to take higher priority in normal situation, they will be assigned energy to no
matter how many energy is left. While we only do upgrade when we at certain energy level.

## Resource Storage and Federal-wise Balancing

Just like most of bots, the clarkok bot will also store resources in both storage and terminal, but different resource
will have different preference when choosing the store structure. The bot uses a declarative way to specify how and
where to store a resource, using below interface.

```typescript


interface ResourceBalanceInfo {
    terminalShort: number;
    terminalLong: number;

    terminalMin: number;
    terminalMax: number;

    storageMax: number;

    factoryMin: number;
    factoryMax: number;

    residesIn: "terminal" | "factory";
}

```

Let's focus on the `min` / `max` pairs for terminal and factory first. It doesn't matter where to put the resource when
receiving. Some tasks will put the resource in terminal, and others will put it in storage. Once the resource has been
put in, the manager will periodically check the level of each resources. It will try to satisfy the need of terminal /
factory first, if the amount of resource is below `xxxMin`. And in return, if the amount is higher than `xxxMax` the
manager will collect the resource back to storage.

The `terminalShort` / `terminalLong` pair is for balancing the resource across colonies. If the amount of some resource
in terminal is below `terminalShort` in one colony, the federal state machine will ask others to transfer some, if there
is exceeding resource in those colonies. But if no such colony can be found, we will try to buy it from the public
market. However for some resource, if the amount in the terminal is higher than `terminalLong`, and the amount in the
storage is also higher than `storageMax`, we will try to sell it.

We will have different balancing info for different resource in different colony. And the manager / federal level state
machine will help balancing those resource automatically. 
