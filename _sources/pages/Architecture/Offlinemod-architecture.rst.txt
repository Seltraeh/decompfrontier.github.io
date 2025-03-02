Offline mod architecture
================================

As Brave Frontier is a REST application that connects to a Server,
the normal operation flow of the game works as follows:

.. image:: ../../images/serverarch.png

The game simply connects to the game servers hosted by Gumi.

As we don't have the source code of the client (yet), changing
how the game works to be a true offline mod would simply be impossible,
as we don't have a way to remove all the networking code.

Therefore, a different approach is taken, which is the one to create
a Brave Frontier emulator (a Server) which reimplements the official server
in a way that allows the game to work properly.

The server is then injected in the main Brave Frontier application which
contains the specific patches to allow the Server to start and the client
to connect to this server.

As the server is hosted locally on the device, on demands, when the client
is started, the game will not connect to the internet at all to work, therefore
this will achieve an offline mod of some sort.

.. image:: ../../images/offlinemodarch.png

A special exception is made during development of the offline mod
itself, where it would be hard to debug the game client to verify if the
emulator code is correct, therefore, the offline mod is started as a standalone
server in the developer's PC and a special patched version of the game client
is provided that allows the client to connect to this standalone server.

This architecture is no different than the original connection that Brave Frontier
is doing but, some special patches are still implemented to allow the server to be
setted up without big troubles (such as the No SSL patch).

.. image:: ../../images/offlinedevarch.png
