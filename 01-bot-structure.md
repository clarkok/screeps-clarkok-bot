# Bot Structure

====

Clarkok bot is a multi-room bot, it structures its owned rooms as colonies. Each colony consists of an owned room, and
its surrounding remote outposts. And all the colonies join their force and form a federal state.

Like a few other bots, the clarkok bot uses state machines as its primary control method. More specifically, the bot
uses a three-level state machine. From a top-down order, they are federal level, colony level and task level.

 * The federal level controls the highest level decision making, and coordinates all the colonies, it will give orders
   to the colony level, either via memory, or via a flag-based directive system to be mentioned below 
 * The colony level controls whatever inside a colony, like managing resources, spawning creeps and assigning tasks to
   them
 * The task level controls a single creep or a squad during a single task

And each level can be broken down to multiple small features, those features manage their own states. The state machines
are not written out explicitly. For federal level and colonies level features, each of them will have a method
`TickFederalXXX` or `TickDirectiveXXX` which will be invoked for each tick, to check and modify their own states. For
task level tasks, each of them will have a generator function whose `.next()` will be called each tick, to operate their
creeps or squads.

## The Command Chain, Directive System and Task System

The highest commands will be given by human player or the federal level state machine. And most of time the command will
be given in the following format: "HOW do WHICH colony do WHAT at WHEN", for example I need colony W3N8 attack an
invader core in room W4N5 using a single attacker now. The directive is given by place flag, whose primary color is red
(attack), and secondary color is red (single attacker), in W4N5 (the target), and set the flag name to
`Flag1?owner=W3N8`, indicating the directive is given to W3N8. The flag can be placed either by human, or by the federal
level.

There are 2 major reasons to use the flags here, the first being the flags is visible to both human player, and the bot,
so I can see those existing directives when viewing the target room. And the second reason is what make flags better
than visuals, I can remove the flag if I want to stop the directive. For now we don't have a working android client,
adding / removing flags is the primary approach that I use to control my bot using my phone : )

Once a flag is put on the map, by either the player or by the federal level state, it will be picked up by the colony
level. The colony level feature checker for this directive will be triggered to parse the flag name / location and
meaning, and appending a directive object to the colony's memory. If no owner can be found in the flag name, the
corresponding directive feature will find the most suitable colony and continue, otherwise remove the flag if no such
colony can be found.

Then for each tick, the directive executor will be called for each colony. The directive executor will check the current
situation for the directive, require corresponding creeps or squads from the colony, and assign tasks to them. And once
the tasks is assigned, a generator-based thread will be spawned to operate the creep / squad to finished the task.

## Federal Level

The clarkok bot is more like a federal state instead of an empire, where colonies operate almost independently and they
will help each other only when needed / asked to. For now in the federal level, we have 4 features.

 * *Assigning outposts* - for calculating optimal outpost assignment for each owned room, base on the CPU, net gain for
   the outpost, the spawn availability and etc.
 * *Balancing resources* - for balancing exceeding mineral / resources between colonies, and sell / buy resources if
   public market is available
 * *Pre-calculating* room score - for running room planner for every room and record the result, so when expanding the
   bot will know which room is a good target
 * Season #2 related

## Colony Level

There are 2 kinds of features in the colony level. The first kind of features are called strategies, every colonies will
execute them, and the second kind of features are called directives, they will only be assign to colonies when needed.
As for now the bot has 4 strategies, including development, mineral-related, intel and defence, and 21 directives,
including offense, claim, deliver energy / resource to other location or help new colonies upgrade faster. 

## Task Level

As for now the bot has more than 40 tasks for individual creep, and a handful tasks for squads. Some of the tasks are
lifetime tasks, once assigned to a creep or a squad, it will continue operate them until they die out. And others have a
terminate condition, once the task is done, the creep / squad will be set free and can be re-assigned to another task.
