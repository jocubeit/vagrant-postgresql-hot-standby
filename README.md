# Vagrant: Postgres 9.1 Hot Standby Streaming Replication (Master/Slave)

This project comprises scripts to set up a Vagrant environment running 
PostgreSQL 9.1 Hot Standby Streaming Replication (Master/Slave).

After looking over a number of articles and blog posts regarding how to do this, including the
PostgreSQL wiki (<http://wiki.postgresql.org/wiki/Hot_Standby>), I found
them all wanting in various ways. Nothing's better than working scripts to see how something
can be set up: So here it is. Perhaps some of the work here can be the basis for some Puppet
or Chef.

As much as possible, this script uses stock files, setup, and packages, and mucks about
with the system only enough to get the script running unattended. Additionally, `vagrant ssh`
and the vagrant share are avoided during setup so that the scripts might work against a traditional VPS.

## Issues

1. On occasion on OS/X, a `vagrant destroy` may crash the system. This may be related to low memory
resources on the host system, the host being hibernated, or `vagrant destroy` running
against "cold" VMs. `vagrant destroy` seems now to be quite stable for single instance
setups, but this one uses two. Maybe that's the problem.

2. The setup of the master and slave allows ssh connections via public keys in both directions.
The only connection that is really important is the one from the master-to-slave to rsync the
archive data over to the slave.

3. To make the psql scripting easier, in the `setup-master` script, the setup user is given
the most liberal settings in `/etc/sudoers`.

4. This is not suitable for a production install. If you want to try it, update the keys in
the `ssh/` directory with your own, and restore the sudoers settings.

## Setup

0. Install VirtualBox from here: <https://www.virtualbox.org/wiki/Downloads>
(tested with 4.1.8 only; 4.1.10 and later: not tested.)

1. The wait-for-reboot script uses netcat to check the availability of the ssh port (netcat seems
to work better than the stock nc).

        brew install netcat

2. To your /etc/hosts file, add

        33.33.33.12    master
        33.33.33.13    slave

3. To build the systems

        bundle
        bundle exec vagrant box add lucid64 http://files.vagrantup.com/lucid64.box
        bundle exec vagrant up
        ./setup

4. One way to verify

        vagrant ssh master
          sudo -u postgres psql -d postgres -c "create database sampledb;"
          sudo -u postgres psql -d sampledb -c "create table tab (val int); insert into tab values (42);"
          exit

        vagrant ssh slave
          sudo bash
          su postgres -c 'psql -d sampledb -c "select * from tab;"'
          exit
          exit

5. Fancier: In `setup-generic` there are lines that are commented out. These lines will conduct an update/upgrade of the system, and will ensure that the VirtualBox 4.1.8 Guest Additions are installed.

## Understanding what's going on

The process followed here is essentially the one outlined on the PostgreSQL wiki (<http://wiki.postgresql.org/wiki/Hot_Standby>). The main things you want to look at are the config
files `pg_hba.conf`, `postgresql.conf`, and `recovery.conf`. To make it easy to compare the files that
are shipped with a vanilla install, I've put original copies of those files into the `stock-configs/` directory,
so that you can diff them with the ones that have been edited for the systems. Also, in the edited
files, I've added comments for each line that has been added.

In the setup, there are two gotchas that seem to slow everyone down. One is that the `recovery.conf`
file goes in the PostgreSQL data directory, not in the main config directory (if they are different
on your system). Second, getting the WAL files over via rsync is a pain. See the scripts for one way
to do it.

## TODOs

1. Add notes that demonstrate failover / promotion of slave to master.

2. The scripts could be even less dependent on the file layout from the Debian PostgresSQL installs.
