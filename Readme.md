Create Service for federation senders
`sudo nano /etc/systemd/system/matrix-synapse-federation-sender\@.service`

```
[Unit]
Description=Synapse %i
AssertPathExists=/etc/matrix-synapse/workers/%i.yaml

# This service should be restarted when the synapse target is restarted.
PartOf=matrix-synapse.target
ReloadPropagatedFrom=matrix-synapse.target

# if this is started at the same time as the main, let the main process start
# first, to initialise the database schema.
After=matrix-synapse.service

[Service]
Type=notify
NotifyAccess=main
User=matrix-synapse
WorkingDirectory=/var/lib/matrix-synapse
EnvironmentFile=-/etc/default/matrix-synapse
ExecStart=/opt/venvs/matrix-synapse/bin/python -m synapse.app.federation_sender --config-path=/etc/matrix-synapse/homeserver.yaml --config-path=/etc/matrix-synapse/conf.d/ --config-path=/etc/matrix-synapse/wo>ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=3
SyslogIdentifier=matrix-synapse-%i

[Install]
WantedBy=matrix-synapse.target
```

Create a service for the workers
`sudo nano /etc/systemd/system/matrix-synapse-worker\@.service`

```
[Unit]
Description=Synapse %i
AssertPathExists=/etc/matrix-synapse/workers/%i.yaml

# This service should be restarted when the synapse target is restarted.
PartOf=matrix-synapse.target
ReloadPropagatedFrom=matrix-synapse.target

# if this is started at the same time as the main, let the main process start
# first, to initialise the database schema.
After=matrix-synapse.service

[Service]
Type=notify
NotifyAccess=main
User=matrix-synapse
WorkingDirectory=/var/lib/matrix-synapse
EnvironmentFile=-/etc/default/matrix-synapse
ExecStart=/opt/venvs/matrix-synapse/bin/python -m synapse.app.generic_worker --config-path=/etc/matrix-synapse/homeserver.yaml --config-path=/etc/matrix-synapse/conf.d/ --config-path=/etc/matrix-synapse/worke>ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=3
SyslogIdentifier=matrix-synapse-%i

[Install]
WantedBy=matrix-synapse.target
```



within the homeserver.yaml


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