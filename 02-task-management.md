# Task Management

Instead of the so-called role-based approach mentioned in the Screeps tutorial, the clarkok bot uses a task-basedsolution, together with a role requirement system to manage the creeps and tasks in each colony.

Creeps used by the clarkok bot are categorized in roles, for example we have role Miner, Carrier, Worker etc. As for now,the bot has 38 roles. The meaning of the term 'role' here is slightly different to the 'role' in the tutorial. A role in the clarkok bot decides the layout of creep body with current available energy in the room, and decides which kind of tasks acreep in the role can take.

For each tick, the strategies and directives in colony level state machine will generate a list of required tasks,together with a list of required roles and number of creeps required in each role. After the execution of strategies anddirectives, the colony will summarize the requirement, sort the tasks by the priority, and assign them to free creeps orspawn new creeps if we still have the role quota. In normal situations we won't spawn a new creep in a role if the number ofcurrent living creeps in that role reaches the requirement.

Comparing to the role-based approach, task-based approach is more flexible. Its easier to re-purpose a creep whenit has no other things to do, for example if an outpost defender killed all the enemies in its assigned outpost, it canbe reassigned to defend another outpost. And tasks can somehow be transfered between creeps, for example if a workerfinishes building all construction sites after an RCL growth, it can take the task to upgrade the controller, and once adedicated upgrader is spawned, the worker can give the upgrade task back to the upgrader and take the task to build up ramparts.

The role requirement mechanism is designed to decouple the logic of controlling creep count from the logic to create tasks. Forexample we can predicate the number of remote carriers (aka. haulers in some other bots) precisely when we know the pathlengths from our storage to the outpost sources, so we can limit the remote carriers count while only create remote haultasks when the container near the source actually contain the energy to haul.

Also the role requirement is a good indicator of the spawn availability. Other parts of the bot like the outpostassignment can use this information to help making decisions. The downside of this approach is higher CPU usage. Butthis can be walked-around by caching the result over ticks and only recalculate when needed.
