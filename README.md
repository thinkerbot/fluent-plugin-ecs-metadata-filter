# fluent-plugin-ecs-metadata-filter

[![Build Status](https://travis-ci.org/michaelgruber/fluent-plugin-ecs-metadata-filter.svg?branch=master)](https://travis-ci.org/michaelgruber/fluent-plugin-ecs-metadata-filter)
[![Code Climate](https://codeclimate.com/github/michaelgruber/fluent-plugin-ecs-metadata-filter/badges/gpa.svg)](https://codeclimate.com/github/michaelgruber/fluent-plugin-ecs-metadata-filter)
[![Test Coverage](https://codeclimate.com/github/michaelgruber/fluent-plugin-ecs-metadata-filter/badges/coverage.svg)](https://codeclimate.com/github/michaelgruber/fluent-plugin-ecs-metadata-filter/coverage)

Filter plugin to add AWS ECS metadata to fluentd events. Based on [fabric8io/fluent-plugin-kubernetes_metadata_filter](https://github.com/fabric8io/fluent-plugin-kubernetes_metadata_filter).

## Requirements

| fluent-plugin-ecs-metadata-filter | fluentd | ruby |
|------------------------|---------|------|
| >= 0.2.0 | >= v0.14.0 | >= 2.1 |
|  < 0.2.0 | >= v0.12.0 | >= 1.9 |

## Installation

```ruby
gem install fluent-plugin-ecs-metadata-filter
```

## Configuration

Configuration options for `fluent.conf` are:

* `cache_size`     - Size of the cache of ECS container metadata which reduces requests to the API server  - default: `1000`
* `cache_ttl`      - TTL in seconds for each cached element. Set to negative value to disable TTL eviction - default: `3600` (1 hour)
* `fields_key`     - Key in the final record holding the metadata fields.  Set to "" to set fields in the record itself - default: `ecs`
* `fields`         - Array of metadata fields that should be added to a log record                         - default: `docker_name`, `family`, `cluster`, `name` - **Available options:**
  + `cluster`
  + `container_instance_arn`
  + `container_instance_version`
  + `desired_status`
  + `docker_id`
  + `docker_name`                - Name of the docker container
  + `family`
  + `known_status`
  + `name`                       - Name as specified in the task definition
  + `task_arn`
  + `version`
* `merge_json_log` - Merge in JSON format as top level keys in the log, if the log is JSON                 - default: `true`
* `tag_regexp`     - Regular expression used to extract the `docker_id` from the fluentd tag               - default: `var\.lib\.docker\.containers\.(?<docker_id>[a-z0-9]{64})\.[a-z0-9]{64}-json.log$` (note the full 64-character id is needed - a short id will not work)

Reading from the docker container 

```
<source>
  type tail
  path /var/lib/docker/containers/*/*-json.log
  pos_file fluentd-docker.pos
  time_format %Y-%m-%dT%H:%M:%S
  tag ecs.*
  format json
  read_from_head true
</source>

<filter ecs.var.lib.docker.containers.*.*-json.log>
  type ecs_metadata
</filter>

<match **>
  type stdout
</match>
```
