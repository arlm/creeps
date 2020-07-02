# Introduction

Until now, we have created new creeps directly in the console. It’s not a good idea to do it constantly since the very idea of Screeps is making your colony control itself. You will do well if you teach your spawn to produce creeps in the room on its own.

This is a rather complicated topic and many players spend months perfecting and refining their auto-spawning code. But let’s try at least something simple and master some basic principles to start with.

# Auto-spawning creeps

You will have to create new creeps when old ones die from age or some other reasons. Since there are no events in the game to report death of a particular creep, the easiest way is to just count the number of required creeps, and if it becomes less than a defined value, to start spawning.

There are several ways to count the number of creeps of the required type. One of them is filtering `Game.creeps` with the help of the `_.filter` function and using the role in their memory. Let’s try to do that and bring the number of creeps into the console.

Add the output of the number of creeps with the role `harvester` into the console.

Documentation:

* [Game.creeps](http://docs.screeps.com/api/#Game.creeps)
* [lodash.filter](https://lodash.com/docs#filter)

## Code

```js
var roleHarvester = require('role.harvester');
var roleUpgrader = require('role.upgrader');

module.exports.loop = function () {

    var harvesters = _.filter(Game.creeps, (creep) => creep.memory.role == 'harvester');
    console.log('Harvesters: ' + harvesters.length);

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'upgrader') {
            roleUpgrader.run(creep);
        }
    }
}
```

# Working with two harversters

Let’s say we want to have at least two harvesters at any time. The easiest way to achieve this is to run `StructureSpawn.spawnCreep` each time we discover it’s less than this number. You may not define its name (it will be given automatically in this case), but don’t forget to define the needed role.

We may also add some new `RoomVisual` call in order to visualize what creep is being spawned.

Add the logic for `StructureSpawn.spawnCreep` in your main module.

Documentation:

* [StructureSpawn.spawnCreep](http://docs.screeps.com/api/#StructureSpawn.spawnCreep)
* [RoomVisual](http://docs.screeps.com/api/#RoomVisual)

## Code

```js
var roleHarvester = require('role.harvester');
var roleUpgrader = require('role.upgrader');

module.exports.loop = function () {

    var harvesters = _.filter(Game.creeps, (creep) => creep.memory.role == 'harvester');
    console.log('Harvesters: ' + harvesters.length);

    if(harvesters.length < 2) {
        var newName = 'Harvester' + Game.time;
        console.log('Spawning new harvester: ' + newName);
        Game.spawns['Spawn1'].spawnCreep([WORK,CARRY,MOVE], newName,
            {memory: {role: 'harvester'}});
    }

    if(Game.spawns['Spawn1'].spawning) { 
        var spawningCreep = Game.creeps[Game.spawns['Spawn1'].spawning.name];
        Game.spawns['Spawn1'].room.visual.text(
            '🛠️' + spawningCreep.memory.role,
            Game.spawns['Spawn1'].pos.x + 1,
            Game.spawns['Spawn1'].pos.y,
            {align: 'left', opacity: 0.8});
    }

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'upgrader') {
            roleUpgrader.run(creep);
        }
    }
}
```

# Killing a creep

Now let’s try to emulate a situation when one of our harvesters dies. You can now give the command `suicide` to the creep via the console or its properties panel on the right.

Make one of the harvesters suicide.

Documentation:

* [Creep.suicide](http://docs.screeps.com/api/#Creep.suicide)

## Code

```js
Game.creeps['Harvester1'].suicide()
```

As you can see from the console, after we lacked one harvester, the spawn instantly started building a new one with a new name.

# Releasing memory

An important point here is that the memory of dead creeps is not erased but kept for later reuse. If you create creeps with random names each time it may lead to memory overflow, so you should clear it in the beginning of each tick (prior to the creep creation code).

Add code to clear the memory.

## Code

```js
var roleHarvester = require('role.harvester');
var roleUpgrader = require('role.upgrader');

module.exports.loop = function () {

    for(var name in Memory.creeps) {
        if(!Game.creeps[name]) {
            delete Memory.creeps[name];
            console.log('Clearing non-existing creep memory:', name);
        }
    }

    var harvesters = _.filter(Game.creeps, (creep) => creep.memory.role == 'harvester');
    console.log('Harvesters: ' + harvesters.length);

    if(harvesters.length < 2) {
        var newName = 'Harvester' + Game.time;
        console.log('Spawning new harvester: ' + newName);
        Game.spawns['Spawn1'].spawnCreep([WORK,CARRY,MOVE], newName,
            {memory: {role: 'harvester'}});
    }

    if(Game.spawns['Spawn1'].spawning) { 
        var spawningCreep = Game.creeps[Game.spawns['Spawn1'].spawning.name];
        Game.spawns['Spawn1'].room.visual.text(
            '🛠️' + spawningCreep.memory.role,
            Game.spawns['Spawn1'].pos.x + 1,
            Game.spawns['Spawn1'].pos.y,
            {align: 'left', opacity: 0.8});
    }

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'upgrader') {
            roleUpgrader.run(creep);
        }
    }
}
```

# Renewing creeps

Now the memory of the deceased is relegated to oblivion which saves us resources.

Apart from creating new creeps after the death of old ones, there is another way to maintain the needed number of creeps: the method `StructureSpawn.renewCreep`. Creep aging is disabled in the Tutorial, so we recommend that you familiarize yourself with it on your own.

Documentation:

* [StructureSpawn.renewCreep](http://docs.screeps.com/api/#StructureSpawn.renewCreep)
