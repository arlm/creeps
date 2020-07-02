# Introduction

The world of Screeps is not the safest place. Other players may have claims on your territory. Besides, your room may be raided by neutral NPC creeps occasionally. So you ought to think about your colony defense in order to develop it successfully.

Documentation:

* [Defending your room](http://docs.screeps.com/defense.html)

This hostile creep has come from the left entry and attacked your colony. It’s good that we have walls to restrain it temporarily. But they will fall sooner or later, so we need to deal with the problem.

# Safe mode

The surest way to fend off an attack is using the room **Safe Mode**. In safe mode, no other creep will be able to use any harmful methods in the room (but you’ll still be able to defend against strangers).

The safe mode is activated via the room controller which should have activations available to use. Let’s spend one activation to turn it on in our room.

Activate safe mode.

Documentation:

* [StructureController.activateSafeMode](http://docs.screeps.com/api/#StructureController.activateSafeMode)

## Code

```js
Game.spawns['Spawn1'].room.controller.activateSafeMode();
```

As you can see, the enemy creep stopped attacking the wall – its harmful methods are blocked. We recommend that you activate safe mode when your defenses fail.

Now let’s cleanse the room from unwanted guests.

# Building towers

Towers are the easiest way to actively defend a room. They use energy and can be targeted at any creep in a room to attack or heal it. The effect depends on the distance between the tower and the target.

To start with, let’s lay a foundation for our new tower. You can set any place you wish inside the walls and place the construction site there with the help of the button “Construct” on the upper panel.

Place the construction site for the tower (manually or using the code below).

Documentation:

* [StructureTower](http://docs.screeps.com/api/#StructureTower)
* [Room.createConstructionSite](http://docs.screeps.com/api/#Room.createConstructionSite)

## Code

```js
Game.spawns['Spawn1'].room.createConstructionSite( 23, 22, STRUCTURE_TOWER );
```

The creep Builder1 has immediately started the construction. Let’s wait until it finishes.

# Haversting energy for the tower

A tower uses energy, so let’s set the harvester role to bring energy to the tower along with other structures. To do this, you need to add the constant `STRUCTURE_TOWER` to the filter of structures your harvester is aimed at.

Add `STRUCTURE_TOWER` to the module `role.harvester` and wait for the energy to appear in the tower.

## Code

```js
var roleHarvester = {

    /** @param {Creep} creep **/
    run: function(creep) {
        if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
        }
        else {
            var targets = creep.room.find(FIND_STRUCTURES, {
                    filter: (structure) => {
                        return (structure.structureType == STRUCTURE_EXTENSION ||
                                structure.structureType == STRUCTURE_SPAWN ||
                                structure.structureType == STRUCTURE_TOWER) &&
                                structure.store.getFreeCapacity(RESOURCE_ENERGY) > 0;
                    }
            });
            if(targets.length > 0) {
                if(creep.transfer(targets[0], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
        }
    }
};

module.exports = roleHarvester;
```

# Using the tower to destroy enemies

Excellent, your tower is ready to use!

Like a creep, a tower has several similar methods: `attack`, `heal`, and `repair`. Each action spends 10 energy units. We need to use `attack` on the closest enemy creep upon its discovery. Remember that distance is vital: the effect can be several times stronger with the same energy cost!

To get the tower object directly you can use its ID from the right panel and the method `Game.getObjectById`.

Destroy the enemy creep with the help of the tower.

Documentation:

* [Game.getObjectById](http://docs.screeps.com/api/#Game.getObjectById)
* [RoomObject.pos](http://docs.screeps.com/api/#RoomObject.pos)
* [RoomPosition.findClosestByRange](http://docs.screeps.com/api/#RoomPosition.findClosestByRange)
* [StructureTower.attack](http://docs.screeps.com/api/#StructureTower.attack)

## Code

```js
var roleHarvester = require('role.harvester');
var roleUpgrader = require('role.upgrader');
var roleBuilder = require('role.builder');

module.exports.loop = function () {

    var tower = Game.getObjectById('6c95402edc3440c444fabc93');
    if(tower) {
        var closestHostile = tower.pos.findClosestByRange(FIND_HOSTILE_CREEPS);
        if(closestHostile) {
            tower.attack(closestHostile);
        }
    }

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'upgrader') {
            roleUpgrader.run(creep);
        }
        if(creep.memory.role == 'builder') {
            roleBuilder.run(creep);
        }
    }
}
```

The enemy creep is eliminated and our colony can breathe easy. However, the invader has damaged some walls during the brief attack. You’d better set up auto-repair.

# Using the tower to repair structures

Damaged structures can be repaired by both creeps and towers. Let’s try to use a tower for that. We’ll need the method `repair`. You will also need the method `Room.find` and a filter to locate the damaged walls.

Note that since walls don’t belong to any player, finding them requires the constant `FIND_STRUCTURES` rather than `FIND_MY_STRUCTURES`.

Repair all the damaged walls.

Documentation:

* [Room.find]()
* [StructureTower.repair]()

## Code

```js
var roleHarvester = require('role.harvester');
var roleUpgrader = require('role.upgrader');
var roleBuilder = require('role.builder');

module.exports.loop = function () {

    var tower = Game.getObjectById('6c95402edc3440c444fabc93');
    if(tower) {
        var closestDamagedStructure = tower.pos.findClosestByRange(FIND_STRUCTURES, {
            filter: (structure) => structure.hits < structure.hitsMax
        });
        if(closestDamagedStructure) {
            tower.repair(closestDamagedStructure);
        }

        var closestHostile = tower.pos.findClosestByRange(FIND_HOSTILE_CREEPS);
        if(closestHostile) {
            tower.attack(closestHostile);
        }
    }

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'upgrader') {
            roleUpgrader.run(creep);
        }
        if(creep.memory.role == 'builder') {
            roleBuilder.run(creep);
        }
    }
}
```

All the damage from the attack has been repaired!

Congratulations, you have completed the Tutorial! Now you have enough knowledge and code to start playing in the online mode. Choose your room, found a colony, and set out on your own quest for domination in the world of Screeps!

If you want to delve deeper in the subtleties of the game or have any questions, please feel free to refer to:

* [Documentation](http://docs.screeps.com/)
* [Forum](https://screeps.com/forum/)
* [Slack chat](http://chat.screeps.com/)

Have fun scripting!