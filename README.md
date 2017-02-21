dj-wasabi.telegraf
=========

Build status:

[![Build Status](https://travis-ci.org/dj-wasabi/ansible-telegraf.svg?branch=master)](https://travis-ci.org/dj-wasabi/ansible-telegraf)

This role will install and configure telegraf.

Telegraf is an agent written in Go for collecting metrics from the system it's running on, or from other services, and writing them into InfluxDB.

Design goals are to have a minimal memory footprint with a plugin system so that developers in the community can easily add support for collecting metrics from well known services (like Hadoop, Postgres, or Redis) and third party APIs (like Mailchimp, AWS CloudWatch, or Google Analytics).

(https://github.com/influxdb/telegraf)

Requirements
------------

No requirements. (Yes, an Influxdb server somewhere on the network will help though ;-) )

Role Variables
--------------

The following parameters can be set for the Telegraf agent:

* `telegraf_agent_version`: The version of Telegraf to install. Default: `1.0.0`
* `telegraf_agent_rpm_url`: The full path to the RPM file located on a webserver.
* `telegraf_agent_interval`: The interval configured for sending data to the server. Default: `10`
* `telegraf_agent_debug`: Run Telegraf in debug mode. Default: `False`
* `telegraf_agent_round_interval`: Rounds collection interval to 'interval' Default: True
* `telegraf_agent_flush_interval`: Default data flushing interval for all outputs. Default: 10
* `telegraf_agent_flush_jitter`: Jitter the flush interval by a random amount. Default: 0
* `telegraf_agent_collection_jitter`: Jitter the collection by a random amount. Default: 0 (since v0.13)
* `telegraf_agent_metric_batch_size`: The agent metric batch size. Default: 1000  (since v0.13)
* `telegraf_agent_metric_buffer_limit`: The agent metric buffer limit. Default: 10000  (since v0.13)
* `telegraf_agent_quiet`: Run Telegraf in quiet mode (error messages only). Default: `False` (since v0.13)
* `telegraf_agent_logfile`: The agent logfile name. Default: '' (means to log to stdout) (since v1.1)
* `telegraf_agent_omit_hostname`: Do no set the "host" tag in the agent. Default: `False` (since v1.1)

Full agent settings reference: https://github.com/influxdata/telegraf/blob/master/docs/CONFIGURATION.md#agent-configuration

You can set tags for the host running telegraf:

	telegraf_agent_tags:
	  - tag_name: some_name
	    tag_value: some_value

Specifying an output. The default is set to localhost, you'll have to specify the correct influxdb server:

	telegraf_agent_output:
	  - type: influxdb
	    config:
	      - urls = ["http://localhost:8086"]
	      - database = "telegraf"
            tagpass:
              - diskmetrics = ["true"]

The config will be printed line by line into the configuration, so you could also use:

	config:
		- # Print an documentation line

and it will be printed in the configuration file.

There are two properties which are the same, but are used differently. Those are:

* `telegraf_plugins_default`
* `telegraf_plugins_extra`

With the property `telegraf_plugins_default` it is set to use the default set of Telegraf plugins. You could override it with more plugins, which should be enabled at default.

	telegraf_plugins_default:
	  - plugin: cpu
	    config:
	      - percpu = true
	  - plugin: disk
	  - plugin: io
	  - plugin: mem
	  - plugin: system
	  - plugin: swap
	  - plugin: netstat

Every telegraf agent has these as an default configuration.

The 2nd parameter `telegraf_plugins_extra` can be used to add plugins specific to the servers goal. Following is an example for using this parameter for MySQL database servers:

	cat group_vars/mysql_database
	telegraf_plugins_extra:
		- plugin: mysql
		  config:
		  	- servers = ["root:{{ mysql_root_password }}@tcp(localhost:3306)/"]


Telegraf plugin options:

* `tags` An k/v tags to apply to a specific input's measurements. Can be used on any stage for better filtering for example in outputs.
* `pass`: An array of strings that is used to filter metrics generated by the current plugin. Each string in the array is tested as a prefix against metric names and if it matches, the metric is emitted.
* `drop`: The inverse of pass, if a metric name matches, it is not emitted.
* `tagpass`: (added in Telegraf 0.1.5) tag names and arrays of strings that are used to filter metrics by the current plugin. Each string in the array is tested as an exact match against the tag name, and if it matches the metric is emitted.
* `tagdrop`: (added in Telegraf 0.1.5) The inverse of tagpass. If a tag matches, the metric is not emitted. This is tested on metrics that have passed the tagpass test.
* `interval`: How often to gather this metric. Normal plugins use a single global interval, but if one particular plugin should be run less or more often, you can configure that here.

An example might look like this:

	telegraf_plugins_default:
	  - plugin: disk
	    interval: 12
            tags:
                - diskmetrics = "true"
	    tagpass:
	      - fstype = [ "ext4", "xfs" ]
    	  - path = [ "/opt", "/home" ]



Dependencies
------------
No dependencies

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: dj-wasabi.telegraf }

##Contributors
The following have contributed to this Ansible role:

  * aferrari-technisys
  * stvnwrgs
  * lhoss
  * thecodeassassin
  * Ismael
  * romainbureau

# Molecule

This roles is configured to be tested with Molecule. You can find on this page some more information regarding Molecule: https://werner-dijkerman.nl/2016/07/10/testing-ansible-roles-with-molecule-testinfra-and-docker/

License
-------

BSD

Author Information
------------------

Please let me know if you have issues. Pull requests are also accepted! :-)

mail: ikben [ at ] werner-dijkerman . nl
