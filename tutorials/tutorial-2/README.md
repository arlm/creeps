# Introduction

In this Tutorial section we’ll talk about a key strategic object in your room: Room Controller. By controlling this invincible structure you can build facilities in the room. The higher the controller level, the more structures available to build.

# Upgrader

You will need a new worker creep to upgrade your controller level. Let's call it "Upgrader1". In following sections we'll discuss how to create creeps automatically, but for now let's send a command manually to the console.

Spawn a creep with the body `[WORK,CARRY,MOVE]` and the name `Upgrader1`.

Documentation:

* [Control](http://docs.screeps.com/control.html)
* [Game.spawns](http://docs.screeps.com/api/#Game.spawns)
* [StructureSpawn.spawnCreep](http://docs.screeps.com/api/#StructureSpawn.spawnCreep)

## Code

`Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Upgrader1' );`

Creep "Upgrader1" went to perform the same task as the harvester, but we don't want it to. We need to differentiate creep roles.

# Memory

To do that, we need to utilize the `memory` property of each creep that allows writing custom information into the creep's "memory". Let's do this to assign different roles to our creeps.

All your stored memory is accessible via the global Memory object. You can use it any way you like.

Write a property `role='harvester'` into the memory of the harvester creep and `role='upgrader'` — to the upgrader creep with the help of the console.

Documentation:

* [Memory object](http://docs.screeps.com/global-objects#Memory-object)
* [Creep.memory](http://docs.screeps.com/api/#Creep.memory)

## Code

```js
Game.creeps['Harvester1'].memory.role = 'harvester';
Game.creeps['Upgrader1'].memory.role = 'upgrader';
```

You can check your creeps' memory in either the creep information panel on the left or on the "Memory" tab.

#  Defining behaviors

Now let's define the behavior of the new creep. Both creeps should harvest energy, but the creep with the role `harvester` should bring it to the spawn, while the creep with the role `upgrader` should go to the Controller and apply the function `upgradeController` to it (you can get the Controller object with the help of the `Creep.room.controller` property).

In order to do this, we’ll create a new module called `role.upgrader`.

Create a new module `role.upgrader` with the behavior logic of your new creep.

Documentation:

* [RoomObject.room](http://docs.screeps.com/api/#RoomObject.room)
* [Room.controller](http://docs.screeps.com/api/#Room.controller)
* [Creep.upgradeController](http://docs.screeps.com/api/#Creep.upgradeController)

## Code

```js
var roleUpgrader = {

    /** @param {Creep} creep **/
    run: function(creep) {
     if(creep.store[RESOURCE_ENERGY] == 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.upgradeController(creep.room.controller) == ERR_NOT_IN_RANGE) {
                creep.moveTo(creep.room.controller);
            }
        }
    }
};

module.exports = roleUpgrader;
```

Perfect, you have upgraded your Controller level!

**Important**: If you don’t upgrade your Controller within 20,000 game ticks, it loses one level. On reaching level 0, you will lose control over the room, and another player will be able to capture it freely. Make sure that at least one of your creeps regularly performs the function `upgradeController`.
