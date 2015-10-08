# logspout-kinesis
A [logspout](https://github.com/gliderlabs/logspout) adapter that uses the [go-kinesis](https://github.com/sendgridlabs/go-kinesis) library to send the logs from all docker containers to a configurable [Amazon Kinesis](http://aws.amazon.com/de/documentation/kinesis/) stream to allow further processing by [logstash](https://www.elastic.co/products/logstash).

The source of this project is heavily based on the [logspout-redis-logstash](https://github.com/rtoma/logspout-redis-logstash) sources.

## Build
Run the `build.sh` script to create the docker image.

## Usage

### Run
Run the logspout kinesis container on the docker host where you want to capture logs by running: 

```
docker run -t -i -p 8000:80 \
  -e LOGSPOUT=ignore \
  -e AWS_ACCESS_KEY=YOUR_AWS_ACCESS_KEY \
  -e AWS_SECRET_KEY=YOUR_AWS_SECRET_KEY \
  -e AWS_REGION_NAME=YOUR_AWS_REGION_NAME \
  -e LK_DOCKER_HOST=YOUR_DOCKER_HOST_NAME \
  --restart="always" \
  --volume=/var/run/docker.sock:/tmp/docker.sock:ro \
  moovel/logspout-kinesis \
  kinesis://YOUR_KINESIS_STREAM_NAME
```
#### Authentication
If you have a valid IAM profile present for the instance you running on, you can omit the env variables `AWS_ACCESS_KEY`, `AWS_SECRET_KEY` from the run command and it will try to query the meta-data service for IAM information .  

#### Partition
This adapter uses the docker host as the partition key. You can pass the value of it using the env variable `LK_DOCKER_HOST`.

#### Http Stream
This build includes the `httpstream` module which provides realtime access to all logs via the http/websocket protocol. After launching the container using docker run command from above you should now be able to connect to the stream: `curl http://YOUR_DOCKER_HOST_IP:8000/logs` 

#### Label support
This adapter supports [docker labels](https://docs.docker.com/userguide/labels-custom-metadata/) by adding them to the json message which gets then sent to the kinesis stream.

### Configuration

#### logspout url
The configuration is provided via the query parameters in the logspout url. Just add the desired parameters by appending them: `kinesis://myStream?buffer_size=20`. 

Alternatively you can set the specified environment variable when the config option supports that.

| Parameter               | Description                                                                                                            | Default | ENV var |
|-------------------------|------------------------------------------------------------------------------------------------------------------------|---------|---------|
| docker_host    | The host name docker is running on.                                                                                                     | "unknown-docker-host"      | LK_DOCKER_HOST |
| partition_key    | The partition key for kinesis.                                                                                                     | docker_host value     | LK_PARTITION_KEY |
| use_v0_layout    | Use the logstash v0 layout.                                                                                                     | false      | LK_USE_V0_LAYOUT |
| add_blocks_when_buffer_full | Should the batch producer add blocks even if the buffer is full?                                                       | false   | |
| buffer_size              | Size of the internal kinesis records buffer.                                                                           | 10000   | |
| flush_interval           | Number of seconds when a batch gets flushed and sent.                                                                  | 1       | |
| batch_size               | Number of records per batch.                                                                                           | 10      | |
| max_attempts_per_record    | Number of retries.                                                                                                     | 10      | |
| stat_interval            | **NOT RELEVANT YET** StatInterval will be used to make a *best effort* attempt to send stats *approximately*, when this interval elapses. | 1       | |

Since this configuration is a one-to-one mapping to the [go-kinesis](https://github.com/sendgridlabs/go-kinesis) library defaults, head to the libary [docs](http://godoc.org/github.com/sendgridlabs/go-kinesis/batchproducer#Config) for further information.

#### Debug
Debug mode can be enabled using the `DEBUG` env variable via `docker run -e DEBUG=1 `.

## Future improvements
* Add a [StatReceiver](http://godoc.org/github.com/sendgridlabs/go-kinesis/batchproducer#StatReceiver) that sends stats to cloudwatch to allow monitoring.
