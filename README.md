# screeps-clarkok-bot

[![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]

This work is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

## What's Screeps

[Screeps](https://screeps.com) is a MMORTS coding game in which you can write AI to operate units called creeps together with afew other structures. It's a sandbox game so everyone can do anything as long as it is not against ToS. To me, the goalis to create a competitive bot with high automation level, to survive and to conquer.

## But there is no code in this repo

The game itself doesn't prevent players from using open source bots developed by others, but overall it is acoding game. So most of fun comes from developing the bot yourself. I'm not going to put my bot open source for others touse it, but instead I'll introduce the structure and the design of it, to help beginners learn quickly, and to helpmyself sort my mind.

## Current status of the clarkok bot

The clarkok bot has gone through a handful of rewrites, the latest one started from Aug 23rd, 2020. In this rewrite, I switched to a generator-based implementation in typescript.

While the clarkok bot is not even close to one of the most advanced bots in the world, it's not that bad. As of writing,[the bot](https://screeps.com/a/#!/profile/clarkok) is currently at #24 in MMO expansion rank, and #70 in the powerrank. It achieved #17 in the past season 1, #11 in season 2 and #8 in season 3.

Currently, the clarkok bot is fully automated in managing all the in-room stuff, and has the functionality to attack andexpand, but the automated decision making for long range operations is not prefect yet.

It has 51k lines of code, 1.31MB after bundling, with a 1.22MB source map.

```

> wc `find ./src -name *.ts` | tail -n 1
51713  123817 1594716 total

```

![line count](https://github.com/clarkok/screeps-clarkok-bot/blob/master/image/line-count.png)

## Content

  * Basic Stuff
    1. [Bot Structure](01-bot-structure.md)
    2. [Task Management](02-task-management.md)
    3. [Room Layout](03-room-layout.md)
    4. [Room Score and Auto Expansion](04-room-score-and-auto-expansion.md)
    5. [Resource Management and Logistics](05-resource-management-and-logistics.md)
    6. [Path and Movement](06-path-and-movement.md)
    7. [CPU Management](07-cpu-management.md)
  * Common Features
    - [planned] Outpost and Remote Mining
    - [planned] Claim New Rooms

## Other resources for this game

 * [Official docs](https://docs.screeps.com/)
 * [Community Wiki](https://wiki.screepspl.us/)

## Disclaimer

 * I'm still learning English, so don't be surprised if you find tons of typos or even grammar issues in the repo 😛
 * Feel free to pick any ideas from here
 
Please @clarkok on Screeps Discord or open an issue for anything.

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
