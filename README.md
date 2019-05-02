### Introduction

This script is a simple example to show how masking can be done using SQL Scripts and Actifio.
This script works with a single database called dmdb.   

smalldb is a discovered and protected database and an on-demand workflow using LiveClone has been created that does a masking prep-mount.

The sequence of events are for a workflow:

1)  A LiveClone of dmdb is refreshed from the lastest snapshot
2)  This LiveClone is prep-mounted to a masking host where dmdb is prep-mounted as a database called prepdmdb.
The workflow needs to ensure the prep-mounted DB has that name or the SQL script will fail.
3)  The workflow calls the launcher.sh as a post mount task (after the prepmount). This starts a second .sh file called masking.sh that runs SQL commands placed in maskscript.sql
If the SQL fails the prep-mount fails.
4)  Once the masking is complete, the prep-mount is unmounted and the liveclone is now in a masked state.
5)  The masked liveClone is now mounted to the target host as maskedsmalldb by the workflow.

This means the workflow needs to be setup as follows:

* Workflow Type:  Liveclone
* Schedule Type:  On Demand
* Mount for Pre-Processing:  On
* Post-Script:  launcher.sh   (or whatever the .sh file is called)
* Create New Virtual Application:  On
* SQL Server Database Name:  Should match the DB name in the masking.sh script  

On the next panel you can set it up any way you like,  if the database is called dmdb, then ideally use this:

* Source DB name:      dmdb
* Prepmount DB name:   prepdmdb 
* Final mount DB name: maskdmdb

### Validation

Compare the masked data between the source DB (dmdb) and the mounted masked copy (maskdmdb).

### Important details to be aware of

1)  The BAT file only runs when these are all set:   "%ACT_MULTI_OPNAME%" == "scrub-mount" if "%ACT_MULTI_END%" == "true" if "%ACT_PHASE%" == "post" 

This ensures the script only runs after all parts of the prep-mount are complete including mounting of logs and starting of prepmounted Database.

### Manual test of bat file

You can run the bat file with a parameter of 'test' to do a manual set of masking.