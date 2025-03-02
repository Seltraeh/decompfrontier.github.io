TODO
===============================

What's missing for a possible release of the offline mod? (The order doesn't mean that something has more priority than other)


Code
-----------------------------

.. task-list::
    :custom:

    + [ ] Is it really a good architecture? I'm afraid there's a lot more that can be borrowed as a lession from Spring/ASP.NET
    + [ ] HIGH PRIORITY: Remove all the current MST/Packet classes and switch to an external generator like with Python. This can help with external tools and having an unified packet/mst definition for stuff like decomp as well. (Parsing the responses with a decomp is a big redundant work)
    + [ ] Implement actual info saving routines, nothing is really saved or connected atm
    + [x] :del:`Find a way to avoid restarting the app to run the migrations, possibly run the migrations before the server boots up (seems like a Drogon skill issue by me)` Fixed in https://github.com/decompfrontier/server/commit/0069e4e99dad9052ceb5e86bd771f967f724f7ca
    + [ ] Implement proper DLL entrypoint (Java ok: https://github.com/decompfrontier/client/blob/main/proj.android/app/src/main/java/it/arves100/gimuserver/OfflineMod.java, Win32 proxy ok: https://github.com/decompfrontier/offline-proxy/commit/d1b9c70d6d3414b7f44d69496613e4ce7cbac223)

Unit managment
-----------------------------

.. task-list::
    :custom:

    + [  ] Decode the MST properly and read it, store it somewhere (probably a local cache like SQL?)
    + [  ] Unit equipment (Sphere)
    + [  ] Unit favorite
    + [  ] Unit Skill point
    + [  ] Unit fusion
    + [  ] Unit selling
    + [  ] Team managment
    + [  ] Unit evolution
    + [  ] Unit Omni evolution
    + [  ] Imps
    + [  ] Frogs
    + [  ] Rex units
    + [  ] Elgifs

Items
-----------------------------

.. task-list::
    :custom:

    + [  ] Decode the MST properly and read it
    + [  ] Item selling
    + [  ] Use in dungeon
    + [  ] Use before dungeon

Town
-----------------------------

.. task-list::
    :custom:

    + [  ] Properly make/parse all the town levelup info
    + [  ] Actually show the town buttons
    + [  ] Bazaar exchanges
    + [  ] Levelup
    + [  ] Item crafting
    + [  ] Sphere crafting
    + [  ] Drop

Quests
-----------------------------

.. task-list::
    :custom:

    + [  ] Properly decide a format for all missions and stuff ( https://github.com/cheahjs/bravefrontier_data/tree/master/missions_parsed can help? )
    + [  ] Decode and document everything for quests/missions/AI
    + [  ] Start one mission (during the login tutorial test)
    + [  ] Do one zone of the mission (Grand Gaia -> Mistral)
    + [  ] Friends/support unit pickup?

General
-----------------------------

.. task-list::
    :custom:

    + [  ] Decode a lot of unknown stuff in the user information
    + [  ] Tutorial
    + [  ] Account creation
    + [  ] Save the user data somewhere in sqlite
    + [  ] Android libgame.so patches
    + [  ] iOS libgame.so patches?

Website
-----------------------------

.. task-list::
    :custom:

    + [  ] Document the currently written MSTs/responses from the client decomp
    + [  ] Explain the Game server GMEE
    + [  ] Explain the Game server feature list (propably document it first)
    + [  ] Begin to document the Unit stuff (refeer to the beta documentation picture)

Social
-----------------------------

.. task-list::
    :custom:

    + [  ] Gift receive
    + [  ] Gift pickup
    + [  ] Item list to give

General QoL (not priorities but can definitly help people to discover and document)
---------------------------------------------------------------------------------------

.. task-list::
    :custom:

    + [  ] Minimal (?) HTML Admin tool to modify all the parameters of unit/unitteam/player
    + [  ] External tool to modify all the JSONs in deploy/ directory and BF MSTs

To define/think about them
-----------------------------

.. task-list::
    :custom:

    + [  ] Social features (friends, arena, colosseum should be done?)
    + [  ] Grand quests
    + [  ] Randal capital (crafting, achievements, keys)
    + [  ] Raid
    + [  ] Colosseum
    + [  ] Arena
    + [  ] Challenge Arena maybe? Some code is still there...
    + [  ] Chapter 2 support
    + [  ] Chapter 3 support (Summoner unit, etc etc)
    + [  ] Mystery box
    + [  ] Daily mission (a format has to be decided for how they work)
    + [  ] Daily login roulette
    + [  ] Medals
