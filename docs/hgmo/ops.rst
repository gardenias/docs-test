.. _hgmo_ops:

=================
Operational Guide
=================

Deploying to hg.mozilla.org
===========================

All code running on the servers behind hg.mozilla.org is in the
version-control-tools repository.

Deployment of new code to hg.mozilla.org is performed via an Ansible
playbook defined in the version-control-tools repository. To deploy new
code, simply run::

   $ ./deploy hgmo

.. important::

   Files from your local machine's version-control-tools working copy
   will be installed on servers. Be aware of any local changes you have
   performed, as they might be reflected on the server.

For minor deployments, pre-announcement of changes is not necessary: just do
the deployment. As part of the deployment, an IRC notification will be issued
in #vcs by Ansible.

For major upgrades (upgrading the Mercurial release or other major changes
such as reconfiguring SSH settings or other changes that have a higher chance
of fallout, pre-announcement is highly recommended.

Pre-announcements should be made to
`dev-version-control <mailto:dev-version-control@lists.mozilla.org>`_.

Deployment-time announcements should be made in ``#vcs``. In addition, the
on-duty Sheriff (they will have ``|Sheriffduty`` appended to their IRC nick)
should be notified. Anyone in ``#releng`` with ``|buildduty`` in their IRC
nick should also be notified. Sending an email to ``sheriffs@mozilla.org``
can't also hurt.

If extra caution is warranted, a bug should be filed against the Change Advisory
Board. This board will help you schedule the upgrade work. Details can be found
at https://wiki.mozilla.org/IT/ChangeControl.

Deployment Gotchas
------------------

Not all processes are restarted as part of upgrades. Notably, the ``httpd`` +
WSGI process trees are not restarted. This means that Python or Mercurial
upgrades may not been seen until the next ``httpd`` service restart. For this
reason, deployments of Mercurial upgrades should be followed by manually
restarting ``httpd`` when each server is out of the load balancer.

Restarting httpd/wsgi Processes
-------------------------------

.. note:: this should be codified in an Ansible playbook

If a restart of the ``httpd`` and WSGI process tree is needed, perform the
following:

1. Log in to the Zeus load balancer at https://zlb1.ops.scl3.mozilla.com:9090
2. Find the ``hgweb-http`` pool.
3. Mark as host as ``draining`` then click ``Update``.
4. Poll the *draining* host for active connections by SSHing into the host
   and curling ``http://localhost/server-status?auto``. If you see more than
   1 active connection (the connection performing server-status), repeat until
   it goes away.
5. ``service httpd restart``
6. Put the host back in service in Zeus.
7. Repeat 3 to 6 until done with all hosts.

Forcing a hgweb Repository Re-clone
===================================

It is sometimes desirable to force a re-clone of a repository to each
hgweb node. For example, if a new Mercurial release offers newer
features in the repository store, you may need to perform a fresh clone
in order to *upgrade* the repository on-disk format.

To perform a re-clone of a repository on hgweb nodes, the
``hgmo-reclone-repos`` deploy target can be used::

   $ ./deploy hgmo-reclone-repos mozilla-central releases/mozilla-beta

The underlying Ansible playbook integrates with the load balancers and
will automatically take machines out of service and wait for active
connections to be served before performing a re-clone. The re-clone
should thus complete without any client-perceived downtime.

Repository Mirroring
====================

The replication/mirroring of repositories is initiated on the master/SSH
server. An event is written into a distributed replication log and it is
replayed on all available mirrors. See :ref:`hgmo_replication` for more.

Most repository interactions result in replication occurring automatically.
In a few scenarios, you'll need to manually trigger replication.

The ``vcsreplicator`` Mercurial extension defines commands for creating
replication messages. For a full list of available commands run
``hg help -r vcsreplicator``. The following commands are common.

hg replicatehgrc
   Replicate the hgrc file for the repository. The ``.hg/hgrc`` file will
   effectively be copied to mirrors verbatim.

hg replicatesync
   Force mirrors to synchronize against the master. This ensures the repo
   is present on mirrors, the hgrc is in sync, and all repository data from
   the master is present.

   Run this if mirrors ever get out of sync with the master. It should be
   harmless to run this on any repo at any time.

.. important::

   You will need to run ``/var/hg/venv_tools/bin/hg`` instead of
   ``/usr/bin/hg`` so Python package dependencies required for
   replication are loaded.

Creating New Review Repositories
================================

In order to conduct code review in MozReview, a special review repository
must be configured.

Creating new review repositories is simple::

  $ ./deploy mozreview-create-repo

Then, simply enter requested data in the prompts and the review repository
should be created.

.. note::

   This requires root SSH access to reviewboard-hg1.dmz.scl.mozilla.com
   and for the specified Bugzilla account to have admin privileges on
   reviewboard.mozilla.org.

Marking Repositories as Read-only
=================================

Repositories can be marked as read-only. When a repository is read-only,
pushes are denied with a message saying the repository is read-only.

To mark an individual repository as read-only, create a
``.hg/readonlyreason`` file. If the file has content, it will be printed
to the user as the reason the repository is read-only.

To mark all repositories on hg.mozilla.org as read-only, create the
``/etc/mercurial/readonlyreason`` file. If the file has content, it will
be printed to the user.

Retiring Repositories
=====================

Users can :ref:`delete their own repositories <hgmo_delete_user_repo>` - this section applies only to
non-user repositories.

Convention is to retire (aka delete) repositories by moving them out of
the user accessible spaces on the master and deleting from webheads.

This can be done via ansible playbook in the version-control-tools
repository::

  $ cd ansible
  $ ansible-playbook -i hosts -e repo=relative/path/on/server hgmo-retire-repo.yml

.. _hgmo_ops_monitoring:

SSH Server Services
===================

This section describes relevant services running on the SSH servers. There
is a single master server at any given time and a hot standby ready to be
promoted to master should the master go down.

hg-master.target
----------------

This systemd target provides a common target for starting and stopping
all systemd units that should only be running on the active master server.

The unit only starts if the ``/repo/hg/master.<hostname>`` file is present.
e.g. ``hgssh1.dmz.scl3.mozilla.com`` will only start the target if
``/repo/hg/master.hgss1.dmz.scl3.mozilla.com`` is present.

sshd_hg.service
---------------

This systemd service provides the SSH server for accepting external SSH
connections that connect to Mercurial.

This is different from the system's SSH service (``sshd.service``). The
differences from a typical SSH service are as follows:

* The service is running on port 222 (not port 22)
* SSH authorized keys are looked up in LDAP (not using the system auth)
* All logins are processed via ``pash``, a custom Python script that
  dispatches to Mercurial or performs other adminstrative tasks.

This service should always be running on all servers, even if they aren't
the master.

hg-bundle-generate.timer and hg-bundle-generate.service
-------------------------------------------------------

These systemd units are responsible for creating Mercurial bundles for
popular repositories and uploading them to S3. The bundles it produces
are also available on a CDN at https://hg.cdn.mozilla.net/.

These bundles are advertised by Mercurial repositories to facilitate
:ref:`bundle-based cloning <hgmo_bundleclone>`, which drastically reduces
the load on the hg.mozilla.org servers.

This service only runs on the master server.

pushdataaggregator.service
--------------------------

This systemd service monitors the state of the replication mirrors and
copies fully acknowledged/applied messages into a new Kafka topic
(``replicatedpushdata``).

The ``replicatedpushdata`` topic is watched by other services to react to
repository events. So if this service stops working, other services
will likely sit idle.

This service only runs on the master server.

``pulsenotifier.service``
-------------------------

This systemd service monitors the ``replicatedpushdata`` Kafka topic
and sends messages to Pulse to advertise repository events.

For more, see :ref:`hgmo_notification`.

The Pulse notifications this service sends are relied upon by various
applications at Mozilla. If it stops working, a lot of services don't
get notifications and things stop working.

This service only runs on the master server.

``unifyrepo.service``
---------------------

This systemd service periodically aggregates the contents of various
repositories into other repositories.

This service and the repositories it writes to are currently experimental.

This service only runs on the master server.

Monitoring and Alerts
=====================

hg.mozilla.org is monitored by Nagios.

check_zookeeper
---------------

check_zookeeper monitors the health of the ZooKeeper ensemble running on
various servers. The check is installed on each server running
ZooKeeper.

The check verifies 2 distinct things: the health of an individual ZooKeeper
node and the overall health of the ZooKeeper ensemble (cluster of nodes).
Both types of checks should be configured where this check is running.

Expected Output
^^^^^^^^^^^^^^^

When everything is functioning as intended, the output of this check
should be::

   zookeeper node and ensemble OK

Failures of Individual Nodes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A series of checks will be performed against the individual ZooKeeper
node. The following error conditions are possible:

NODE CRITICAL - not responding "imok": <response>
   The check sent a ``ruok`` request to ZooKeeper and the server failed to
   respond with ``imok``. This typically means the node is in some kind of
   failure state.

NODE CRITICAL - not in read/write mode: <mode>
   The check sent a ``isro`` request to ZooKeeper and the server did not
   respond with ``rw``. This means the server is not accepting writes. This
   typically means the node is in some kind of failure state.

NODE WARNING - average latency higher than expected: <got> > <expected>
   The average latency to service requests since last query is higher than
   the configured limit. This node is possibly under higher-than-expected
   load.

NODE WARNING - open file descriptors above percentage limit: <value>
   The underlying Java process is close to running out of available file
   descriptors.

   We should never see this alert in production.

If any of these node errors is seen, ``#vcs`` should be notified and the
on call person for these servers should be notified.

Failures of Overall Ensemble
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A series of checks is performed against the ZooKeeper ensemble to check for
overall health. These checks are installed on each server running ZooKeeper
even though the check is seemingly redundant. The reason is each server may
have a different perspective on ensemble state due to things like network
partitions. It is therefore important for each server to perform the check
from its own perspective.

The following error conditions are possible:

ENSEMBLE WARNING - node (HOST) not OK: <state>
   A node in the ZooKeeper ensemble is not returning ``imok`` to an ``ruok``
   request.

   As long as this only occurs on a single node at a time, the overall
   availability of the ZooKeeper ensemble is not compromised: things should
   continue to work without service operation. If the operation of the
   ensemble is compromised, a different error condition with a critical
   failure should be raised.

ENSEMBLE WARNING - socket error connecting to HOST: <error>
   We were unable to speak to a host in the ensemble.

   This error can occur if ZooKeeper is not running on a node it should be
   running on.

   As long as this only occurs on a single node at a time, the overall
   availability of the ZooKeeper ensemble is not compromised.

ENSEMBLE WARNING - node (HOST) is alive but not available
   A ZooKeeper server is running but it isn't healthy.

   This likely only occurs when the ZooKeeper ensemble is not fully available.

ENSEMBLE CRITICAL - unable to find leader node; ensemble likely not writable
   We were unable to identify a leader node in the ZooKeeper ensemble.

   This error almost certainly means the ZooKeeper ensemble is down.

ENSEMBLE WARNING - only have X/Y expected followers
   This warning occurs when one or more nodes in the ZooKeeper ensemble
   isn't present and following the leader node.

   As long as we still have a quorum of nodes in sync with the leader,
   the overall state of the ensemble should not be compromised.

ENSEMBLE WARNING - only have X/Y in sync followers
   This warning occurs when one or more nodes in the ZooKeeper ensemble
   isn't in sync with the leader node.

   This warning likely occurs after a node was restarted or experienced some
   kind of event that caused it to get out of sync.

check_vcsreplicator_lag
-----------------------

``check_vcsreplicator_lag`` monitors the replication log to see if
consumers are in sync.

This check runs on every host that runs the replication log consumer
daemon, which is every *hgweb* machine. The check is only monitoring the
state of the host it runs on.

The replication log consists of N independent partitions. Each partition
is its own log of replication events. There exist N daemon processes
on each consumer host. Each daemon process consumes a specific partition.
Events for any given repository are always routed to the same partition.

Consumers maintain an offset into the replication log marking how many
messages they've consumed. When there are more messages in the log than
the consumer has marked as applied, the log is said to be *lagging*. A
lagging consumer is measured by the count of messages it has failed to
consume and by the elapsed time since the first unconsumed message was
created. Time is the more important lag indicator because the replication
log can contain many small messages that apply instantaneously and thus
don't really constitute a notable lag.

When the replication system is working correctly, messages written by
producers are consumed within milliseconds on consumers. However, some
messages may take several seconds to apply. Consumers do not mark a message
as consumed until it has successfully applied it. Therefore, there is
always a window between event production and marking it as consumed where
consumers are out of sync.

Expected Output
^^^^^^^^^^^^^^^

When a host is fully in sync with the replication log, the check will
output the following::

   OK - 8/8 consumers completely in sync

   OK - partition 0 is completely in sync (X/Y)
   OK - partition 1 is completely in sync (W/Z)
   ...

This prints the count of partitions in the replication log and the
consuming offset of each partition.

When a host has some partitions that are slightly out of sync with the
replication log, we get a slightly different output::

   OK - 2/8 consumers out of sync but within tolerances

   OK - partition 0 is 1 messages behind (0/1)
   OK - partition 0 is 1.232 seconds behind
   OK - partition 1 is completely in sync (32/32)
   ...

Even though consumers are slightly behind replaying the replication log,
the drift is within tolerances, so the check is reporting OK. However,
the state of each partition's lag is printed for forensic purposes.

Warning and Critical Output
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The monitor alerts when the lag of any one partition of the replication
log is too great. As mentioned above, lag is measured in message count
and time since the first unconsumed message was created. Time is the more
important lag indicator.

When a partition/consumer is too far behind, the monitor will issue a
**WARNING** or **CRITICAL** alert depending on how far behind consumers
are. The output will look like::

   WARNING - 2/8 partitions out of sync

   WARNING - partition 0 is 15 messages behind (10/25)
   OK - partition 0 is 5.421 seconds behind
   OK - partition 1 is completely in sync (34/34)
   ...

The first line will contain a summary of all partitions' sync status. The
following lines will print per-partition state.

The check will also emit a warning when there appears to be clock drift
between the producer and the consumer.::

   WARNING - 0/8 partitions out of sync
   OK - partition 0 is completely in sync (25/25)
   WARNING - clock drift of -1.234s between producer and consumer
   OK - partition 1 is completely in sync (34/34)
   ...

Remediation to Consumer Lag
^^^^^^^^^^^^^^^^^^^^^^^^^^^

If everything is functioning properly, a lagging consumer will self
correct on its own: the consumer daemon is just behind (due to high
load, slow network, etc) and it will catch up over time.

In some rare scenarios, there may be a bug in the consumer daemon that
has caused it to crash or enter a endless loop or some such. To check
for this, first look at systemd to see if all the consumer daemons
are running::

   $ systemctl status vcsreplicator@*.service

If any of the processes aren't in the ``active (running)`` state, the
consumer for that partition has crashed for some reason. Try to start it
back up:

   $ systemctl start vcsreplicator@*.service

You might want to take a look at the logs in the journal to make sure the
process is happy:

   $ journalctl -f --unit vcsreplicator@*.service

If there are errors starting the consumer process (including if the
consumer process keeps restarting due to crashing applying the next
available message), then we've encountered a scenario that will
require a bit more human involvement.

.. important::

   At this point, it might be a good idea to ping people in #vcs or
   page Developer Services on Call, as they are the domain experts.

If the consumer daemon is stuck in an endless loop trying to apply
the replication log, there are generally two ways out:

1. Fix the condition causing the endless loop.
2. Skip the message.

We don't yet know of correctable conditions causing endless loops. So,
for now the best we can do is skip the message and hope the condition
doesn't come back::

   $ /var/hg/venv_replication/bin/vcsreplicator-consumer /etc/mercurial/vcsreplicator.ini --skip

.. important::

   Skipping messages could result in the repository replication state
   getting out of whack.

   If this only occurred on a single machine, consider taking the
   machine out of the load balancer until the incident is investigated
   by someone in #vcs.

   If this occurred globally, please raise awareness ASAP.

.. important::

   If you skip a message, please file a bug in
   `Developer Services :: hg.mozilla.org <https://bugzilla.mozilla.org/enter_bug.cgi?product=Developer%20Services&component=Mercurial%3A%20hg.mozilla.org>`_
   with details of the incident so the root cause can be tracked down
   and the underlying bug fixed.

check_pushdataaggregator_lag
----------------------------

``check_pushdataaggregator_lag`` monitors the lag of the aggregated replication
log (the ``pushdataaggregator.service`` systemd service).

The check verifies that the aggregator service has copied all fully
replicated messages to the unified, aggregate Kafka topic.

The check will alert if the number of outstanding ready-to-copy messages
exceeds configured thresholds.

.. important::

   If messages aren't being copied into the aggregated message log, derived
   services such as Pulse notification won't be writing data.

Expected Output
^^^^^^^^^^^^^^^

Normal output will say that all messages have been copied and all partitions
are in sync or within thresholds::

   OK - aggregator has copied all fully replicated messages

   OK - partition 0 is completely in sync (1/1)
   OK - partition 1 is completely in sync (1/1)
   OK - partition 2 is completely in sync (1/1)
   OK - partition 3 is completely in sync (1/1)
   OK - partition 4 is completely in sync (1/1)
   OK - partition 5 is completely in sync (1/1)
   OK - partition 6 is completely in sync (1/1)
   OK - partition 7 is completely in sync (1/1)

Failure Output
^^^^^^^^^^^^^^

The check will print a summary line indicating total number of messages
behind and a per-partition breakdown of where that lag is. e.g.::

   CRITICAL - 2 messages from 2 partitions behind

   CRITICAL - partition 0 is 1 messages behind (1/2)
   OK - partition 1 is completely in sync (1/1)
   CRITICAL - partition 2 is 1 messages behind (1/2)
   OK - partition 3 is completely in sync (1/1)
   OK - partition 4 is completely in sync (1/1)
   OK - partition 5 is completely in sync (1/1)
   OK - partition 6 is completely in sync (1/1)
   OK - partition 7 is completely in sync (1/1)

   See https://mozilla-version-control-tools.readthedocs.io/en/latest/hgmo/ops.html
   for details about this check.

Remediation to Check Failure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the check is failing, first verify the Kafka cluster is operating as
expected. If it isn't, other alerts on the hg machines should be firing.
**Failures in this check can likely be ignored if the Kafka cluster is in
a known bad state.**

If there are no other alerts, there is a chance the daemon process has
become wedged. Try bouncing the daemon::

   $ systemctl restart pushdataaggregator.service

Then wait a few minutes to see if the lag decreased. You can also look at
the journal to see what the daemon is doing::

   $ journalctl -f --unit pushdataaggregator.service

If things are failing, escalate to VCS on call.

check_pulsenotifier_lag
-----------------------

``check_pulsenotifier_lag`` monitors the lag of Pulse
:ref:`hgmo_notification` in reaction to server events.

The check is very similar to ``check_vcsreplicator_lag``. It monitors the
same class of thing under the hood: that a Kafka consumer has read and
acknowledged all available messages.

For this check, the consumer daemon is the ``pulsenotifier`` service running
on the master server. It is a systemd service (``pulsenotifier.service``). Its
logs are in ``/var/log/pulsenotifier.log``.

Expected Output
^^^^^^^^^^^^^^^

There is a single consumer and partition for the pulse notifier Kafka
consumer. So, expected output is something like the following::

   OK - 1/1 consumers completely in sync

   OK - partition 0 is completely in sync (159580/159580)

   See https://mozilla-version-control-tools.readthedocs.io/en/latest/hgmo/ops.html
   for details about this check.

Remediation to Check Failure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are 3 main categories of check failure:

1. pulse.mozilla.org is down
2. The ``pulsenotifier`` daemon has crashed or wedged
3. The hg.mozilla.org Kafka cluster is down

Looking at the last few lines of ``/var/log/pulsenotifier.log`` should
indicate reasons for the check failure.

If Pulse is down, the check should be acked until Pulse service is restored.
The Pulse notification daemon should recover on its own.

If the ``pulsenotifier`` daemon has crashed, try restarting it::

   $ systemctl restart pulsenotifier.service

If the hg.mozilla.org Kafka cluster is down, lots of other alerts are
likely firing. You should alert VCS on call.

Adding/Removing Nodes from Zookeeper and Kafka
==============================================

When new servers are added or removed, the Zookeeper and Kafka clusters
may need to be *rebalanced*. This typically only happens when servers
are replaced.

The process is complicated and requires a number of manual steps. It
shouldn't be performed frequently enough to justify automating it.

Adding a new server to Zookeeper and Kafka
------------------------------------------

The first step is to assign a Zookeeper ID in Ansible. See
https://hg.mozilla.org/hgcustom/version-control-tools/rev/da8687458cd1
for an example commit. Find the next available integer **that hasn't been
used before**. This is typically ``N+1`` where ``N`` is the last entry
in that file.

.. note::

   Assigning a Zookeeper ID has the side-effect of enabling Zookeeper
   and Kafka on the server. On the next deploy, Zookeeper and Kafka
   will be installed.

Deploy this change via ``./deploy hgmo``.

During the deploy, some Nagios alerts may fire saying the Zookeeper
ensemble is missing followers. e.g.::

   hg is WARNING: ENSEMBLE WARNING - only have 4/5 expected followers

This is because as the deploy is performed, we're adding references to
the new Zookeeper server before it is actually started. These warnings
should be safe to ignore.

Once the deploy finishes, start Zookeeper on the new server::

   $ systemctl start zookeeper.service

Nagios alerts for the Zookeeper ensemble should clear after Zookeeper
has started on the new server.

Wait a minute or so then start Kafka on the new server::

   $ systemctl start kafka.service

At this point, Zookeeper and Kafka are both running and part of their
respective clusters. Everything is in a mostly stable state at this
point.

Rebalancing Kafka Data to the New Server
----------------------------------------

When the new Kafka node comes online, it will be part of the Kafka
cluster but it won't have any data. In other words, it won't
really be used (unless a cluster event such as creation of a new
topic causes data to be assigned to it).

To have the new server actually do something, we'll need to run
some Kafka tools to rebalance data.

The tool used to rebalance data is
``/opt/kafka/bin/kafka-reassign-partitions.sh``. It has 3 modes of operation,
all of which we'll use:

1. Generate a reassignment plan
2. Execute a reassignment plan
3. Verify reassignments have completed

All command invocations require a ``--zookeeper`` argument defining
the Zookeeper servers to connect to. The value for this argument should
be the ``zookeeper.connect`` variable from ``/etc/kafka/server.properties``.
e.g. ``hgssh4.dmz.scl3.mozilla.com:2181/hgmoreplication,hgweb11.dmz.scl3.mozilla.com:2181/hgmoreplication``.
**If this value doesn't match exactly, things may not go as planned.**

The first step is to generate a JSON document that will be used to perform
data reassignment. To do this, we need a list of broker IDs to move data
to and a JSON file listing the topics to move.

The list of broker IDs is the set of Zookeeper IDs as defined in
``ansible/group_vars/hgmo`` (this is the file you changed earlier to
add the new server). Simply select the servers you wish for data to
exist on. e.g. ``7,8,9,10,11``.

The JSON file denotes which Kafka topics should be moved. Typically
every known Kafka topic is moved. Use the following as a template::

   {
     "topics": [
       {"topic": "pushdata"},
       {"topic": "pushlog"},
       {"topic": "replicatedpushdata"},
       {"topic": "__consumer_offsets"}
     ],
     "version": 1
   }

Once you have all these pieces of data, you can run
``kafka-reassign-partitions.sh`` to generate a proposed reassignment plan::

   $ /opt/kafka/bin/kafka-reassign-partitions.sh \
     --zookeeper <hosts> \
     --generate \
     --broker-list <list> \
     --topics-to-move-json-file topics.json

This will output 2 JSON blocks::

   Current partition replica assignment

   {...}
   Proposed partition reassignment configuration

   {...}

You'll need to copy and paste the 2nd JSON block (the proposed reassignment)
to a new file, let's say ``reassignments.json``.

Then we can execute the data reassignment::

   $ /opt/kafka/bin/kafka-reassign-partitions.sh \
     --zookeeper <hosts> \
     --execute \
     --reassignment-json-file reassignments.json

Data reassignment can take up to several minutes. We can see the status
of the reassignment by running:

   $ /opt/kafka/bin/kafka-reassign-partitions.sh \
     --zookeeper <hosts> \
     --verify \
     --reassignment-json-file reassignments.json

If your intent was to move Kafka data off a server, you can verify data
has been removed by looking in the ``/var/lib/kafka/logs`` data on
that server. If there is no topic/partition data, there should be no
sub-directories in that directory. If there are sub-directories
(they have the form ``topic-<N>``), adjust your ``topics.json``
file, generate a new ``reassignments.json`` file and execute a
reassignment.

Removing an old Kafka Node
--------------------------

Once data has been removed from a Kafka node, it can safely be turned off.

The first step is to remove the server from the Zookeeper/Kafka list
in Ansible. See https://hg.mozilla.org/hgcustom/version-control-tools/rev/adc5024917c7
for an example commit. Deploy this via ``./deploy hgmo``.

Next, stop Kafka and Zookeeper from the server::

   $ systemctl stop kafka.service
   $ systemctl stop zookeeper.service

At this point, the old Kafka/Zookeeper node is shut down and should no
longer be referenced.

Clean up by disabling the systemd services::

   $ systemctl disable kafka.service
   $ systemctl disable zookeeper.service
