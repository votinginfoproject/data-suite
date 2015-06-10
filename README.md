Data Dashboard
==============

Voting Information Project's Data Dashboard for states.

[Learn more](https://votinginfoproject.org/)

## Purpose

The data-suite repo consists of a docker-compose file that will allow you to run both data-processor and Metis at the same time.

## Running in Docker

> You will first need to ensure that data-processor and Metis are in the right file structure in relation to this
> repo. It should look like:


```
-- voting-information-project/
  -- data-suite/
    docker-compose.yml
    ...
  -- data-processor/
  -- Metis/
```

> To run in docker, the simplest way is to have [docker-compose](https://docs.docker.com/compose/)
> installed (`brew install docker-compose`).

1. `docker-compose build`
1. `docker-compose up`
1. Then hit [http://localdocker:4000/](http://localdocker:4000/), assuming you have a localdocker host file entry pointing to your docker host. If not, replace localdocker with your docker host IP address.
