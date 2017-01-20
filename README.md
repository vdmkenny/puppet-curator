# puppet-curator

This module installs, configures, and manages [curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/index.html), an [elastic](https://www.elastic.co/) product that allows management of an elasticsearch cluster indices.

It has been inspired of the [jlambert121-curator](https://github.com/jlambert121/jlambert121-curator) module but completely redesigned in order to support curator `> 4.x`

## Usage

This is a example usage of how you can use this module

### Include the main class

```puppet

include ::curator

```

### Define an action

```puppet

curator::action { 'purge_logstash_over_45_days':
  entities => {
    1 => {
      'action' => 'delete_indices',
      'description' => 'Delete indices older than 45 days (based on index name)',
      'options' => {
        'continue_if_exception' => 'True',
        'disable_action'        => 'False',
        'ignore_empty_list'     => 'True',
      },
      'filters' => [
        {
          'filtertype' => 'pattern',
          'kind'       => 'prefix',
          'value'      => 'logstash-',
        },
        {
          'filtertype' => 'age',
          'source'     => 'name',
          'direction'  => 'older',
          'timestring' => '%Y.%m.%d',
          'unit'       => 'days',
          'unit_count' => '45',
        },
      ],
    },
  },
}
```

### Define a job that automates the trigger of the action

```puppet

curator::job { 'purge_logstash_over_45_days_everyday':
  action => 'purge_logstash_over_45_days',
  minute => '0',
  hour   => '0',
}

```

### Builtin Hiera support

You can also directly use hiera in order to manage your resources :

```yaml

curator::actions:
  'purge_logstash_over_45_days':
    1:
      action: delete_indices
      description: Delete indices older than 45 days (based on index name)
      options:
        continue_if_exception: 'True'
        disable_action: 'False'
        ignore_empty_list: 'True'
      filters:
        - filtertype: pattern
          kind: prefix
          value: logstash-
        - filtertype:  age
          source:      name
          direction:   older
          timestring:  '%Y.%m.%d'
          unit:        days
          unit_count:  '45'

curator::jobs:
  'purge_logstash_over_45_days_everyday':
    action: 'purge_logstash_over_45_days'
    minute: '0'
    hour: '0'

```

## Limitations

This module has been built on and tested against Puppet 4.3 and higher.

The module has been tested on:

- Debian 7/8
- CentOS 6/7
- Ubuntu 14.04, 16.04

Testing on other platforms has been light and cannot be guaranteed.

## Contributing

Please report bugs and feature request using [GitHub issue tracker](https://github.com/mvisonneau/puppet-curator/issues).

For pull requests, it is very much appreciated to check your Puppet manifest with puppet-lint to follow the recommended Puppet style guidelines from the [Puppet Labs style guide](http://docs.puppetlabs.com/guides/style_guide.html).