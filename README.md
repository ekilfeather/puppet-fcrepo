####Table of Contents

1. [Overview](#overview)
2. [Module Description - What the module does and why it is useful](#module-description)
3. [Prerequisites](#prerequisites)
3. [Setup - The basics of getting started with fcrepo](#setup)
    * [What fcrepo affects](#what-fcrepo-affects)
    * [Setup requirements](#setup-requirements)
    * [Beginning with fcrepo](#beginning-with-fcrepo)
4. [Usage - Configuration options and additional functionality](#usage)
5. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)

##Overview

The fcrepo module installs, configures, and manages Fedora 4 in a single or clustered 
environment.

##Module Description

The fcrepo module manages running Fedora 4 repositories in a single or a clustered 
environment.  The module installs Tomcat,
installs the Fedora WAR file and sets up the FCREPO_HOME directory. It can be used to manage 
the configuration files for every Fedora instance on each node in the cluster.

##Prerequisites

To use this module, you need:

1. Puppet installed (of course). This has been tested with Puppet 3.8.1. 
2. The following Puppet modules:
    * puppetlabs/stdlib
    * 7terminals/tomcat
3. Java already installed on the machine. Usually this can be installed by either:
    * Setting up your OS's packaged Java via Puppet by using the official puppetlabs/java 
    module. This should configure the proper path to the Java installation.
    * Installing a package (such as Oracle's Java RPM), which also sets up the proper path.
    Installing this via a local yum repository can allow Puppet to easily install this 
    for your machines.
4. You'll need to download the Tomcat binary distribution package: 
   <http://tomcat.apache.org/download-70.cgi>
   Choose the correct tar-gzipped package for your platform.  Only `.tar.gz` packages
   are supported at this time.

5. You'll need `fcrepo.war`, built from maven or retrieved from Fedora releases:
<https://wiki.duraspace.org/display/FF/Downloads>

###Install and configure a base installation of Puppet

Puppet Labs has good step-by-step documentation for getting a Puppet master 
and Puppet clients set up.

Install:  <http://docs.puppetlabs.com/guides/installation.html>

Setup:  <http://docs.puppetlabs.com/guides/setting_up.html>

Make sure your agents can contact the master puppet server and receive their 
catalog information:

```sudo puppet agent --test```

###Common Puppet Setup Issues

 * Make sure hosts are all time synch'ed via NTP
 * Use lowercase hostnames, including DNS entries
 * Only install the version of Ruby that is required by the puppetmaster package (Ubuntu)
 * If your host uses a web proxy, include that directive in puppet.conf and also set environment variables. Both are required for module installation.

  
```
http_proxy_host=myproxy.example.com
http_proxy_port=3128
```

```
$ export https_proxy=http://myproxy.example.com:3128
$ export http_proxy=http://myproxy.example.com:3128
```

###Install the extra Puppet modules on your puppet master

```
sudo puppet module install puppetlabs/stdlib
sudo puppet module install 7terminals/tomcat
```

##Setup

###What fcrepo affects

* Fedora service user and group
* Tomcat standalone install
* Fedora WAR
* Fedora directories (home and data)
* Fedora configuration files
* Tomcat service

This module creates a user and group to manage the Fedora service and files,
creates a software directory and a data directory and assigns ownership of
them to the fedora user, then installs standalone versions of Tomcat.  
The module installs Fedora in a
sandboxed environment, with infrastructure software downloaded and
installed from binary distributions, and should work on any Unix environment.

It also deploys the Fedora WAR and Fedora configuration files,
and manages the Fedora Tomcat service.

###Beginning with fcrepo

####Build and install the module

1. Clone this project, change to the `puppet-fcrepo` directory. 

2. Copy the Tomcat binary distribution file you downloaded (see 
[Prerequisites](#prerequisites), above) into the module's `files/` directory:

```
    cp /path/to/source/packages/*.tar.gz files/
```

3. Copy the Fedora 4 WAR file into the module's `files/` directory,
with the name `fcrepo.war`: 

```
    cp /path/to/fcrepo-webapp-<VERSION>.war files/fcrepo.war
```

4. Build the module: 

```
    puppet module build .
```

5. Install the module:

```
    sudo puppet module install pkg/sprater-fcrepo-<version>.tar.gz --ignore-dependencies
```

   where `<version>` is the current version of the module.

You can always update these files (especially `fcrepo.war`) later by
replacing them in the `/etc/puppet/modules/fcrepo/files` directory, then
running the Puppet agent on each of your nodes.

####Enable the module in Puppet

`include 'fcrepo'` in the puppet master's `site.pp` file (located in manifests folder) is enough to get 
you up and running.  If you wish to pass in parameters such as which user and
group to create then you can use instead:                                                                                    

```puppet                                                                                                                                 
class { '::fcrepo':
  user                => 'fcrepo',
  group               => 'fcrepo',
  fcrepo_sandbox_home => '/opt/fcrepo',
  fcrepo_datadir      => '/opt/fcrepo/data',
  fcrepo_configtype   => /fcrepo-4.4.0-minimal-default',
}
```
Note: Placing the above include and class outside of specific node definitions, as above, will apply the fcrepo role to every puppet node. Alternately, place them within an appropriate node block.

##Usage

##Reference

###Classes

####Public Classes

* fcrepo:  Main class, includes all other classes

####Private Classes

* fcrepo::install: Creates the user and group, ensures that the correct
  directories exist, and installs the base software and the Fedora WAR.
* fcrepo::config: Manages the configuration files.
* fcrepo::service: Manages the Tomcat service.

###Parameters

The following parameters are available in the fcrepo module.  They
are grouped into __Environment__, __Infrastructure__, and __Fedora__.

The defaults are defined in `fcrepo::params`, and may be changed there, or
overridden in the Puppet files that include the `fcrepo` class.

####Environment

#####`user`

The Unix user that will own the Fedora directories, software, and data.

Default: **fcrepo**

#####`group`

The Unix group that will own the Fedora directories, software, and data.

Default: **fcrepo**

#####`user_profile`

The absolute path to the shell profile file that should be modified to
update the PATH environment variable.  Can be set to a system-wide profile
(i.e. `/etc/profile`).

Default is **/home/_user_/.bashrc**

####Infrastructure

Software packages by default are installed in the Fedora 4 sandbox directory, owned
by the Fedora Unix user and group.  The user's PATH is modified to point first to 
these tools in the sandbox, and other environment variables may be set in the user's 
profile file.

#####`java_homedir`

The location where Java is installed; this variable is used to set JAVA_HOME. The default 
will attempt to use the Red Hat system Java location.

Default:  **/usr/java/default**

#####`tomcat_source`

The *exact* name of the Tomcat binary distribution package, in *.tar.gz format.
This file should be installed under the module's `files/` directory 
(usually `/etc/puppet/modules/fcrepo/files/`).

Default:  **apache-tomcat-7.0.50.tar.gz**

#####`tomcat_deploydir`

The Tomcat base directory (CATALINA_HOME).

Default:  **_fcrepo sandbox home_/tomcat7**

#### Fedora

#####`fcrepo_sandbox_home`

The home directory for the Fedora environment sandbox.

Default: **/fedora**

#####`fcrepo_datadir`

The Fedora data directory.

Default: **/data**

#####`fcrepo_configdir`

The Fedora configuration directory.

Default: **/fedora/config**

#####`fcrepo_configdir`

The version and type of configuration files that should be used. The configuration files
can vary depending on whether you are using Fedora 4.4.0 or 4.5.0, and whether you are
running under a default (single) or clustered configuration. Some possible choices are:
* fcrepo-4.4.0-minimal-default
* fcrepo-4.4.0-clustered
* fcrepo-4.5.0-minimal-default
* fcrepo-4.5.0-clustered

The minimal default versions use these defaults from the fcrepo package, which includes
using leveldb for infinispan storage.

Default: **fcrepo-4.4.0-minimal-default**

##Limitations

This module does not define the raw filesystem devices, nor mount
any filesystems.  Make sure the filesystem(s) in which the sandbox
and data directories will reside are created and mounted.

This module does not set a password for the Fedora Unix user.  You'll
need to do that yourself.

##Development

See the [DEVELOPERS](DEVELOPERS.md) file for more information on modifying, 
testing, and building this module.
