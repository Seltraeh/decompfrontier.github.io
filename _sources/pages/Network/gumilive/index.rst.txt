Gumi Live
=================================

The Gumi Live API server is a server called in Brave Frontier
before the connection to the actual game server.
This server is the first to be called by the game (during the login screen).

This server is responsable for the following things:

*    The initial dynamic BF background
*    Version checking during startup (and optionally showing maintenance)
*    Guest account login
*    Facebook account login
*    Telling the game which account server and game server to connect

There are, currently known, 2 different Gumi Live API endpoints, which might also
be found in different Gumi games than Brave Frontier.

The first API known as the "Live API", it was used in BF up to an unknown
version, the endpoint ended with live.gumi.sg, this api supported two different
versions, one inside /api/1.0/ the other in /api/1.1/.

The second API known as the "SL API", was used in BF up to the latest client version, the
endpoint ended with api-sl.gl.gumi.sg. (DLS is also part of this API)

This document currently covers documentation of the SL API only.

.. toctree::
    :caption: Gumi Live
    :name: toc-network-gumilive
    :maxdepth: 1

    SLDLS
    SLAPI
