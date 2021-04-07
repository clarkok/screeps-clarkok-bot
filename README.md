# screeps-clarkok-bot

====

## What's Screeps

[Screeps](https://screeps.com) is a MMORTS game in which you can write AI to operate units called creeps together with a
few other structures. It's a sandbox game so everyone can do anything on it as long as it is not against ToS. To me, the
goal is to create a competitive bot with high automation level, to survive and to conquer. 

## But there is no code in this repo

The game itself doesn't prevent gamers from using open source bots developed by other players, but overall its a coding
game. So the most of fun comes from building the bot yourself. So I'm not going to put my bot open source for others to
use it, but instead I'll introduce the structure and the design of it, to help beginners learn quick, and to help myself
sort my mind.

## Current status of clarkok bot

The clarkok bot has been through a handful of rewrites, the latest one started from Aug 23, 2020. In this rewrite, I
switched to a generator-based implementation in typescript.

While the clarkok bot is not even close to one of the most advanced bots in the world, it's not that bad. As of writing,
[it](https://screeps.com/a/#!/profiler/clarkok) currently at #62 in MMO expansion rank, and #70 in the power rank. It
achieved #17 in the past season 1 and it is doing well in the on-going season 2.

Currently, clarkok bot is fully automated in managing in-room stuff, and has the functionality to do attack / expansion,
but the automated decision making for long range operations is not prefect yet. It has 37k lines of code, 418KB after
bundling and minifying, with a 540KB source map.

```

> wc `find ./src -name *.ts` | tail 1
37624   89438 1168501 total

```

## Other resources for this game

 * [Official docs](https://docs.screeps.com/)
 * [Community Wiki](https://wiki.screepssp.us/)

## Content

 1. [Bot Structure](01-bot-structure.md)
 2. [Task Management](02-task-management.md)
 3. [Room Layout](03-room-layout.md)
 4. [Resource Management and Logistics](04-resource-management-and-logistics.md)
 5. [Path and Movement](05-path-and-movement.md)

