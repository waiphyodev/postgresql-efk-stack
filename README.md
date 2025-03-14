# postgresql-efk-stack

1. specify **postgresql.conf** to store log file as postgresql.log under log folder of root

```conf
log_destination = 'stderr'
logging_collector = on
log_statement = all
log_directory = '/log'
log_filename = 'postgresql.log'
```

2. `-v ${PWD}/postgres-data/log:/log:Z` is mounting postgresql.log from container to host and `-v ${PWD}/postgresql.conf:/etc/postgresql/postgresql.conf:Z` is mounting postgresql.conf from host to container

```bash
podman run -d --rm \
  --name postgres-test \
  -v ${PWD}/postgres-data/log:/log:Z \
  -v ${PWD}/postgresql.conf:/etc/postgresql/postgresql.conf:Z \
  -e POSTGRES_PASSWORD=password \
  postgres \
  -c 'config_file=/etc/postgresql/postgresql.conf'
```

3. specify **fluent-bit.conf** to collect log from its own mounted from host

```conf
[SERVICE]
    Flush         5
    Log_Level     info
    Daemon        off

[INPUT]
    Name          tail
    Path          /var/log/postgresql.log

[OUTPUT]
    Name          stdout
    Match         *
```

4. `-v ${PWD}/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf:Z` is mounting config file to container and ` -v ${PWD}/postgres-data/log/postgresql.log:/var/log/postgresql.log:Z` is mounting **postgresql.log** which exposed from postgres container to fluent-bit

```bash
podman run -d --rm \
  --name fluent-bit \
  -v ${PWD}/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf:Z \
  -v ${PWD}/postgres-data/log/postgresql.log:/var/log/postgresql.log:Z \
  fluent-bit:latest
```
