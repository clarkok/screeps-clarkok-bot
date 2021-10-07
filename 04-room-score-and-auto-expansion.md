# Room Score and Auto Expansion

----

The clarkok bot has the ability to choose its next expanding target. To achieve this, the bot will score every room ithas intel of, and choose a best room when doing auto expansion. The score is calculated in below manner:

> Room Score = Nature Score * Environment Multiplier

The nature score part means how good does a room look like on its own. It depends on the invariable things like the terrain,locations of nature objects like its controller, sources and mineral, and its surrounding rooms. On the other side, theenvironment multiplier means how good is a room in the current situation. It depends on the changing environment likelocations and mineral kinds of the current owned rooms, enemies in this region, and etc.

The nature scores will be pre-calculated and cached only once for each room, in the background with exceeding CPU. Andit is also used to determined the best room layout. While the environment multiplier will only be calculated whenneeded, just before the expansion target selection.

## Nature Score

The clarkok bot expresses the nature score in the unit `Energy / Tick`. The nature score also consists of two parts, and the ratio varies for different configurations.

1. the **development score**, the estimated energy per tick dumped into the controller till RCL6
2. the **contribution score**, the estimated energy per tick the room can share with others after RCL7

For each attempt in the [room layout](03-room-layout.md) step, the planner will calculate the nature score with the planresult yielded in the current attempt, and choose the attempt yield the best room score as the final plan result.

### Development Score

This part is calculated as below:

> Ticks to Rcl6 = Total Energy Consumption / Total Energy Net Income per Tick with a Single Spawn
> Development Score = Energy Required for Controller / Ticks to Rcl6

Where total energy consumption consists of the energy for upgrading controller, the energy for building structures and build upramparts, together with the energy spent on creeps to haul / build.

### Contribution Score

This part is calculated as below:

> Contribution Score = Total Energy Net Income per Tick - Structure Up Keep Energy per Tick

## Environment Multiplier

The environment multiplier is a float point number to be multiplied to the nature score when choosing the expansion target. Factors like the mineral situations, distance from owned colonies and failed expansion attempts can all affect this multiplier and as a result affect the target selection.

## Auto Expansion

The auto expansion happened upon the GCL is greater than the currentt owned room count. As for now the clarkok bot doesn't have multi-shards ability, so we don't consider the GCL distribution across shards.

Auto expansion start with iterate through rooms with available room nature scores, the bot will calculate environment multiplier for each of them, and choose the one with highest final score. Then the bot will find the most suitable colony for the expansion target, and place a directive with the directive system mentioned in the [task management](02-task-management.md). After that the chosen colony will start working on the expansion.

The auto expansion will have a give up clock for each attempt. If the time is up and the expansion is still not done, we will give up and record the failed attempt, and in the future we will lower the environment multiplier for this room to avoid duplicated failures.
