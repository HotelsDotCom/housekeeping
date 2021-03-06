# Housekeeping Vacuum Tool

# Overview

The Housekeeping Vacuum Tool looks for any files and folders in the data locations of a Hive table that are not referenced in either the Hive metastore or the Housekeeping database. Any paths discovered are considered "orphaned data" and are then scheduled for removal via the Housekeeping process. The tool is typically used if previously orphaned path locations have been lost (which can happen if the Housekeeping database has become corrupt or a previous Housekeeping run did not finish gracefully due to some unexpected failure or being killed prematurely etc.) By running the Vacuum Tool against a Hive table one can identify data that is no longer referenced by Hive and is thus a candidate for deletion.  

## Install

### Pre-requisites

The Vacuum Tool makes use of various Hadoop and Hive libraries and client executables so these must be installed on the machine where the tool will be run. It does not require a Hadoop cluster, a Hadoop client "jump box" will suffice.

#### EMR 

If you are planning to run the Vacuum Tool on EMR you may need to set up the EMR classpath by exporting the following environment variables before calling the `bin/vacuum.sh` script:

```bash
export HCAT_LIB=/usr/lib/hive-hcatalog/share/hcatalog/
export HIVE_LIB=/usr/lib/hive/lib/
```

Note that the paths above are correct as of when this document was last updated but may differ across EMR versions, refer to the [EMR release guide](http://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-release-components.html) for more up-to-date information if necessary.

### Unpack and set up the Vacuum Tool

[Download the TGZ](https://repository.sonatype.org/service/local/artifact/maven/redirect?r=central-proxy&g=com.hotels&a=housekeeping-vacuum-tool&p=tgz&v=RELEASE&c=bin) from Maven central and then uncompress the file by executing:

```bash
tar -xzf housekeeping-vacuum-tool-<version>-bin.tgz
```

Although it's not necessary, we recommend exporting the environment variable `HOUSEKEEPING_TOOL_HOME` by setting its value to wherever you extracted it to:

```bash
export HOUSEKEEPING_TOOL_HOME=/<foo>/<bar>/housekeeping-vacuum-tool-<version>
```

## Usage

**Note:** _All updates to the table being vacuumed must be paused for the duration of the vacuum process otherwise there is a risk that folders that have been newly created but not yet added to the metastore will be considered candidates for housekeeping._

Run with your respective replication YAML configuration file:

```bash
$HOUSEKEEPING_TOOL_HOME/bin/vacuum.sh \
  --config=<your-config>.yml \
  [--dry-run=true] \
  [--partition-batch-size=1000]
```

The `dry-run` option allows you to observe the status of paths on the file system, the metastore, and the Housekeeping database without actually scheduling anything for deletion. The `partition-batch-size` is the number of partitions to retrieve in each batch from a table, and can be changed to a lower number if an out of memory exception occurs.

## YAML Configuration

|Property|Required|Description|
|:----|:----:|:----|
|`catalog.name`|Yes|A name for the source catalog for events and logging.|
|`catalog.hive-metastore-uris`|Yes|Fully qualified URI of the source cluster's Hive metastore Thrift service. This property mimics the Hive property `hive.metastore.uris` and allows multiple comma separated URIs.|
|`catalog.site-xml`|No|A list of Hadoop configuration XML files to add to the configuration for the source.|
|`catalog.configuration-properties`|No|A list of `key: value` pairs to add to the Hadoop configuration for the source.|
|`catalog.metastore-tunnel.route`|No|An SSH tunnel can be used to connect to source metastores. The tunnel may consist of one or more hops which must be declared in this property.|
|`catalog.metastore-tunnel.private-keys`|No|A comma-separated list of paths to any SSH keys required in order to set up the SSH tunnel.|
|`catalog.metastore-tunnel.known-hosts`|No|Path to a known hosts file.|
|`catalog.metastore-tunnel.port`|No|The port on which SSH runs on the source master node. Default is `22`.|
|`catalog.metastore-tunnel.local-host`|No|The address on which to bind the local end of the tunnel. Default is `localhost`.|
|`tables.database-name`|Yes|The Hive database name for the table the vacuum tool will interrogate.|
|`tables.table-name`|Yes| The Hive table name for the table the vacuum tool will interrogate.|
|`housekeeping.schema-name`|No|The schema name that is used in your housekeeping instance. Defaults to `housekeeping`.|
|`housekeeping.h2.database`|No|If the `housekeeping.data-source.url` is not overridden then the default H2 database can be configured using this property which also controls where H2 will write its database files. Defaults to `${instance.home}/data/${instance.name}/${housekeeping.schema-name}` (where `instance.home`, `instance.name` and `housekeeping.schema-name` can be configured separately for more fine-grained control).|
|`housekeeping.data-source.driver-class-name` |No|The fully qualified class name of your database driver. Defaults to the H2 driver if not configured.|
|`housekeeping.data-source.url` |No| JDBC URL for your database. Defaults to H2 filesystem database if not specified. |
|`housekeeping.data-source.username` |No| Username for your database.|
|`housekeeping.data-source.password` |No| Password for your database.|
|`housekeeping.db-init-script`|No|A file containing a script to initialise your schema can be provided if it does not already exist.|
|`tables-validation.hive-table-properties`|No| A list of Hive table properties that need to exist in every configured table. If any of these properties do not exist then the vacuum tool won't run. Set this to a custom property or an empty list if you vacuum tables that are not replicated by [Circus Train](https://github.com/HotelsDotCom/circus-train). We always recommend running with `--dry-run=true` first and carefully reviewing the results before doing a "real" vacuum. Default is `com.hotels.bdp.circustrain.replication.event`.|

### Example YAML Configurations

#### Vacuum Tool configured with MySQL Housekeeping database

In order to use an external JDBC-compliant database, the JDBC driver for this database must be made available on the CLASSPATH of the vacuum tool. 
This can be achieved by one of the following:
* Adding the path to the driver jar file to the Housekeeping bootstrap CLASSPATH (e.g. `export HOUSEKEEPING_CLASSPATH=/path/to/mysql-connector-java-x.y.z.jar`). 
* Placing the driver jar file in `$VACUUM_TOOL_HOME/lib/`.

The configuration then needs to be updated to be something like below:

```yaml
catalog:
  name: vacuum_tool
  hive-metastore-uris: thrift://my-metastore-uri:9083

tables:
- database-name: db
  table-name: table_1

housekeeping:
  schema-name: my_db
  dataSource:
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://db-host:3306/${housekeeping.schema-name}
    username: user
    password: foo
```

Note: To use MySQL and similar database systems, the schema specified in the configuration needs to exist, as the value for `housekeeping.data-source.url` needs to be a valid URI. 

#### Vacuum Tool configured with H2 Housekeeping database

The Vacuum tool already has the required H2 drivers on its CLASSPATH so the only change required to use H2 is to create a configuration file similar to below:

```yaml
catalog:
  name: vacuum_tool
  hive-metastore-uris: thrift://my-metastore-uri:9083

tables:
- database-name: db
  table-name: table_1

housekeeping:
  schema-name: my_db
  db-init-script: file:///tmp/schema.sql
  h2:
      # Location of H2 DB on filesystem
      database: /home/hadoop/vacuumtest/data/${housekeeping.schema-name}
  dataSource:
      username: user
      password: foo
```
