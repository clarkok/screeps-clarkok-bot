# Room Score and Auto Expansion

----

The clarkok bot has the ability to choose it next expanding target. To achieve this, the bot will score every room it
has intel of, and choose a best room when doing auto expansion. The score is calculated in below manner:

> Room Score = Nature Score * Environment Multiplier

The nature score part means how good is a room on its own. It depends on the invariable things like the terrain,
locations of nature objects like its controller, sources and mineral, and its surrounding rooms. On the other hand, the
environment multiplier means how good is a room on the current situation. It depends on the changing environment like
locations and mineral kinds of the current owned rooms, enemies in this region.

The nature scores will be pre-calculated and cached only once for each room in the background, with exceeding CPU. And
it is also used to determined the best room layout. While the environment multiplier will only be calculated when
needed, just before the expansion target selection.

## Nature Score

The clarkok bot express the nature score in the unit `Energy / Tick`. The nature score also consists of two parts:

1. the **development score**, the estimated energy per tick dumped into the controller on its way to RCL6
2. the **contribution score**, the estimated energy per tick the room can share with others after RCL7

For each attempt in the [room layout](03-room-layout.md) step, the planner will calculate the nature score with the plan
result yielded in the current attempt, and choose the attempt yield the best room score as the final plan result.

### Development Score

This part is calculated as below:

> Ticks to Rcl6 = Total Energy Required / Total Energy Net Income per Tick
> Development Score = Energy Required for Controller / Ticks to Rcl6

Where total energy consists of the energy for upgrading controller, the energy for building structures and top up
ramparts, together with the energy spent on creeps to haul / build.

### Contribution Score

This part is calculated as below:

> Contribution Score = Total Energy Net Income per Tick - Structure Up Keep Energy per Tick 

## Environment Multiplier


