# Create and Configure Federation Sender and Generic Wrokers

## Table of contents
 - [Create Worker configs](https://github.com/jameskitt616/matrix-synapse-worker-config#create-worker-configs)
 - [Create Services](https://github.com/jameskitt616/matrix-synapse-worker-config#create-services)
 - [Update homeserver.yaml](https://github.com/jameskitt616/matrix-synapse-worker-config#update-homeserveryaml)
 - [Restart Synapse and the workers](https://github.com/jameskitt616/matrix-synapse-worker-config#restart-synapse-and-the-workers)

## Create Worker configs

### Create log-configs

[nano /etc/matrix-synapse/federation-worker-log.yaml](https://github.com/jameskitt616/matrix-synapse-worker-config/blob/master/etc/matrix-synapse/federation-worker-log.yaml)

[nano /etc/matrix-synapse/generic-worker-log.yaml](https://github.com/jameskitt616/matrix-synapse-worker-config/blob/master/etc/matrix-synapse/generic-worker-log.yaml)

### Create worker-configs

create a `workers` directory within the matrix-synapse folder

`mkdir /etc/matrix-synapse/workers`

and create [all the workers](https://github.com/jameskitt616/matrix-synapse-worker-config/tree/master/etc/matrix-synapse/workers)

## Create Services

[nano /etc/systemd/system/matrix-synapse-federation-sender\\@.service](https://github.com/jameskitt616/matrix-synapse-worker-config/blob/master/etc/systemd/system/matrix-synapse-federation-sender\\@.service)

[nano /etc/systemd/system/matrix-synapse-worker\\@.service](https://github.com/jameskitt616/matrix-synapse-worker-config/blob/master/etc/systemd/system/matrix-synapse-worker\\@.service)

[nano /etc/systemd/system/matrix-synapse.target](https://github.com/jameskitt616/matrix-synapse-worker-config/blob/master/etc/systemd/system/matrix-synapse.target)

[nano /etc/systemd/system/matrix-synapse.service](https://github.com/jameskitt616/matrix-synapse-worker-config/blob/master/etc/systemd/system/matrix-synapse.service)

## Update homeserver.yaml

Update the Workers section in your `homeserver.yaml`

Also define a random secret string at the `worker_replication_secret` line, also update this in your [workers configs](https://github.com/jameskitt616/matrix-synapse-worker-config/tree/master/etc/matrix-synapse/workers).
```
## Workers ##

# Disables sending of outbound federation transactions on the main process.
# Uncomment if using a federation sender worker.
#
send_federation: false

# It is possible to run multiple federation sender workers, in which case the
# work is balanced across them.
#
# This configuration must be shared between all federation sender workers, and if
# changed all federation sender workers must be stopped at the same time and then
# started, to ensure that all instances are running with the same config (otherwise
# events may be dropped).
#
federation_sender_instances:
  - federation_sender1
  - federation_sender2

# When using workers this should be a map from `worker_name` to the
# HTTP replication listener of the worker, if configured.
#
instance_map:
  generic_worker1:
    host: localhost
    port: 8036
  generic_worker2:
    host: localhost
    port: 8037
  federation_sender1:
    host: localhost
    port: 8034
  federation_sender2:
    host: localhost
    port: 8035


worker_replication_secret: "define own random string"


# Configuration for Redis when using workers. This *must* be enabled when
# using workers (unless using old style direct TCP configuration).
#
redis:
  # Uncomment the below to enable Redis support.
  #
  enabled: true

  # Optional host and port to use to connect to redis. Defaults to
  # localhost and 6379
  #
  #host: localhost
  #port: 6379

  # Optional password if configured on the Redis instance
  #
  #password: <secret_password>
```

## Restart Synapse and the workers

`sudo systemctl restart matrix-synapse.target`