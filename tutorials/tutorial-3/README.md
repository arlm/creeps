# Extensions

The Controller upgrade gives access to some new structures: walls, ramparts, and extensions. We’ll discuss walls and ramparts in the next Tutorial section, for now let’s talk about extensions.

**Extensions** are required to build larger creeps. A creep with only one body part of one type works poorly. Giving it several `WORK`s will make him work proportionally faster.

However, such a creep will be costly and a lone spawn can only contain 300 energy units. To build creeps costing over 300 energy units you need spawn extensions.

The second Controller level has **5 extensions** available for you to build. This number increases with each new level.

You can place extensions at any spot in your room, and a spawn can use them regardless of the distance. In this Tutorial we have already placed corresponding construction sites for your convenience.

# Creaing builders

Let’s create a new creep whose purpose is to build structures. This process will be similar to the previous Tutorial sections. But this time let’s set `memory` for the new creep right in the method Spawn.`spawnCreep` by passing it in the third argument.

Spawn a creep with the body `[WORK,CARRY,MOVE]`, the name `Builder1`, and {role:'builder'} as its memory.

Documentation:

* [StructureSpawn.spawnCreep](http://docs.screeps.com/api/#StructureSpawn.spawnCreep)

## Code

```js
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Builder1',
    { memory: { role: 'builder' } } );
```

Our new creep won’t move until we define the behavior for the role `builder`.

# Creating the builder role

As before, let’s move this role into a separate module `role.builder`. The building is carried out by applying the method `Creep.build` to the construction sites searchable by `Room.find(FIND_CONSTRUCTION_SITES)`. The structure requires energy which your creep can harvest on its own.

To avoid having the creep run back and forth too often but make it deplete the cargo, let’s complicate our logic by creating a new Boolean variable `creep.memory.building` which will tell the creep when to switch tasks. We'll also add new creep.say call and `visualizePathStyle` option to the `moveTo` method to visualize the creep's intentions.

Create the module `role.builder` with a behavior logic for a new creep.

Documentation:

* [RoomObject.room](http://docs.screeps.com/api/#RoomObject.room)
* [Room.find](http://docs.screeps.com/api/#Room.find)
* [Creep.build](http://docs.screeps.com/api/#Creep.build)
* [Creep.say](http://docs.screeps.com/api/#Creep.say)

## Code

```js
var roleBuilder = {

    /** @param {Creep} creep **/
    run: function(creep) {

        if(creep.memory.building && creep.store[RESOURCE_ENERGY] == 0) {
            creep.memory.building = false;
            creep.say('🔄 harvest');
        }
        if(!creep.memory.building && creep.store.getFreeCapacity() == 0) {
            creep.memory.building = true;
            creep.say('🚧 build');
        }

        if(creep.memory.building) {
            var targets = creep.room.find(FIND_CONSTRUCTION_SITES);
            if(targets.length) {
                if(creep.build(targets[0]) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
        }
        else {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
     }
    }
};

module.exports = roleBuilder;
```

# Using the builder role

Let’s create a call of the new role in the main module and wait for the result.

By using the module `role.builder` in the new creep, build all 5 extensions.

## Code

```js
var roleHarvester = require('role.harvester');
var roleBuilder = require('role.builder');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'builder') {
            roleBuilder.run(creep);
        }
    }
}
```

Your extensions have been built. Now let’s learn to work with them.

# Using extensions

Maintaining extensions requires you to teach your harvesters to carry energy not just to a spawn but also to extensions. To do this, you can either use the `Game.structures` object or search within the room with the help of `Room.find(FIND_STRUCTURES)`. In both cases, you will need to filter the list of items on the condition `structure.structureType == STRUCTURE_EXTENSION` (or, alternatively, `structure instanceof StructureExtension`) and also check them for energy load, as before.

Refine the logic in the module `role.harvester`.

Documentation:

* [Game.structures](http://docs.screeps.com/api/#Game.structures)
* [Room.find](http://docs.screeps.com/api/#Room.find)
* [StructureExtension](http://docs.screeps.com/api/#StructureExtension)

## Code

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
                        return (structure.structureType == STRUCTURE_EXTENSION || structure.structureType == STRUCTURE_SPAWN) &&
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

# Logging progress

To know the total amount of energy in the room, you can use the property Room.energyAvailable. Let’s add the output of this property into the console in order to track it during the filling of extensions.

Fill all the 5 extensions and the spawn with energy.

## Code

```js
var roleHarvester = require('role.harvester');
var roleBuilder = require('role.builder');

module.exports.loop = function () {

    for(var name in Game.rooms) {
        console.log('Room "'+name+'" has '+Game.rooms[name].energyAvailable+' energy');
    }

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'builder') {
            roleBuilder.run(creep);
        }
    }
}
```

# Creating big harversters

In total, we have 550 energy units in our spawn and extensions. It is enough to build a creep with the body `[WORK,WORK,WORK,WORK,CARRY,MOVE,MOVE]`. This creep will work 4 times faster than a regular worker creep. Its body is heavier, so we’ll add another `MOVE` to it. However, two parts are still not enough to move it at the speed of a small fast creep which would require 4x`MOVE`s or building a road.

Spawn a creep with the body `[WORK,WORK,WORK,CARRY,MOVE,MOVE]`, the name `HarvesterBig`, and `harvester` role.

## Code

```js
Game.spawns['Spawn1'].spawnCreep( [WORK,WORK,WORK,WORK,CARRY,MOVE,MOVE],
    'HarvesterBig',
    { memory: { role: 'harvester' } } );
```

Building this creep took energy from all storages and completely drained them.

Now let’s select our creep and watch it work.

As you can see on the right panel, this powerful creep harvests 8 energy units per tick. A few such creeps can completely drain an energy source before it refills thus giving your colony a maximum energy boost.

Hence, by upgrading your Controller, constructing new extensions and more powerful creeps, you considerably improve the effectiveness of your colony work. Also, by replacing a lot of small creeps with fewer large ones, you save CPU resources on controlling them which is an important prerequisite to play in the online mode.

In the next section, we’ll talk about how to set up the automatic manufacturing of new creeps.
