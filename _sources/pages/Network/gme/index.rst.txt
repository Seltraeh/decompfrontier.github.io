Game Server
===================

The game server (gme) is the main server which all the actions of Brave Frontier are done.

The original server was written in PHP (according to a leak of one misconfigured file) as a REST api, 
each key of the JSON are not written in plaintext, rather they seems to be an hash of the field name,
this is presumably done to complex the analysis of the packets.



.. toctree::
    :caption: Game Server
    :name: toc-network-gme
    :maxdepth: 1

