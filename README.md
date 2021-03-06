# dbanon

[![Build Status](https://travis-ci.org/mpchadwick/dbanon.svg?branch=master)](https://travis-ci.org/mpchadwick/dbanon)

A run-anywhere, dependency-less database anonymizer.

## Installation

Download [the latest release from GitHub](https://github.com/mpchadwick/dbanon/releases).

## Usage

`dbanon` reads from `stdin` and writes to `stdout`.

```
mysqldump --complete-insert mydb | dbanon -config=myconfig.yml | gzip > mydb.sql.gz
```

The `-config` flag can use bundled configurations or point to the path of a custom configuration file. 

### Configuration

#### Magento 2

`dbanon` bundles a [default Magento 2 configuration file](etc/magento2.yml). However you almost certainly won't use it directly.

At minimum, you'll first need to run the `map-eav` subcommand. This translates EAV attribute codes to their respective attribute ids.

You must feed it a `mysqldump` of `eav_entity_type` and `eav_attribute` (in that order).

```
mysqldump --complete-insert mydb eav_entity_type eav_attribute | dbanon -config=magento2 map-eav > ~/magento2-mapped.yml
```

`map-eav` will replace the attribute codes in the config file with attribute ids and print an updated config to `stdout`.

Next you'd run `dbanon` with the config generated by `map-eav`.

```
mysqldump --complete-insert mydb | dbanon -config=~/magento2-mapped.yml | gzip > mydb.sql.gz
```


Most Magento 2 databases, however, will have additional data that needs to be anonymized beyond the default bundled file. 

For this you'll first want to create a new configuration file based off the bundled configuration. Instructions on customizing the configuration file are included in the "Custom Configuration" section.


#### Custom Configuration

Specify the path to your config file via the `-config` flag

```
mysqldump --complete-insert mydb | dbanon -config=myconfig.yml | gzip > mydb.sql.gz
```

See [the `etc` directory](etc/) for examples.

Columns are specified as key / value pairs. The value string winds up getting passed to [this function](https://github.com/mpchadwick/dbanon/blob/e800bae7b8ef2b37bcaf3440273a919ed33ab283/src/provider.go#L19), which gets random values from [`dmgk/faker`](https://github.com/dmgk/faker)


## Limitations

- Currently only supports MySQL
- Currently requires `mysqldump` be run with `--complete-insert` flag.