Welcome to the Basho Data Platform.

Overview
----

The Basho Data Platform (BDP) is an extension to Riak. In addition to the offerings of Riak ( distributed, decentralized data storage ), the Data Platform provides the ability to run Spark under the supervision of Riak.

Below, you will find the “quick start” directions for setting up and using Riak. For more information, browse the following files:

* Welcome to the Da
* LICENSE: the license under which Riak and the Data Platform is released
* doc/
 * admin.org: Riak Administration Guide
 * architecture.txt: details about the underlying design of Riak
 * basic-client.txt: slightly more detail on using Riak
 * basic-setup.txt: slightly more detail on setting up Riak
 * man/riak.1.gz: manual page for the riak(1) command
 * man/riak-admin.1.gz manual page for the riak-admin(1) command
 * raw-http-howto.txt: using the Riak HTTP interface

Where to find more
----
 
Below, you’ll find a basic introduction to starting and using Riak as a key/value store. For more information about Riak’s extended feature set, including MapReduce, Search, Secondary Indexes, various storage strategies, and more, please visit our docs here: [Basho Data Platform Docs](http://docs.basho.com/dataplatform/latest/).

Quick Start
----

1. Build Riak+BDP
2. Install Spark
3. Start Riak Nodes/Start Spark

Building the Data Platform
----
* **Note**: the `develop` branch currently only supports Erlang R16B02/B03

Assuming you have a working Erlang ( R16B02/R16B03 ) installation, building Riak+BDP should be as simple as:

```bash
$ cd $DATA_PLATFORM
$ make rel
```
* **Note**: If you prefer to make `dev` instances of BDP, you can substitute make rel with `make devrel`, which will create 8 dev bdp BDP nodes. You can also make individual nodes with `make stagedev[N]` where N is the node number. For example, `make stagedev1` will create a single BDP dev node.

Installing Spark
----

Installing Spark to work with the Basho Data Platform is relatively straight forward.

* Download [Spark](http://www.apache.org/dyn/closer.cgi/spark/spark-1.4.0/spark-1.4.0-bin-hadoop2.6.tgz) -- BDP currently only supports 1.4.0 if you're planning on using the Spark Connector, otherwise BDP is capable of running any version of Spark.

Once you have downloaded Spark, unpack it and copy the contents to your BDP build

```
$ wget http://www.apache.org/dyn/closer.cgi/spark/spark-1.4.0/spark-1.4.0-bin-hadoop2.6.tgz
$ tar -zxvf spark-1.4.0-bin-hadoop2.6.tgz
$ cp -R spark-1.4.0-bin-hadoop2.6 $DATA_PLATFORM/rel/riak/lib/data_platform-1/priv/spark-master
```
* **Note**: If you've made `dev` instances of BDP the target for Spark will be different:

```
$ cp -R spark-1.4.0-bin-hadoop2.6 $DATA_PLATFORM/deps/data_platform/priv/spark-master
``` 

After you have copied Spark to the appropriate location, you will need to copy the *run* support items for Spark. These scripts are located in `$DATA_PLATFORM/deps/data_platform/priv/extras_templates/spark` directory.

```
$ cd $DATA_PLATFORM/rel/riak/lib/data_platform-1/priv/extras_template
$ cp -R spark/ ../spark-master/
```
* **Note**: If you've made `dev` instances of BDP the source and target for Spark will be different:

```
$ cd $DATA_PLATFORM/deps/data_platform/priv/extras_template
$ cp -R spark/ ../spark-master/
```


Starting Basho Data Platform
----

Once you have successfully built BDP, you can start the server with the following commands:

```
$ cd $DATA_PLATFORM/rel/riak
$ bin/riak start
```
* **Note**: The $DATA_PLATFORM/rel/riak directory is a complete, self-contained instance of BDP and Erlang. It is ***strongly*** suggested that you move this directory outside the source tree if you plan to run a production instance.

* **Note++**: If you have gone the `devrel` route from above, BDP is ***not*** built as a stand-alone instance of BDP and Erlang but rather a sym-linked version, where each node is individually configured but `deps` are shared. This is ***not*** suitable for a production release.


#Server Management


Configuration
----
Configuration for the Riak server is stored in the `$RIAK/rel/riak/etc` directory. There are two files:

* vm.args: This file contains the arguments that are passed to the Erlang VM in which Riak runs. The default settings in this file shouldn't need to be changed for most environments.
* app.config: This file contains the configuration for the Erlang applications that run on the Riak server.

Control
----

The Basho Data Platform has two aspects of control to it: Riak and the BDP Service Manager.

###Riak

####bin/riak

This script is the primary interface for starting and stopping the Riak server.

To start a daemonized ( background ) instance of Riak:

```
$ bin/riak start
```

Once a server is running in the background you can attach to the Erlang console via:

```
$ bin/riak attach
```

Alternatively, if you want to run a foreground instance of Riak, start it with:

```
$ bin/riak console
```

Stopping a foreground or background instance of Riak can be done from a shell prompt via:

```
$ bin/riak stop
```
Or if you are attached/on the Erlang console:

```
(riak@127.0.0.1)1> q().
```

You can determine if the server is running by:

```
$ bin/riak ping
```

####bin/riak-admin

This script provides access to general administration of the Riak server. The below commands assume you are running a default configuration for parameters such as cookie.

To join a new Riak node to an existing cluster:

```
$ bin/riak start # If a local server is not already running
$ bin/riak-admin join <node in cluster>
```
* **Note** You must have a local node running for this work.

To verify that the local Riak node is able to read/write data:

```
$ bin/riak-admin test
```

To backup a node or cluster run the following:

```
$ bin/riak-admin backup riak@X.X.X.X <directory/backup_file> node
$ bin/riak-admin backup riak@X.X.X.X <directory/backup_file> all
```
Restores can function in two ways:

1. If the backup file was of a node, then only the node will be restored.
2. If the backup file contains data for a cluster, all nodes in the cluster will be restored.

To restore a backup file:

```
$ bin/riak-admin restore riak@X.X.X.X riak <directory/backup_file>
```

To view the status of a node:

```
$ bin/riak-admin status
```

If you change the IP or node name, you will need to use the reip command:

```
$ bin/riak-admin reip <old_nodename> <new_nodename>
```

####bin/data-platform-admin

This script provides access to BDP specific administration of the Riak server. The below commands assume that you have installed spark per directions above and are running a default configuration for parameters such as cookie.

To join a new BDP node to an existing cluster:

```
$ bin/riak start # If a local server is not already running
$ bin/data-platform-admin join <node in cluster>
```
* **Note** You must have a local node running for this to work.

To prepare the Riak cluster to work with spark:

```
$ bin/riak-admin bucket-type create strong '{"props":{"consistent":true}}'
$ bin/riak-admin bucket-type create maps '{"props":{"datatype":"map"}}'
$ bin/riak-admin bucket-type activate maps
```
To ensure that the map bucket type creation was successful:

```
$ bin/riak-admin bucket-type status maps
```

To add a new service configuration to a BDP cluster:

```
$ bin/data-platform-admin add-service-config <config-name> <service-type> <configuration>
```

Sample Spark service configuration:

```
$ bin/data-platform-admin add-service-config my-spark-master spark-master RIAK_HOSTS="RIAK_IP_1:RIAK_PB_PORT,RIAK_IP_2:RIAK_PB_PORT"
```
* **Note** `RIAK_IP_1:RIAK_PB_PORT` correspond to your Riak cluster. 

To start a service with BDP, using an existing configuration:

```
$ bin/data-platform-admin start-service <node in cluster> <group name> <service configuration>
```
* **Note** Group name is any valid string to describe your service group.

Continuing the example above, to start Spark:

```
$ bin/data-platform-admin start-service <node in cluster> my-spark-group my-spark-master
```

To stop a service running in a BDP cluster:

```
$ bin/data-platform-admin stop-service <node in cluster> <group name> <service>
```

To view available services within a BDP cluster:

```
$ bin/data-platform-admin services
```

To view services running on a specific BDP cluster node:

```
$ bin/data-platform-admin node-services <node in cluster>
```

To view all nodes in a BDP cluster running a specific service:

```
$ bin/data-platform-admin service-nodes <service type>
```


Contributing to Riak and the Basho Data Platform and Reporting Bugs
----

Basho encourages contributions to Riak from the community. Here's how to get started:

* For the appropriate sub-projects that are affected by your change. For this repository if your changes are for release generation or packaging.
* Make your changes and run the test suite. ( See below )
* Commit your changes and push them to your fork.
* Open a pull-request for the appropriate projects.
* Basho engineers will review your pull-request, suggest changes ( if neccessary ) and merge it when it's ready and/or offer feedback.

To report a bug or issue, please open a [new issue](https://github.com/basho/data_platform/issues) against this repository.

You can read the full guidelines for bug reporting and code constributions on the Riak Docs.


Testing
----
To make sure that your patch works, be sure to run the test suite in each modified sub-project and dialyzer from the top-level project to detect static code errors.

To run the QuickCheck properties included in Riak sub-projects, download [QuickCheck Mini from Quviq](http://quviq.com/index.html).

* **Note** Some properties that require features of the full version of QuickCheck will fail.

####Running unit tests

The unit tests for each subproject can be run with `make` or `rebar` like so:

```
make eunit
```
```
./rebar skip_deps=true eunit
```

####Running dialyzer

Dialyzer performs statis analysis of the code to discover defects, edge-cases and discrepancies between type specifications and the actual implementation.

Dialyzer requires a pre-built code analysis table called a PLT. Building a PLT is expensive and can take up to 30 minutes on some machines. Once built, you generally want to avoid clearning or rebuilding the PLT unless you have had significant changes in your buil ( a new version of Erlang, for example ).

#####Build the PLT

Here's the command to build the PLT:

```
make build_plt
```

#####Check the PLT

If you have build the PLT before, check it before you run Dialyzer again. This will take much less time than building the PLT from scratch:

```
make check_plt
```

#####Run Dialyzer

```
make dialyzer
```
