Convert script
==============

the convert script (you may not believe it) converts data.

unfortunately microsoft data formats are kind of interesting,
so we need to transform the files to less interesting, but more usable formats.
this is what the convert script does.


the convert script is divided into two parts:
content dependend and independend exports.

Converting media
----------------

* depends on existing original game installation
* converts images, sounds, units and all other data values
  to compatible formats
* creates files for the proprietary data package,
  its files will be used if free media files are missing.

Exporting structures
--------------------

* totally independent of original game installation
* exports original content structures
* prevents redundancy:
  * data structures are needed for media export (to convert them)
  * data structures are needed to import them in the game
  * => structure definition at a single point.
* generation is integrated into make system
* generated files stored in src/gamedata/


data injections
===============

for media files
---------------

each file has an id and a file extension.
files can be "redefined" by overlay archives.

all archives habe to be ordered, so that high priority files override low prio files.

a dict has to be filled with values:
`(fileid, extension, destination) => DRS(archive)`

each insertion call to this dict has to be checked with the extraction rules.

this dict gets updated with the higher priority values.
before performing the update, create the update dict, and also check for
extraction rules at each insertion.

after the file dict is complete, process it, and convert the media files.

For data files
--------------

simply update or delete values in the parsed python data structure.
these modifications will be applied like this:

```python
def fix_data(data):
	#make lists for all units building locations
	for each unit:
		unit.building_loc = [unit.building_loc]

	huskarl_castle.building_loc.append(huskarl_barracks.building_loc)
	delete huskarl_barracks

	#value updating is done like this:
	get_unit(data, "hussar")["hp"] = 9001
	data.terrain.terrains[6].slp_id = 1337

	return data
```

if you have any better idea, propose it...


Dynamic input data
==================

read list of archives etc from config file

read extraction rules from a file given as parameter to script
* this simplifies the management of needed media files
* updates to this file can be made without changing the makefile

create archive profiles
* specify archive names and where to find them
* this enables using mods like "forgotten empires"
* type-specific storage and folder structure
  * define rules for saving specific ids/filesuffixes to specific folders
    * e.g. map "slp" => "graphics/",
      meaning all slps will be stored into that folders
    * map specific filenames,
      e.g. "015015.slp" => "graphics/terrain/deepwater",
   -> this leads to "graphics/terrain/deepwater_015015.png"
* select profiles by existences of "empires2*.dat"
  * if "Games/Forgotten\ Empires/Data/empires2_x1_p1.dat" exists,
    use the forgotten profile
  * if no "Data/empires2_x1_p1.dat" exists, the 1.0c patch is missing
  * => we can determine which version is available,
       by checking which dat files are here
  * ==> we are compatible to age2:aok, age2:aoc, age2:aoc1.0c, age2:tfe