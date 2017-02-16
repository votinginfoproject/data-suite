Data Suite
==============

Voting Information Project's Data Dashboard for states

[Learn more](https://votinginfoproject.org/)

## Purpose

The data-suite repo consists of a docker-compose file that will allow you to a
production-like system with both a processor and dashboard.

## Prerequisites

First, you'll need some tools to get the system built:

- Docker: [Docker for Mac][docker] is nice; otherwise install both `docker` and
  `docker-compose` with your favorite package manager
- VALID AWS credentials with write permissions to both S3 and SQS
- [AWS CLI tool][awscli] (also available in Homebrew)

In addition to cloning this repository, you'll need to clone
the [data-processor repository][data-processor] and
the [Metis repository][metis]. Make sure these three repositories are checked
out in the same parent directory as this repo; it should look similar to this:

    $ tree -L 2 ~/src/vip/
    ~/src/vip
    ├── data-processor
    │   ├── README.md
    │   ├── docker-compose.yml
    │   ├── src
    │   └── test
    ├── data-suite
    │   ├── README.md
    │   ├── docker-compose.yml
    │   ├── resources
    │   └── script
    └── Metis
        ├── Dockerfile
        └── README.md

## Running in Docker
### Setup an Environment

Make a copy of `.env_sample`, named `.env`, and put your AWS keys, etc where
they belong. Make sure that the buckets and queues exist; if not, create them.

If you installed `awscli`, you can check and create new queues easily.

    $ aws sqs list-queues
    {
        "QueueUrls": [
            "https://queue.amazonaws.com/123456789012/data-suite-staging",
            "https://queue.amazonaws.com/123456789012/data-suite-staging-fail",
            "https://queue.amazonaws.com/123456789012/data-suite-development",
            "https://queue.amazonaws.com/123456789012/data-suite-development-fail",
            "https://queue.amazonaws.com/123456789012/data-suite-production",
            "https://queue.amazonaws.com/123456789012/data-suite-production-fail",
        ]
    }

    $ aws sqs create-queue --queue-name productionlike-test
    {
        "QueueUrl": "https://queue.amazonaws.com/123456789012/productionlike-test"
    }

Similarly, if you need to create a bucket, use

    aws s3 mb s3://productionlike-test

The sample has comments describing the purpose of all configuration variables
required. Once the environment file is complete, you can build and run the
system.

### Build & Run

1. `docker-compose build --no-cache`
1. `docker-compose create && docker-compose start`

This will build and start the services, returning control to your terminal. To
watch the logs, use `docker-compose logs -f`. To watch only the processor
service, just say so: `docker-compose logs -f processor`.

Then visit the dashboard: [http://localhost:54000/](http://localhost:54000/)

When you're done with the system `docker-compose stop` will turn eveything off
in an orderly fashion.

## Testing the System

The first step to getting a feed processed is to copy the file to your S3
bucket:

    aws s3 cp /path/to/your/vipFeed-08-2017 s3://your-unprocessed-feed-bucket

Then, send an EDN-formatted message to your queue containing the filename of
your feed:

    aws sqs send-message \
        --region us-east-1 \
        --queue-url https://sqs.us-east-1.amazonaws.com/12345/your-queue \
        --message-body "{:filename \"vipFeed-08-2017-11-07.zip\"}"

If you had a set of representative feeds, you could process each of them
multiple times as a rough simulation of load. Given a set of feeds in a S3
bucket named `load-simulation` and a queue named the same, you could flood the
system:

    ./script/flood us-east-1 \
                   https://sqs.us-east-1.amazonaws.com/123456/your-queue \
                   5 \ # Send n messages for each file
                   vipFeed-10-2016-11-08.zip,vipFeed-16-2016-11-08.zip # comma-separated list of filenames

[docker]: https://docs.docker.com/docker-for-mac/
[metis]: https://github.com/votinginfoproject/Metis
[data-suite]: https://github.com/votinginfoproject/suite
[data-processor]: https://github.com/votinginfoproject/data-processor
[aws-cli]: https://github.com/aws/aws-cli
