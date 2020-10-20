Big Chungus 
===========

Big Chungus is a single 6TB RAM node residing in ARCUS-HTC (moved from ARCUS-B circa Jan 2020). Access is given by Simon Benjamin on a case-by-case basis.

## Hardware




## Access

Big Chungus lives in `arcus-htc`, accessible from an gateway ARC network `oscgate.arc.ox.ac.uk`. Access from outside of the Oxford University network may first require VPN to server `vpn.ox.ac.uk` (using `oxfordusername@ox.ac.uk`). 

Access using your regular ARC account `[USER]` by two tunnels:

```bash
ssh [USER]@oscgate.arc.ox.ac.uk
```
then
```bash
ssh [USER]@arcus-htc
```

This is the level from which jobs should be submitted. 

> Scheduled jobs will remain in queue unless the user has been granted access to Big Chungus. 

Big Chungus can also be SSH'd into directly, though this should be avoided in general, and done only if the node has first been reserved via SLURM. 
Check if Big Chungus is in use via:
```bash
eh
```
After submitting a job (see below), SSH directly into Big Chungus (from within `arcus-htc`) via:
```bash
cnode4101 ??
```

## Job Submission