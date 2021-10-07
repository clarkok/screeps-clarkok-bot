# CPU Management

----

Nothing is infinite. In the early game the spawn time would be the limitation, in the mid game the energy and in the late game, it is the CPU. Without any control, as you are adding more and more features in your bot, the CPU will eventually be insufficient no matter how you optimize the code. 

The CPU management in the clarkok bot can be split into three parts: 1) CPU planning, 2) different behavior upon different CPU situations and 3) emergency recovery. 

## CPU Planning

Features like combat and the remote mining can predicate precisely how many intents they will consume in each tick, or at least provide a upper bound. For the combat the upper bound of the intents can be counted by the number of creeps × the actions they can take in the single tick, while for the remote mining we can sum up the average intent consumption for every tasks involved, like mine, haul, reserve and etc.

But the number of the intents itself it not enough to predicate the overall CPU usage, there will always be some other computational cost around intents. In the clarkok bot, the intents only take up ~50% of the overall CPU. To weight this computational cost in, we introduce a global metric of average CPU consumption per intent, and thus that metric in the clarkok bot is around 0.4 as of writing.

With this metric, we can then use a total intent count as a cap to plan our CPU usage. And the plan will happen every 1500 ticks, to align with the creep lifetime. Then the next gen of the creeps will follow the plan.

## Behavior Changes

Some features can behave differently upon different CPU situations. To name a few:

- remote mining will use [bucket chain](https://wiki.screepspl.us/index.php/Energy#Bucket_Chain) when hauling, to trade spawn time with the cost of higher CPU
- market handling will consider the energy cost and adjust the price accordingly when choosing the best order if there is exceeding CPU
- colonies will choose creeps over links to haul energy around if there are free creeps and CPU to save the 3% link cost

## Emergency Recovery

Plans can't always keep up with the changes. A long path finding or a complex calculation can happen from time to time and take a lot of CPU, causing not all of the creeps can finish their task in a tick. Or something changes for a remote outpost and there are still 1000+ ticks before another planing, causing the average CPU per intent increase. Even we have the bucket mechanism to buffer the CPU a bit, they can't hold long especially on shards with higher CPU limit.

In order to prevent catastrophic failures, and help the bot recover from the worst situation, we built a few safe guards. First of all, the bot will stop everything if the bucket is below 500, the tickLimit. It mostly happen at the very beginning of a round, the clarkok bot will pause a few ticks before it can actually do anything.

Then we have 2 layers of throttling, the soft one and the hard one, that apply to the task level state machines. The soft one keeps the execution time under the tickLimit, so we don't see the error saying the execution goes out of time. And the hard one can kick in when the bucket is low, to help maintain a relatively healthy bucket level.

The execution sequence of the clarkok bot is split into roughly 5 stage for each tick:

1. Colony level state machines -- to issue the tasks for house keeping and for each directive
2. Task level state machines -- those are the generator based threads
3. Assign resource requirements for paused tasks
4. Task level state machines part 2
5. Federal state machines -- which may have heavy computation

Those throttling comes into play at the stage 2 and the stage 4. If the `Game.cpu.getUsed` and the bucket meet the situation, the hard throttling will immediately stop all the task level state machines, while the soft one will allow those high priority tasks, like the combat, and skip those not-so-important tasks.
