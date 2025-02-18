.. _pbm-commands:

|pbm.app| commands
**********************************************************************



``pbm CLI`` is the command line utility to control the backup system. This page describes |pbm.app| commands available in |PBM|.

For how to get started with |PBM|, see :ref:`initial-setup`.

.. _help:

pbm help
===========

Returns the help information about |pbm.app| commands.

.. _config:

pbm config
==================

Sets, changes or lists |PBM| configuration.

The command has the following syntax:

.. code-block:: bash

   $ pbm config [<flags>] [<key>]

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``--force-resync``
     - Resync backup list with the current storage
   * - ``--list``
     - List current settings
   * - ``--file=FILE`` 
     - Upload the config information from a YAML file
   * - ``--set=SET``
     - Set a new config option value. Specify the option in the <key.name=value> format.
   * - ``-o``, ``--out=text``
     - Shows the output format as either plain text or a JSON object. Supported values: text, json
   
.. admonition:: |PBM| configuration output
   :class: toggle

   .. code-block:: javascript

      {
        "pitr": {
          "enabled": false,
          "oplogSpanMin": 0
        },
        "storage": {
          "type": "filesystem",
          "s3": {
            "region": "",
            "endpointUrl": "",
            "bucket": ""
          },
          "azure": {},
          "filesystem": {
            "path": "<my-backup-dir>"
          }
        },
        "restore": {
          "batchSize": 500,
          "numInsertionWorkers": 10
        },
        "backup": {}
      }

.. admonition:: Setting a config value
   :class: toggle

   .. code-block:: javascript

      [
        {
          "key": "pitr.enabled",
          "value": "true"
        }
      ]

.. _backup:

pbm backup
======================

Creates a backup snapshot and saves it in the remote backup storage. 

The command has the following syntax:

.. code-block:: bash

   $ pbm backup [<flags>]

For more information about using ``pbm backup``, see :ref:`pbm.running.backup.starting` 

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``-t``, ``--type``
     - The type of backup. Supported values: physical, logical (default). When not specified, |PBM| makes a logical backup.
       
       .. note::

          Physical backups feature is the technical preview quality.

   * - ``--compression``
     - Create a backup with compression. 
       Supported compression methods: ``gzip``, ``snappy``, ``lz4``, ``s2``, ``pgzip``, ``zstd``. Default: ``s2``
       The ``none`` value means no compression is done during backup.
   * - ``--compression-level``
     - Configure the compression level from 0 to 10. The default value depends on the compression method used. 
   * - ``-o``, ``--out=text``
     - Shows the output format as either plain text or a JSON object. Supported values: text, json
   * - ``--wait``
     - Wait for the backup to finish. The flag blocks the shell session.


.. admonition:: JSON output
   :class: toggle

   .. code-block:: javascript

      {
        "name": "<backup_name>",
        "storage": "<my-backup-dir>"
      }


.. _restore:

pbm restore
=====================

Restores database from a specified backup / to a specified point in time. Depending on the backup type, makes either logical or physical restore.

The command has the following syntax:

.. code-block:: bash

   $ pbm restore [<flags>] [<backup_name>]

For more information about using ``pbm restore``, see :ref:`pbm.running.backup.restoring`.

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``--time=TIME``
     - Restores the database to the specified point in time. Available for logical restores and if :ref:`PITR` is enabled.
   * - ``-w``
     - Wait for the restore to finish. The flag blocks the shell session.
   * - ``-o``, ``--out=text``
     - Shows the output format as either plain text or a JSON object. Supported values: text, json
   * - ``--base-snapshot``
     - Restores the database from a specified backup to the specified point in time. Without this flag, the most recent backup preceding the timestamp is used for point in recovery. Available in |PBM| starting from version 1.6.0.
   * - ``--replset-remapping`` 
     - Maps the replica set names for the data restore / oplog replay. The value format is ``to_name_1=from_name_1,to_name_2=from_name_2``
 
       
.. admonition:: Restore output
   :class: toggle
    
   .. code-block:: javascript

      {
        "name": "<restore_name>"
        "snapshot": "<backup_name>"
      }
  
.. admonition:: Point-in-time restore
   :class: toggle 

   .. code-block:: javascript

      {
        "name":"<restore_name>",
        "point-in-time":"<backup_name>"
      }

.. _cancel:       

pbm cancel-backup
========================

Cancels a running backup. The backup is marked as canceled in the backup list.

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``-o``, ``--out=text``
     - Shows the output format as either plain text or a JSON object. Supported values: text, json

.. admonition:: JSON output
   :class: toggle

   .. code-block:: javascript

      {
        "msg": "Backup cancellation has started"
      }

