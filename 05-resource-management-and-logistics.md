# Resource Management and Logistics

The clarkok bot can make use of almost all kinds of the resources in the game, from the basic energy and minerals to the power and commodities. Each room can keep a relative healthy economy by themselves and when needed rooms will supporteach other.

## In-room Energy Management

In peace time, there are 2 major energy incomes and 3 major sinks. Most of the energy comes from remote outposts, andthe in room sources. And it is mostly be spent on upgrading controller, spawning creeps and building structures /ramparts. In the bot, the whole energy flow is driven by sink side.

The colony level state machine will monitor the room status, and issue tasks when it feels necessary. Say if someextensions are empty, the colony level state machine will issue refill tasks, or if there are construction sites, thestate machine will issue build tasks.

Once those tasks get assigned, the executor creep will ask the colony for energy and pause itself when waiting for theresponse. Then in the middle of the same tick, the colony will summarize the energy requirement and the availableenergy in the room, like dropped energy, the energy in miners' containers, the energy in storage or in terminal. After that thecolony will assign the available energy to each task, base on a few conditions like task priority, the distance betweenenergy and the creep. Once the energy requirement get fulfilled, the task is unpaused and the creep will go fetchthat assigned energy and continue its work. The colony will keep memory on the energy already assigned, so we won't getin the situation where a creep go a long distance only to find empty containers / tombstones.

The refilling and building tasks tend to have higher priority in normal situation, the colony will allocate energy tothem no matter how much energy is available. While we only do upgrade when we have more energy than we need to reserve.

## Resource Storage and Federal-wise Balancing

Like most of bots, the clarkok bot will store resources in both storage and terminal, but different resource will havedifferent preference when choosing the store structure. The bot uses a declarative way to specify how and where to storea resource, using the below interface.

```typescript
interface ResourceBalanceInfo {
    terminalShort: number;
    terminalLong: number;

    storageMax: number;

    terminalMin: number;
    terminalMax: number;

    factoryMin: number;
    factoryMax: number;

    residesIn: "terminal" | "factory";
}
```

Let's focus on the `min` / `max` pairs for terminal and factory first. It doesn't matter where to put the resource whenreceiving. Some tasks will put the resource in terminal, and others will put it in storage. Once the resource has beenput in, the manager will periodically check the balancing status of each resources. It will try to satisfy the need ofterminal / factory first, if the amount of resource is below `xxxMin`. And in return, if the amount is higher than`xxxMax` the manager will collect the resource back to storage.

The `terminalShort` / `terminalLong` pair is for balancing the resource across colonies. If the amount of some resourcein terminal is below `terminalShort` in one colony, the federal state machine will ask transferring from those colonieshave exceeding resource. If no such colony can be found, we will try to buy it from the public market. However for someresource, if the amount in the terminal is higher than `terminalLong`, and the amount in the storage is also higher than`storageMax`, we will try to sell it.

We will have different balancing info for different resource in different colony. And the manager / federal level statemachine will help balancing those resource automatically.

