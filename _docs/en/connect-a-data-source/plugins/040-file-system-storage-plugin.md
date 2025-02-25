---
title: "File System Storage Plugin"
slug: "File System Storage Plugin"
parent: "Connect a Data Source"
---
You can register a storage plugin configuration that connects Drill to a local file system or to a distributed file system registered in the Hadoop `core-site.xml`, such as S3
or HDFS. By
default, Apache Drill includes a storage plugin configuration named `dfs` that points to the local file
system on your machine by default.

## Connecting Drill to a File System

In a Drill cluster, you typically do not query the local file system, but instead place files on the distributed file system. Currently, you need to use a distributed file system when connecting multiple Drillbits to get complete, consistent query results, simulate a distributed file system by copying files to each node, or use an NFS volume, such as Amazon Elastic File System.

You configure the connection property of the storage plugin workspace to connect Drill to a distributed file system. For example, the following connection property connects Drill to an HDFS cluster from a client:

`"connection": "hdfs://<IP Address>:<Port>/"`

To query a file on HDFS from a node on the cluster, you can simply change the connection from `file:///` to `hdfs://` in the `dfs` storage plugin.

To change the `dfs` storage plugin configuration to point to a different local or a distributed file system, use `connection` attributes as shown in the following examples.

* Local file system example:

  ```
  {
    "type": "file",
    "enabled": true,
    "connection": "file:///",
    "workspaces": {
      "root": {
        "location": "/user/max/donuts",
        "writable": false,
        "defaultInputFormat": null
       }
    },
    "formats" : {
      "json" : {
        "type" : "json"
      }
    }
  }
  ```

Note that a local file system configuration can be based on the mount point of a network file system that has been mounted at the same location in the local file systems of each Drillbit.

* Distributed file system example:

  ```
  {
    "type" : "file",
    "enabled" : true,
    "connection" : "hdfs://10.10.30.156:8020/",
    "workspaces" : {
      "root" : {
        "location" : "/user/root/drill",
        "writable" : true,
        "defaultInputFormat" : null
      }
    },
    "formats" : {
      "json" : {
        "type" : "json"
      }
    }
  }
  ```

To connect to a Hadoop file system, you include the IP address and port number (service port) of the
name node. If you are not sure then you can use "hdfs getconf -confKey fs.default.name" command on hdfs to get the name node details.

### Querying Donuts Example

The following example shows a file type storage plugin configuration with a
workspace named `json_files`. The configuration points Drill to the
`/users/max/drill/json/` directory in the local file system `(dfs)`:

    {
      "type" : "file",
      "enabled" : true,
      "connection" : "file:///",
      "workspaces" : {
        "json_files" : {
          "location" : "/users/max/drill/json/",
          "writable" : false,
          "defaultInputFormat" : json
       }
    },

The `connection` parameter in this configuration is "`file:///`", connecting Drill to the local file system.

To query a file in the example `json_files` workspace, you can issue the `USE`
command to tell Drill to use the `json_files` workspace, which is included in the `dfs`
configuration for each query that you issue:

    USE dfs.json_files;
    SELECT * FROM `donuts.json` WHERE type='frosted'

If the `json_files` workspace did not exist, the query would have to include the
full file path name to the `donuts.json` file:

    SELECT * FROM dfs.`/users/max/drill/json/donuts.json` WHERE type='frosted';

## Mount and unmount commands

**Introduced in release: 1.21**

drill.exec.storage.file.enable_mount_commands

## Configuration options in core-site.xml

### drill.exec.recursive_file_listing_max_size

Use this property to set a limit on the numer of files that Drill will list by recursing into a DFS directory tree. When the limit is encountered the initiating operation will fail with an error. The intended application of this limit is to allow admins to protect their Drillbits from an errant or malicious `SELECT * FROM dfs.huge_workspace LIMIT 10` query, which will cause an OOM given a big enough workspace of files. Defaults to 0 which means no limit.
