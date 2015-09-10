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
-e DEBUG=1 \
-e AWS_ACCESS_KEY=YOUR_AWS_ACCESS_KEY \
-e AWS_SECRET_KEY=YOUR_AWS_SECRET_KEY \
-e AWS_REGION_NAME=YOUR_AWS_REGION_NAME \
-e LK_DOCKER_HOST=YOUR_DOCKER_HOST_NAME \
--volume=/var/run/docker.sock:/tmp/docker.sock \
moovel/logspout-kinesis \
kinesis://YOUR_KINESIS_STREAM_NAME
```

This adapter uses the docker host as the partion key. You can pass the value of it using the env variable `LK_DOCKER_HOST`.

Debug mode can be enabled using the `DEBUG` env variable.

#### Http Stream
This build includes the `httpstream` module which provides realtime access to all logs via the http/websocket protocol. After launching the container using docker run command from above you should now be able to connect to the stream: `curl http://YOUR_DOCKER_HOST_IP:8000/logs` 

#### Label support
This adapter supports docker labels by adding them to the json message which gets then sent to the kinesis stream.

### Configuration
The configuration is provides via the query parameters in the logspout url. Just add the desired parameters by appending them: `kinesis://myStream?buffer_size=20`

| Parameter               | Description                                                                                                            | Default |
|-------------------------|------------------------------------------------------------------------------------------------------------------------|---------|
| add_blocks_when_buffer_full | Should the batch producer add blocks even if the buffer is full?                                                       | false   |
| buffer_size              | Size of the internal kinesis records buffer.                                                                           | 10000   |
| flush_interval           | Number of seconds when a batch gets flushed and sent.                                                                  | 1       |
| batch_size               | Number of records per batch.                                                                                           | 10      |
| max_attempts_per_record    | Number of retries.                                                                                                     | 10      |
| start_interval            | StatInterval will be used to make a *best effort* attempt to send stats *approximately*,// when this interval elapses. | 1       |

Since this configuration is a one-to-one mapping to the [go-kinesis](https://github.com/sendgridlabs/go-kinesis) library defaults, head to the libary [docs](http://godoc.org/github.com/sendgridlabs/go-kinesis/batchproducer#Config) for further information.

## Future improvements
* Add a [StatReceiver](http://godoc.org/github.com/sendgridlabs/go-kinesis/batchproducer#StatReceiver) that sends stats to cloudwatch to allow monitoring.
* Add a more sophisticated partion key configuration.