.. _list:

pbm list
=================

Provides the list of backups / restores. 

As of version 1.4.0, only successfully completed backups and restores are listed. To view information about running and failed backups, run :ref:`status`.
  
When :ref:`PITR` is enabled, the ``pbm list`` also provides the list of valid time ranges for recovery and point-in-time recovery status. 

In versions 1.3.4 and earlier, the command listed all backups and their states such as:

- In progress - A backup is running
- Canceled - A backup was canceled
- Error - A backup was finished with an error
- No status means a backup is complete

The command has the following syntax:

.. code-block:: bash

   $ pbm list [<flags>]

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``--restore``
     - Shows last N restores. Starting with version 2.0, the output shows restore names instead of backup names, as multiple restores can be done from a single backup. 
   * - ``--size=0``
     - Shows last N backups.
   * - ``-o``, ``--out=text``
     - Shows the output format as either plain text or a JSON object. Supported values: ``text``, ``json``
   * - ``--unbacked``
     - Shows |PITR| oplog slices that were saved without the base backup snapshot. Available starting with version 1.8.0.
   * - ``--replset-remapping`` 
     - Maps the replica set names for the data restore / oplog replay. The value format is ``to_name_1=from_name_1,to_name_2=from_name_2``

.. admonition:: List of backups
   :class: toggle

   .. code-block:: javascript

      {
        "snapshots": [
          {
            "name": "<backup_name>",
            "status": "done",
            "completeTS": Timestamp,
            "pbmVersion": "1.6.0"
          }
        ],
        "pitr": {
          "on": false,
          "ranges": [
            {
              "range": {
                "start": Timestamp,
                "end": Timestamp
              }
            },
            {
              "range": {
                "start": Timestamp,
                "end": Timestamp
              },
            {
              "range": {
                "start": Timestamp,
                "end": Timestamp (no base snapshot)
              }  
            }
          ]
        }
      }

.. admonition:: Restore history
   :class: toggle

   .. code-block:: javascript

      [
        {
          "start": Timestamp,
          "status": "done",
          "type": "snapshot",
          "snapshot": "<backup_name>",
          "name": "<restore_name>"
        },
        {
          "start": Timestamp,
          "status": "done",
          "type": "snapshot",
          "snapshot": "<backup_name>",
          "name": "<restore_name>"
        },
        {
          "start": Timestamp,
          "status": "done",
          "type": "snapshot",
          "snapshot": "<backup_name>",
          "name": "<restore_name>"
        }
      ]


.. _delete:

pbm delete-backup
=======================

Deletes the specified backup snapshot or all backup snapshots that are older than the specified time. The command deletes backups that are not running regardless of the remote backup storage being used.

The following is the command syntax:

.. code-block:: bash

   $ pbm delete-backup [<flags>] [<name>]

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``--older-than=TIMESTAMP``
     - Deletes backups older than date / time specified in the format:
     
       - ``%Y-%M-%DT%H:%M:%S`` (e.g. 2020-04-20T13:13:20) or 
       - ``%Y-%M-%D`` (e.g. 2020-04-20)
   * - ``--force``
     - Forcibly deletes backups without asking for user's confirmation   

.. _delete-pitr:

pbm delete-pitr
=======================

Deletes :term:`oplog slices <Oplog slice>` produced for :ref:`pitr`. 

The command has the following syntax:

.. code-block:: bash

   $ pbm delete-pitr [<flags>] 

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``-a``, ``--all``
     - Deletes all oplog slices
   * - ``--older-than=TIMESTAMP``
     - Deletes oplog slices older than date / time specified in the format:
     
       - ``%Y-%M-%DT%H:%M:%S`` (e.g. 2020-04-20T13:13:20) or 
       - ``%Y-%M-%D`` (e.g. 2020-04-20)
    
       When you specify a timestamp, |PBM| rounds it down to align with the completion time of the closest backup snapshot and deletes oplog slices that precede this time. Thus, extra slices remain. This is done to ensure oplog continuity. To illustrate, the PITR time range is ``2021-08-11T11:16:21 - 2021-08-12T08:55:25`` and backup snapshots are:

       .. code-block:: text

          2021-08-12T08:49:46Z 13.49MB [complete: 2021-08-12T08:50:06]
          2021-08-11T11:36:17Z 7.37MB [complete: 2021-08-11T11:36:38] 

       Say you specify the timestamp ``2021-08-11T19:16:21``. The closest backup is ``2021-08-11T11:36:17Z 7.37KB [complete: 2021-08-11T11:36:38]``. PBM rounds down the timestamp to ``2021-08-11T11:36:38`` and deletes all slices that precede this time. As a result, your PITR time range is ``2021-08-11T11:36:38 - 2021-08-12T09:00:25``.

       .. note::

          |PBM| doesn't delete the oplog slices that follow the most recent backup. This is done to ensure point in time recovery from that backup snapshot. For example, if the snapshot is ``2021-07-20T07:05:23Z [complete: 2021-07-21T07:05:44]`` and you specify the timestamp ``2021-07-20T07:05:45``, |PBM| deletes only slices that were made before ``2021-07-20T07:05:23Z``.

   * - ``--force``
     - Forcibly deletes oplog slices without asking a user's confirmation
   * - ``-o``, ``--out=json``
     - Shows the output as either the plain text (default) or a JSON object. Supported values: ``text``, ``json``.

.. _describe-restore:

pbm describe-restore
====================

Shows the detailed information about the restore.

The command has the following syntax:

.. code-block:: bash

   $ pbm describe-restore [<restore-timestamp>] [<flags>] 


The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``-c``, ``--config=CONFIG``
     - Only for **physical restores**. Points |PBM| to a configuration file so it can read the restore status from the remote storage. For example, ``pbm describe-restore -c /etc/pbm/conf.yaml <restore-name>``
   * - ``-o``, ``--out=text``
     - Shows the output as either the plain text (default) or a JSON object. Supported values: ``text``, ``json``.
 
.. admonition:: "Logical restore status"
   :class: toggle

   {
     "name": "2022-08-15T12:35:00.872256719Z",
     "backup": "2022-08-15T12:34:19Z",
     "type": "logical",
     "status": "done",
     "replsets": [
       {
         "name": "rs1",
         "status": "done",
         "last_transition_ts": 1660566907
       }
     ],
     "opid": "62fa3d7460d0d259449f7061",
     "start_ts": 1660566901,
     "last_transition_ts": 1660566908
   }

.. admonition:: "Physical restore status"
   :class: toggle

   {
     "name": "2022-08-15T11:14:55.683148162Z",
     "backup": "2022-08-15T11:14:32Z",
     "type": "physical",
     "status": "done",
     "replsets": [
       {
         "name": "rs1",
         "status": "done",
         "last_transition_ts": 1660562141,
         "nodes": [
           {
             "name": "127.0.0.1:27017",
             "status": "done",
             "last_transition_ts": 1660562136
           }
         ]
       }
     ],
     "opid": "62fa2aaf6e8356a773a0a357",
     "start_ts": 1660562097,
     "last_transition_ts": 1660562146
   }

.. _version:

pbm version
======================

Shows the version of |PBM|.

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``--short``
     - Shows only version info
   * - ``--commit``
     - Shows only git commit info
   * - ``-o``, ``--out=text``
     - Shows the output as either a plain text or a JSON object. Supported values: ``text``, ``json``
       
.. admonition:: Version information
   :class: toggle

   .. code-block:: javascript

      {
        "Version": "1.6.0",
        "Platform": "linux/amd64",
        "GitCommit": "f9b9948bb8201ba1a6400f6558496934a0685efd",
        "GitBranch": "main",
        "BuildTime": "2021-07-28_15:24_UTC",
        "GoVersion": "go1.16.6"
      }

.. _describe-backup:

pbm describe-backup
===================

Provides the detailed information about a backup:

- backup name
- type
- status
- namespaces - what was backed up during a selective backup.
- size
- error message for failed backup
- last write timestamp 
- cluster information: the replica set name, the backup status on this replica set, whether it is used as a config server replica set, last write timestamp

The command has the following syntax:

.. code-block:: bash

   $ pbm describe-backup [<backup-name>] [<flags>] 

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: auto

   * - Flag
     - Description
   * - ``-o``, ``--out=text``
     - Shows the status as either plain text or a JSON object. Supported values: text, json
       
.. admonition:: "JSON output"
   :class: toggle

   {
     "name": "2022-08-17T10:49:03Z",
     "type": "logical",
     "last_write_ts": {
       "T": 1660733348,
       "I": 2
     },
     "namespaces": [
       "flight.*"
     ],
     "mongodb_version": "5.0.10-9",
     "pbm_version": "2.0.0",
     "status": "done",
     "size": 10234670,
     "error": "",
     "replsets": [
       {
         "name": "rs1",
         "status": "done",
         "iscs": false,
         "last_write_ts": {
           "T": 1660733348,
           "I": 2
         },
         "error": ""
       }
     ]
   }

.. _status:

pbm status
===================

Shows the status of |PBM|. The output provides the following information:

- pbm-agent processes version and state, 
- currently running backups or restores
- backups stored in the remote storage
- |PITR| status
- Valid time ranges for point-in-time recovery and the data size
  
The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: auto

   * - Flag
     - Description
   * - ``-o``, ``--out=text``
     - Shows the status as either plain text or a JSON object. Supported values: text, json
   * - ``-s``, ``--sections=SECTIONS``
     - Shows the status for the specified section. You can pass several flags to view the status for multiple sections. Supported values: cluster, pitr, running, backups. 
   * - ``--replset-remapping`` 
     - Maps the replica set names for the data restore / oplog replay. The value format is ``to_name_1=from_name_1,to_name_2=from_name_2``

   
.. admonition:: Status information
   :class: toggle

   .. code-block:: javascript

      {
        "backups": {
          "type": "FS",
          "path": "<my-backup-dir>",
          "snapshot": [
             ...
            {
              "name": "<backup_name>",
              "size": 3143396168,
              "status": "done",
              "completeTS": Timestamp,
              "pbmVersion": "1.6.0"
            },
          ],
          "pitrChunks": {
            "pitrChunks": [
               ...
              {
                "range": {
                  "start": Timestamp,
                  "end": Timestamp
                }
              },
              {
                "range": {
                  "start": Timestamp,
                  "end": Timestamp (no base snapshot) !!! no backup found
                }
              },
            ],
            "size": 677901884
          }
        },
        "cluster": [
          {
            "rs": "<replSet_name>",
            "nodes": [
              {
                "host": "<replSet_name>/example.mongodb:27017",
                "agent": "v1.6.0",
                "ok": true
              }
            ]
          }
        ],
        "pitr": {
          "conf": true,
          "run": false,
          "error": "Timestamp.000+0000 E [<replSet_name>/example.mongodb:27017] [pitr] <error_message>"
        },
        "running": {
            "type": "backup",
            "name": "<backup_name>",
            "startTS": Timestamp,
            "status": "oplog backup",
            "opID": "6113b631ea9ba5b815fee7c6"
          }
      }


.. _logs:

pbm logs
============

Shows log information from all |pbm-agent| processes. 

The command has the following syntax: 

.. code-block:: bash

   pbm logs [<flags>]

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - Flag
     - Description
   * - ``-t``, ``--tail=20``        
     - Shows last N entries. By default, the output shows last 20 entries. 
       ``0`` means to show all log messages.
   * - ``-e``, ``--event=EVENT``    
     - Shows logs filtered by a specified event. Supported events:

       - backup 
       - restore
       - resyncBcpList
       - pitr 
       - pitrestore
       - delete

   * - ``-o``, ``--out=text``
     - Shows log information as text (default) or in JSON format. 
       Supported values: text, json
   * - ``-n``, ``--node=NODE``
     - Shows logs for a specified node or a replica set. 
       Specify the node in the format ``replset[/host:port]`` 
   * - ``-s``, ``--severity=I``     
     - Shows logs filtered by severity level. 
       Supported levels are (from low to high): D - Debug, I - Info (default), W - Warning, E - Error, F - Fatal.

       The output includes both the specified severity level and all higher ones
   * - ``-i``, ``--opid=OPID``
     - Show logs for an operation in progress. The operation is identified by the :term:`OpID`
   * - ``-x``, ``--extra``
     - Show extra data in the text format

Find the usage examples in :ref:`pbm.logs`.

.. admonition:: Logs output
   :class: toggle

   .. code-block:: javascript

      [
        {
          "t": "",
          "s": 3,
          "rs": "rs0",
          "node": "example.mongodb.com:27017",
          "e": "",
          "eobj": "",
          "ep": {
            "T": 0,
            "I": 0
          },
          "msg": "listening for the commands"
        },
        ....
      ]

.. _pbm.oplog-replay:

pbm oplog-replay
===================

Allows to replay the oplog on top of any backup: logical, physical, storage level snapshot (like EBS-snapshot) and restore it to a specific point in time. 

To learn more about the usage, refer to :ref:`oplog-replay`.

The command has the following syntax: 

.. code-block:: bash

   pbm oplog-replay [<flags>]

The command accepts the following flags:

.. list-table:: 
   :header-rows: 1
   :widths: 30 70

   * - ``start=timestamp``
     - The start time for the oplog replay.
   * - ``end=timestamp``
     - The end time for the oplog replay.
   * - ``--replset-remapping`` 
     - Maps the replica set names for the oplog replay. The value format is ``to_name_1=from_name_1,to_name_2=from_name_2``.

.. include:: .res/replace.txt
