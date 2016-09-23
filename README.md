# puppet-bamboo

[![Puppet Forge](http://img.shields.io/puppetforge/v/joshbeard/bamboo.svg)](https://forge.puppetlabs.com/joshbeard/bamboo)
[![Build Status](https://travis-ci.org/joshbeard/puppet-bamboo.svg?branch=master)](https://travis-ci.org/joshbeard/puppet-bamboo)

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Usage](#usage)
    * [Defaults](#defaults)
    * [Examples](#examples)
        * [Customizations](#customizations)
        * [Context Path](#context-path)
        * [Reverse Proxy](#reverse-proxy)
        * [Installation Locations](#installation-locations)
        * [JVM Tuning](#jvm-tuning)
        * [Tomcat Tuning](#tomcat-tuning)
4. [Reference](#reference)
    * [Class: bamboo](#class-bamboo)
    * [Other Classes](#other-classes)
5. [Limitations](#limitations)
6. [Development](#development)
7. [Authors and Contributors](#authors-and-contributors)

## Overview

This is a Puppet module to manage the installation and initial configuration
of Atlassian Bamboo, a continuous integration and build server.

* Manages the download and installation of Bamboo
* Manages some pre-installation configuration settings, such as Tomcat ports,
  proxy configuration, Java options
* Manages a bamboo user, group, and home
* Manages a service for Bamboo

This is a fork of [maestrodev/bamboo](https://github.com/maestrodev/puppet-bamboo),
which appears to be dormant.  It includes improvements from other authors as
well, notably, [Simon Croomes](https://github.com/croomes/puppet-bamboo).

This module tries to follow conventions in the
[Confluence](https://github.com/puppet-community/puppet-confluence),
[Jira](https://github.com/puppet-community/puppet-jira), and
[Stash](https://github.com/puppet-community/puppet-stash) modules

## Prerequisites

* nanliu/staging: [http://forge.puppetlabs.com/nanliu/staging](http://forge.puppetlabs.com/nanliu/staging)
* A Java installation (e.g. via [puppetlabs/java](http://forge.puppetlabs.com/puppetlabs/java))

## Usage

### Defaults

To have Puppet install Bamboo with the default parameters, declare the
bamboo class:

```puppet
class { 'bamboo': }
```

The `bamboo` class serves as a single "point of entry" for the module.

### Examples

#### Customizations

```puppet
class { 'bamboo':
  version      => '5.9.7',
  installdir   => '/opt/bamboo',
  homedir      => '/var/local/bamboo',
  user         => 'bamboo',
  java_home    => '/over/the/rainbow/java',
  download_url => 'https://mirrors.example.com/atlassian/bamboo',
  context_path => 'bamboo',
  proxy        => {
    scheme    => 'https',
    proxyName => 'bamboo.example.com',
    proxyPort => '443',
  },
}
```

#### Context Path

Specifying a `context_path` for instances where Bamboo is being served
from a path (e.g. example.com/bamboo)

```puppet
class { 'bamboo':
  context_path => 'bamboo',
}
```

This configures the embedded Bamboo Tomcat instance with the context path.

#### Reverse Proxy

For instances where Bamboo is behind a reverse proxy

```puppet
class { 'bamboo':
  proxy => {
    scheme    => 'https',
    proxyName => 'bamboo.example.com',
    proxyPort => '443',
  },
}
```

This configures the embedded Bamboo Tomcat instance's connector.

The proxy parameter accepts a hash of Tomcat options for configuring the
connector's proxy settings.  Refer to
[Tomcat's documentation](https://tomcat.apache.org/tomcat-7.0-doc/proxy-howto.html)
for more information.

#### Installation locations

```puppet
class { 'bamboo':
  installdir => '/opt/bamboo',
  homedir    => '/opt/local/bamboo',
}
```

#### JVM Tuning

```puppet
class { 'bamboo':
  java_home    => '/usr/lib/java/custom',
  jvm_xms      => '512m',
  jvm_xmx      => '2048m',
  jvm_permgen  => '512m',
  jvm_opts     => '-Dcustomopt',
}
```

#### Tomcat Tuning

Bamboo's powered by an embedded Tomcat instance, which can be tweaked.

```puppet
class { 'bamboo':
  tomcat_port        => '9090',
  max_threads        => '256',
  min_spare_threads  => '50',
  connection_timeout => '30000',
  accept_count       => '200',
}
```

## Reference

### Class: `bamboo`

#### Parameters


##### `version`

Default: '5.9.7'

The version of Bamboo to download and install.  Should be in a MAJOR.MINOR.PATH
format.

Refer to [https://www.atlassian.com/software/bamboo/download](https://www.atlassian.com/software/bamboo/download)

##### `extension`

Default: 'tar.gz'

The file extension of the remote archive.  This is typically `.tar.gz`.
Accepts `.tar.gz` or `.zip`

##### `installdir`

Default: '/usr/local/bamboo'

The base directory for extracting/installing Bamboo to.  Note that it will
decompress _inside_ this directory to a directory such as
`atlassian-bamboo-5.9.7/`  So an `installdir` of `/usr/local/bamboo` will
ultimately install Bamboo to `/usr/local/bamboo/atlassian-bamboo-5.9.7/` by
default.

Refer to `manage_installdir` and `appdir`

##### `manage_installdir`

Default: `true`

Boolean.  Whether this module should be responsible for managing the
`installdir`

##### `appdir`

Default: `${installdir}/atlassian-bamboo-${version}`

This is the directory that Bamboo gets extracted to within the 'installdir'

By default, this is a subdirectory with the specific version appended to it.

You might want to customize this if you don't want to use the default
`atlassian-bamboo-${version}` convention.

##### `manage_appdir`

Default: `true`

Boolean.  Whether this module should be responsible for managing the `appdir`

##### `homedir`

Default: '/var/local/bamboo'

The home directory for the Bamboo user.  This path will be managed by this
module, even if `manage_user` is false.

##### `context_path`

Default: '' (empty)

For instances where Bamboo is being served from a sub path, such as
`example.com/bamboo`, where the `context_path` would be `bamboo`

##### `tomcat_port`

Default: '8085'

The HTTP port for serving Bamboo.

##### `max_threads`

Default: '150'

Maps to Tomcat's `maxThreads` HTTP attribute.

Refer to [https://tomcat.apache.org/tomcat-7.0-doc/config/http.html](https://tomcat.apache.org/tomcat-7.0-doc/config/http.html)
for more information.

##### `min_spare_threads`

Default: '25'

Maps to Tomcat's `minSpareThreads` HTTP attribute.

Refer to [https://tomcat.apache.org/tomcat-7.0-doc/config/http.html](https://tomcat.apache.org/tomcat-7.0-doc/config/http.html)
for more information.

##### `connection_timeout`

Default: '20000'

Maps to Tomcat's `connectionTimeout` HTTP attribute.

Refer to [https://tomcat.apache.org/tomcat-7.0-doc/config/http.html](https://tomcat.apache.org/tomcat-7.0-doc/config/http.html)
for more information.

##### `accept_count`

Default: '100'

Maps to Tomcat's `acceptCount` HTTP attribute.

Refer to [https://tomcat.apache.org/tomcat-7.0-doc/config/http.html](https://tomcat.apache.org/tomcat-7.0-doc/config/http.html)
for more information.

##### `proxy`

Default: {}

Bamboo's proxy configuration for instances where Bamboo's being served with a
reverse proxy in front of it (e.g. Apache or Nginx).

##### `manage_user`

Default: true

Specifies whether the module should manage the user or not.

##### `manage_group`

Default: true

Specifies whether the module should manage the group or not.

##### `user`

Default: 'bamboo'

Bamboo's installation will be owned by this user and the service will run as this user.

##### `group`

Default: 'bamboo'

Bamboo's installation will be owned by this group.

##### `uid`

Default: undef

Optionally specify a UID for the user.

##### `gid`

Default: undef

Optionally specify a GID for the group.

##### `password`

Default: '*'

Specify a password for the user.

##### `shell`

Default: '/bin/bash'

Specify a shell for the user.

##### `java_home`

Default: '/usr/lib/jvm/java'

Absolute path to the Java installation.

##### `jvm_xms`

Default: '256m'

Amount of memory the JVM will be started with.

##### `jvm_xmx`

Default: '1024m'

Maximum amount of memory the JVM has available.

You may need to increase this if you see `java.lang.OutOfMemoryError`

##### `jvm_permgen`

Default: '256m'

Size of the permanent generation heap. Unlikely that you need to tune this.

##### `jvm_opts`

Default: ''

Any custom options to start the JVM with.

##### `jvm_optional`

Default: ''

From Bamboo's default `setenv.sh`:

    Occasionally Atlassian Support may recommend that you set some specific JVM
    arguments.  You can use this variable below to do that.

##### `download_url`

Default: 'https://www.atlassian.com/software/bamboo/downloads/binary'

The base url to download Bamboo from.  This should be the URL _up to_ the
actual filename.  The default downloads from Atlassian's site.

##### `manage_service`

Default: true

Whether this module should manage the service.

##### `service_ensure`

Default: 'running'

The state of the service, if managed.

##### `service_enable`

Default: true

Whether the service should start on boot.

##### `service_file`

Default: `/etc/init.d/bamboo` for everything except EL7

Path to the init script.  Typically, this is `/etc/init.d/bamboo`. On EL7,
systemd is used and this parameter is set to
`/usr/lib/systemd/system/bamboo.service`

##### `service_template`

Default: $bamboo::params::service_template

Template for the init script/service definition.  The module includes an init
script and systemd service configuration, but you can use your own if you'd
like.  This should refer to a Puppet module template. E.g.
`modulename/bamboo.init.erb`

##### `shutdown_wait`

Default: '20',

Seconds to wait for the Bamboo process to stop. (e.g. service bamboo stop will
wait this interval before attempting to kill the process and returning).

### Other Classes

The following classes are called from the base class.  You shouldn't need to
declare these directly.

* [bamboo::install](manifests/install.pp)
* [bamboo::configure](manifests/configure.pp)
* [bamboo::service](manifests/service.pp)

## Limitations

### Tested Platforms

* el6.x

* Puppet 3.x

If you've used this module on other platforms, please submit a pull request
to add it to this list.

### Bamboo Configuration

This module does not manage the initial setup of Bamboo - the steps that are
done via the web interface once installed and running.  This doesn't _appear_
to be easily managed automatically.  This includes database configuration and
the license.  Ultimately, this configuration is placed in
`${homedir}/bamboo-cfg.xml`.  Contributions are welcome to help manage this.

## Development

Please feel free to raise any issues here for bug fixes. We also welcome
feature requests. Feel free to make a pull request for anything and we make the
effort to review and merge. We prefer with tests if possible.

[Travis CI](https://travis-ci.org/joshbeard/puppet-bamboo) is used for testing.

### How to test the Bamboo module

Install the dependencies:
```shell
bundle install
```

Unit tests:

```shell
bundle exec rake spec
```

Syntax validation:

```shell
bundle exec rake validate
```

Puppet Lint:

```shell
bundle exec rake lint
```


## Authors and Contributors

* Refer to the [CONTRIBUTORS](CONTRIBUTORS) file.
* Original module by [MaestroDev](http://www.maestrodev.com) (http://www.maestrodev.com)
* Josh Beard (<josh@signalboxes.net>) [https://github.com/joshbeard](https://github.com/joshbeard)
* Carlos Sanchez (<csanchez@maestrodev.com>)
