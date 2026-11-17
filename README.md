# puppet-sysctl

#### Table of Contents

1. [Module Description - What the module does and why it is useful](#module-description)
1. [Setup - The basics of getting started with sysctl](#setup)
   * [What sysctl affects](#what-sysctl-affects)
   * [Setup requirements](#setup-requirements)
   * [Beginning with sysctl](#beginning-with-sysctl)
1. [Usage - Configuration options and additional functionality](#usage)
1. [Reference - Parameter reference](#reference)
1. [Limitations - OS compatibility, etc.](#limitations)
1. [Development - Guide for contributing to the module](#development)

## Module description

Manage sysctl variable values. All changes are immediately applied, as well as
configured to become persistent.

* `sysctl`: Defined type to manage sysctl variables by setting a value.
* `sysctl::base`: Base class (included from the defined type).

## Setup

### What sysctl affects

* One file per variable in `/etc/sysctl.d/`.
* The `/etc/sysctl.d/` directory itself, which can optionally be purged.
* The few original settings in `/etc/sysctl.conf`: the value is also replaced
  there, so that running `sysctl -p` does not revert a change made by Puppet.

### Setup requirements

This module requires `puppetlabs/stdlib` (see `metadata.json` for the
supported versions). For persistence to work, the operating system needs to
support looking for sysctl configuration inside `/etc/sysctl.d/`.

### Beginning with sysctl

Declare a `sysctl` resource. The `sysctl::base` class is included
automatically.

```puppet
sysctl { 'net.ipv4.ip_forward': value => '1' }
```

## Usage

Set a value for maximum number of connections per UNIX socket:

```puppet
sysctl { 'net.core.somaxconn': value => '65536' }
```

Make sure there is no explicit value set for swappiness, typically because it
was set at some point but no longer needs to be. The original value for
existing nodes is not reset until the next reboot:

```puppet
sysctl { 'vm.swappiness': ensure => absent }
```

If the order in which the files get applied is important, set it with a file
name prefix, which can also be set globally from `site.pp`:

```puppet
Sysctl { prefix => '60' }
```

### Purging

You may optionally enable purging of the `/etc/sysctl.d/` directory, so that
all files which are not (or no longer) managed by this module are removed.
For the purge to work, the node needs at least one `sysctl` resource, or
`sysctl::base` must be included manually. Forcing a value to
`ensure => absent` always works.

```yaml
sysctl::base::purge: true
```

### Hiera

All sysctl keys can be managed from Hiera through the `values` parameter of
the `sysctl::base` class. If the values are spread across different Hiera
levels, set `hiera_merge_values` to `true` to merge all of them instead of
only using the first one found.

```yaml
sysctl::base::hiera_merge_values: true
sysctl::base::values:
  net.ipv4.ip_forward:
    value: '1'
  net.core.somaxconn:
    value: '65536'
  vm.swappiness:
    ensure: absent
```

### Original /etc/sysctl.d entries

When purging, Puppet may want to remove files from `/etc/sysctl.d/` that were
not created by Puppet but need to be present. Setting the same values with
this module is not always enough: if the file comes from an OS package, it
can re-appear when the package is updated. To work around this, manage an
identical file with this module:

```puppet
package { 'libvirt': ensure => installed }
-> sysctl { 'libvirtd':
  suffix => '',
  source => "puppet:///modules/${module_name}/libvirtd.sysctl",
}
```

## Reference

This module is documented via
[puppet-strings](https://github.com/puppetlabs/puppet-strings). See
[REFERENCE.md](REFERENCE.md) for the full list of classes, defined types and
all of their parameters.

## Limitations

This module supports Puppet 8 (`>= 8.0.0 < 9.0.0`) and `puppetlabs/stdlib`
`>= 9.0.0 < 11.0.0`. The supported OS matrix is defined in `metadata.json`,
and CI runs `pdk validate` and `pdk test unit`.

* RedHat 8, 9, 10
* Rocky 8, 9, 10
* SLES 12, 15
* Ubuntu 20.04, 22.04, 24.04
* Debian 11, 12

## Development

This module is managed with [PDK](https://www.puppet.com/docs/pdk). Before
submitting changes, run the validators and unit tests:

```bash
pdk validate
pdk test unit
```

Keep `REFERENCE.md` in sync by regenerating it whenever parameters or their
documentation change. Never edit it by hand:

```bash
pdk bundle exec puppet strings generate --format markdown
```
