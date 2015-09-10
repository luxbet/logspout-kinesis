# moovel-logspout-kinesis
A [logspout](https://github.com/gliderlabs/logspout) adapter that uses the [go-kinesis](https://github.com/sendgridlabs/go-kinesis) library to send the logs from all docker containers to a configurable [Amazon Kinesis](http://aws.amazon.com/de/documentation/kinesis/) stream to allow further processing by [logstash](https://www.elastic.co/products/logstash).

## Build
Run the `build.sh` script to create the docker image.

## Usage

### Run
Run the logspout kinesis container on the docker host where you want to capture logs by running: 

```
docker run -t -i -e DEBUG=1 -p 8000:80 -e AWS_ACCESS_KEY=YOUR_AWS_ACCESS_KEY -e AWS_SECRET_KEY=YOUR_AWS_SECRET_KEY -e AWS_REGION_NAME=YOUR_AWS_REGION_NAME --volume=/var/run/docker.sock:/tmp/docker.sock moovel/moovel-logspout-kinesis kinesis://YOUR_KINESIS_STREAM_NAME
```

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
