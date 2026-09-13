# NB Scripting

[svg](https://github.com/nulls-mods-community/scripting-docs#nb-scripting)

#### Telegram Channel: [@danyanull](https://t.me/danyanull)

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%BA%D0%B0%D0%BD%D0%B0%D0%BB-%D0%B2-%D1%82%D0%B3-danyanull)

NB Scripting is a technology that allows you to create and use custom scripts in Null’s Brawl battles.

## How does it work?

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%BA%D0%B0%D0%BA-%D1%8D%D1%82%D0%BE-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D0%B5%D1%82)

When each battle starts, the server checks whether custom scripts have been assigned to it. If so, they are loaded.

Loaded scripts only have access to their own battle. For more information about how we isolate scripts, see the [Security](https://github.com/nulls-mods-community/scripting-docs#%D0%B1%D0%B5%D0%B7%D0%BE%D0%BF%D0%B0%D1%81%D0%BD%D0%BE%D1%81%D1%82%D1%8C) section.

Each script must have a global `tick()` function, which is called every game tick.

Scripts can also subscribe to various events, such as when a specific character takes damage.

When the battle ends, the scripts are destroyed.

## Programming Language

[svg](https://github.com/nulls-mods-community/scripting-docs#%D1%8F%D0%B7%D1%8B%D0%BA-%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F)

Scripts are written in Lua. The primary interpreter currently used is [Luau](https://luau.org/).

In some respects, Luau's behavior may differ from what is expected according to the Lua standard. Read more about this here: https://luau.org/compatibility/.

## Available API

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%BD%D0%BE%D0%B5-api)

It is important to note that our battle server is written in Java. Therefore, our API is built around working with Java objects.

Each of these objects is represented in Lua as *userdata*, rather than a *table*. You cannot create these objects yourself — you can only obtain them through our API.

Each Java object belongs to a class, which determines the fields and methods available to it. The complete list of classes is available in a separate document: [API Reference](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md).

Every script has access to an object of the [Server](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#server) class. This object exists as a single instance and serves as the main entry point to our API. It is accessible through the global variable **server**.

### Basic Operations with Java Objects

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%BE%D1%81%D0%BD%D0%BE%D0%B2%D0%BD%D1%8B%D0%B5-%D0%B4%D0%B5%D0%B9%D1%81%D1%82%D0%B2%D0%B8%D1%8F-%D1%81-java-%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D0%B0%D0%BC%D0%B8)

You can get and set fields in the same way as with a regular Lua table:

```lua
clientInfo.ultiCharge = math.min(server.tick, clientInfo.maxUltiCharge)
```

**svg**

Keep in mind that if a field is marked as readonly, attempting to modify it will not throw an error. However, its value will not be changed.

You can call available methods using the colon operator:

```lua
clientInfo = server:getClientInfo(0)
```

**svg**

For objects of the `Iterable` class and its subclasses (`List`, `ArrayList`), you can use a `for` loop:

```lua
for i, character in server.objectManager:getCharacters() do
    ...
end
```

**svg**

### Static Fields of Java Classes

[svg](https://github.com/nulls-mods-community/scripting-docs#%D1%81%D1%82%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B5-%D0%BF%D0%BE%D0%BB%D1%8F-java-%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D0%BE%D0%B2)

Some Java classes can have **static** fields and methods. They differ from regular ones in that their behavior is not tied to any specific object. In most cases, such fields are used for constants, while methods are used for utility functions.

For each Java class, a separate Lua *table* is created and made available as a global variable with the same name. For example, `AttackOrigin` or `LogicCharacter`. This table is not a class; instead, it contains a **copy** of its static fields and methods.

This is especially relevant for enumeration classes (*enums*), such as [AttackOrigin](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#attackorigin) or [GameMode](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#attackorigin). Objects of these classes are always constant and are stored as their static fields.

Example:

```lua
if server.gameMode ~= GameMode.GEM_GRAB then
    character:takeHeal(character.index, 4000, true, nil, AttackOrigin.WEAPON)
end
```

**svg**

Note that even if a class has no accessible static fields or methods, a table will still be created for it and will simply be empty.

### Utility Functions

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%B2%D1%81%D0%BF%D0%BE%D0%BC%D0%BE%D0%B3%D0%B0%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D0%B8)

Our API provides several utility functions. They do not directly interact with the server's Java code, but they help you create or access certain objects.

| **Name**       | **Type**                                  | **Usage**                                                                                |
| -------------- | ----------------------------------------- | ---------------------------------------------------------------------------------------- |
| lookup         | `(type: number, name: string) ⇒ Data`     | Allows you to obtain a Data object by its type and name.                                 |
| createObject   | `(data: Data) ⇒ LogicGameObject`          | Allows you to create a new LogicGameObject instance for a specific Data object.          |
| createCallback | `(class: string, arg: function) ⇒ Object` | Allows you to convert a Lua function into a Java object. Used for subscribing to events. |
| enumAsTable    | `(class: string) ⇒ table`                 | **Deprecated.** Kept only for backward compatibility.                                    |
| log            | `(params: string...) ⇒ void`              | Prints a message to the debug logs and to the friendly-room chat, if applicable.         |

These functions are available globally.

### Lua Standard Library

[svg](https://github.com/nulls-mods-community/scripting-docs#%D1%81%D1%82%D0%B0%D0%BD%D0%B4%D0%B0%D1%80%D1%82%D0%BD%D0%B0%D1%8F-%D0%B1%D0%B8%D0%B1%D0%BB%D0%B8%D0%BE%D1%82%D0%B5%D0%BA%D0%B0-lua)

The Luau interpreter's standard library is available: https://luau.org/library/.

### Working with JSON

[svg](https://github.com/nulls-mods-community/scripting-docs#%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0-%D1%81-json)

A library for working with JSON is globally available to every script: https://github.com/rxi/json.lua.

Example:

```lua
local values = {}
values.key = "value"
log(json.encode(values))
```

**svg**

### Inheritance and Polymorphism

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%BD%D0%B0%D1%81%D0%BB%D0%B5%D0%B4%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B8-%D0%BF%D0%BE%D0%BB%D0%B8%D0%BC%D0%BE%D1%80%D1%84%D0%B8%D0%B7%D0%BC)

Some Java classes can be abstract. Such classes cannot have instances. They are used to generalize multiple subclasses into a single type with common fields or methods.

For example, [LogicGameObject](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#logicgameobject) is an abstract class. When you call `createObject()` or `objectManager:getObject()`, you will actually receive an instance of one of its subclasses: [LogicCharacter](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#logiccharacter), [LogicItem](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#logicitem), [LogicProjectile](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#logicprojectile), or [LogicAreaEffect](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#logicareaeffect).

This works in both directions: `objectManager:addObject()` can also accept any of these subclasses as an argument.

Furthermore, this works not only with abstract classes. In Java, you can always pass or return an object of a child class even when an object of the parent class is expected. However, there are currently no examples of such classes in the available API.

It is also worth discussing `Iterable`, `List`, and `ArrayList`. `ArrayList` is a subclass of the abstract `List` class, which in turn inherits from the abstract `Iterable` class. Within the available API, no other subclasses of `List` or `Iterable` are used — therefore, if you see `Iterable` or `List`, it will always be an `ArrayList`.

### Subscribing to Events

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%BF%D0%BE%D0%B4%D0%BF%D0%B8%D1%81%D0%BA%D0%B0-%D0%BD%D0%B0-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D1%8F)

Let's look at [LogicCharacter](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#logiccharacter) and its **takingDamageListeners** field. This is a list of subscriptions to the damage event.

This list is represented by the `ArrayList` class. In other words, it is a Java object that internally stores other Java objects. We cannot simply add an arbitrary Lua function to it directly.

That is why `createCallback()` exists. Its usage looks like this:

```lua
local callback = function(target, projectile, damage, someData, origin)
    ...
end

local callbackImpl = createCallback("DamageEventListener", callback)
character.takingDamageListeners:add(callbackImpl)
```

**svg**

Using this function, we converted a Lua function into a Java object of the `DamageEventListener` class, which can then be added to the list. Amazing!

Note that if you call `createCallback()` twice with the same function, you will get two different objects. Keep this in mind if you want to check an event subscription using `takingDamageListeners:indexOf()`.

The following classes are currently supported:

| **Name**            | **Type**                                                                                                                     |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| DamageEventListener | `(source: LogicCharacter,<br>projectile: LogicProjectile,<br>damage: number,<br>data: Data,<br>origin: AttackOrigin) ⇒ void` |
| SkillEventListener  | `(skill: Skill) ⇒ void`                                                                                                      |
| BasicEventListener  | `() ⇒ void`                                                                                                                  |

## Security

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%B1%D0%B5%D0%B7%D0%BE%D0%BF%D0%B0%D1%81%D0%BD%D0%BE%D1%81%D1%82%D1%8C)

Each individual script runs in its own isolated sandbox. It has no access to system APIs. You can only work with the objects provided by the NB Scripting API, as well as certain safe functions from the standard library.

Each script has a memory and CPU Time quota. A script may be unloaded at any time if its quota is exceeded.

| **Resource** | **Base Quota**                                                           |
| ------------ | ------------------------------------------------------------------------ |
| CPU Time     | No more than 15 milliseconds of real time per Lua code execution.        |
| Memory       | No more than 16 MiB for the entire script. Java objects are not counted. |

## Interesting Facts

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%B5%D1%81%D0%BD%D1%8B%D0%B5-%D1%84%D0%B0%D0%BA%D1%82%D1%8B)

In Brawl Stars, game logic simulation takes place on the server at a rate of 20 ticks per second.

### Main Tables Containing Game Data

[svg](https://github.com/nulls-mods-community/scripting-docs#%D0%BE%D1%81%D0%BD%D0%BE%D0%B2%D0%BD%D1%8B%D0%B5-%D1%82%D0%B0%D0%B1%D0%BB%D0%B8%D1%86%D1%8B-%D1%81-%D0%B8%D0%B3%D1%80%D0%BE%D0%B2%D1%8B%D0%BC%D0%B8-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D0%BC%D0%B8)

| **Type** | **File(s)**                                           | **Class**                                                                                                          |
| -------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| 6        | `projectiles_skin.csv`, `projectiles_logic.csv`       | [ProjectileData](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#projectiledata)     |
| 15       | `locations.csv`                                       | [LocationData](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#locationdata)         |
| 16       | `characters.csv`                                      | [CharacterData](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#characterdata)       |
| 17       | `area_effects_skin.csv`, `area_effects_logic.csv`     | [AreaEffectData](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#areaeffectdata)     |
| 18       | `items.csv`                                           | [ItemData](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#itemdata)                 |
| 20       | `skills.csv`                                          | [SkillData](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#skilldata)               |
| 23       | `cards.csv`                                           |                                                                                                                    |
| 27       | `tiles.csv`                                           | [TileData](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#tiledata)                 |
| 29       | `skins.csv`                                           | [SkinData](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#skindata)                 |
| 52       | `emotes.csv`                                          |                                                                                                                    |
| 68       | `sprays.csv`                                          |                                                                                                                    |
| 108      | `traits.csv`                                          |                                                                                                                    |
| 117      | `status_effects_skin.csv`, `status_effects_logic.csv` | [StatusEffectData](https://github.com/nulls-mods-community/scripting-docs/blob/main/reference.md#statuseffectdata) |
