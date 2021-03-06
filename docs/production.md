Persistent distributed storage using BTSync
-------------------------------------------

If you're not running [OrientDB](http://www.orientdb.org) in a distributed configuration you need to take special care to backup your database (in case your host goes down).

Below is a simple, yet hackish, way to do this: using BTSync data containers to propagate the [OrientDB](http://www.orientdb.org) config, LIVE databases and backup folders to remote location(s).
Note: don't trust the remote copy of the LIVE database folder unless the server is down and it has correctly flushed changes to disk.

1. Create BTSync shared folders on any remote location for the various folder you want to replicate

  1.1. config: orientdb configuration inside the config folder

  1.2. databases: the LIVE databases folder

  1.3. backup: the place where [OrientDB](http://www.orientdb.org) will store the zipped backups (if you activate the backup in the configuration file)

2. Take note of the BTSync folder secrets CONFIG_FOLDER_SECRET, DATABASES_FOLDER_SECRET, BACKUP_FOLDER_SECRET

3. Launch BTSync data containers for each of the synched folder you created giving them proper names:
  ```bash
docker run -d --name orientdb_config -v /opt/orientdb/config nesrait/btsync /opt/orientdb/config CONFIG_FOLDER_SECRET
docker run -d --name orientdb_databases -v /opt/orientdb/databases nesrait/btsync /opt/orientdb/databases DATABASES_FOLDER_SECRET
docker run -d --name orientdb_backup -v /opt/orientdb/backup nesrait/btsync /opt/orientdb/backup BACKUP_FOLDER_SECRET
```

4. Wait until all files have magically appeared inside your BTSync data volumes:
  ```bash
docker run --rm -i -t --volumes-from orientdb_config --volumes-from orientdb_databases --volumes-from orientdb_backup ubuntu du -h /opt/orientdb/config /opt/orientdb/databases /opt/orientdb/backup
```

5. Finally you're ready to start your OrientDB server
  ```bash
docker run --name orientdb -d \
            --volumes-from orientdb_config \
            --volumes-from orientdb_databases \
            --volumes-from orientdb_backup \
            -p 2424 -p 2480 \
            orientdb-2.0
```


OrientDB distributed
--------------------

If you're running [OrientDB](http://www.orientdb.org) distributed* you won't have the problem of losing the contents of your databases folder since they are already replicated to the other OrientDB nodes. From the setup above simply leave out the "--volumes-from orientdb_databases" argument and [OrientDB](http://www.orientdb.org) will use the container storage to hold your databases' files.

*note: some extra work might be needed to correctly setup hazelcast running inside docker containers ([see this discussion](https://groups.google.com/forum/#!topic/vertx/MvKcz_aTaWM)).


Ad-hoc backups
--------------

With [OrientDB](http://www.orientdb.org) 2.0 we can now create ad-hoc backups by taking advantage of [the new backup.sh script](https://github.com/orientechnologies/orientdb/wiki/Backup-and-Restore#backup-database):

  - Using the orientdb_backup data container that was created above:
    ```bash
    docker run -i -t --volumes-from orientdb_config --volumes-from orientdb_backup orientdb-2.0 ./backup.sh <dburl> <user> <password> /opt/orientdb/backup/<backup_file> [<type>]
    ```

  - Or using a host folder:

    `docker run -i -t --volumes-from orientdb_config -v <host_backup_path>:/backup orientdb-2.0 ./backup.sh <dburl> <user> <password> /backup/<backup_file> [<type>]`

Either way, when the backup completes you will have the backup file located outside of the [OrientDB](http://www.orientdb.org) container and read for safekeeping.

Note: I haven't tried the non-blocking backup (type=lvm) yet but found [this discussion about a docker LVM dependency issue](https://groups.google.com/forum/#!topic/docker-user/n4Xtvsb4RAw).
