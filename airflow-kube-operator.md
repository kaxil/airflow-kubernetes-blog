# Introduction

Today, we are excited to announce a Kubernetes Operator to increase airflow's
viability as a job orchestration engine using the power of the Kubernetes cloud
deployment framework. 

Since its inception, airflow's power as a job
orchestrator has been its flexibility. Airflow offers a wide range of native
operators (for services such as spark, hbase, etc.) while also offering easy
extensibility through its plugin framework. However, one limitation of the
Apache Airflow project is that...

Over the next few months, we will be offering
a series of kubernetes-based offerings that will vastly expand airflows' native
capabilities. With the addition of a Kubernetes Operator, users will be able to
launch arbitrary docker containers with customizable resources, secrets, and...

## What is kubernetes? (Is this section necessary?)

[Kubernetes](https://kubernetes.io/) is an open-source container deployment
engine released by Google. Based on google's own
[Borg](http://blog.kubernetes.io/2015/04/borg-predecessor-to-kubernetes.html),
kubernetes allows for easy deployment of images using a highly flexible API.
Using kubernetes you can [deploy spark jobs](https://github.com/apache-spark-
on-k8s/spark), launch end-to-end applications, or ... using yaml files, python,
golang, or java bindings. The kubernetes API's programatic launching of
containers seemed a perfect marriage with Airflow's "code as configuration"
philosophy.

# The Kubernetes Operator: The "Whatever-your-heart-desires" Operator

As DevOps
pioneers, we are always looking for ways to make our deployments and ETL
pipelines simpler to manage. Any opportunity to reduce the number of moving
parts in our codebase will always lead to future opportunities to break in the
future. The following are a list of benefits the Kubernetes Operator in reducing
the Airflow Engineer's footprint

* **Increased flexibility for deployments**
Airflow's plugin API has always offered a significant boon to engineers wishing
to test new functionalities within their DAGS, however it has always had the
downside that to create a new operator, one must develop an entirely new plugin.
Now any task that can be run within a docker container is accessible through the
same same operator with no extra airflow code to maintain.
* **Flexibility of
configurations and dependencies** For operators that are run within static
airflow workers, dependency management can become quite difficult. If I want to
run one task that requires scipy, and another that requires numpy, I have to
either maintain both dependencies within my airflow worker, or somehow configure

# Examples

## Example 1: Running a basic container

For this first example, let's create a basic docker image that runs simple
python commands. This example will only have two end-results: Succeed or fail.

```python
import sys


def fail():
    return 10 / 0


def succeed():
    return 10 / 1


if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: wordcount <file>", file=sys.stderr)
        exit(-1)
    arg = sys.argv[1]
    if arg == 'pass':
        succeed()
    elif arg == 'fail':
        fail()
    else:
        print('invalid command: {}'.format(arg))
        exit(-1)
```

To run this code, we'll create a DockerFile and an `entrypoint.sh` file that
will run our basic python script (while this entrypoint script is pretty simple
at the moment, we will later see how it can expand to more complex use-cases)

#### Entrypoint.sh

```python
#!/usr/bin/env bash

python /airflow-example.py
```

#### Docker File

```python
FROM ubuntu:16.04

# install python dependencies
RUN apt-get update -y && apt-get install -y \
        wget \
        python-dev \
        python-pip \
        libczmq-dev \
        libcurlpp-dev \
        curl \
        libssl-dev \
        git \
        inetutils-telnet \
        bind9utils

RUN pip install -U setuptools && \
    pip install -U pip


COPY airflow-example.py /airflow-example.py
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

Finally, we will create a simple DAG file that runs a passing and failing
version of our example script. This script can be run on any executor.

```python
# Airflow DAG File that creates two kubernetes operators, one that passes and one that fails

passing = KubernetesPodOperator(namespace='default',
                          image="airflow-test",
                          arguments=["succeed"],
                          labels={"foo": "bar"},
                          name="passing-test",
                          task_id="task1"
                          )

failing = KubernetesPodOperator(namespace='default',
                          image="airflow-test",
                          arguments=["succeed"],
                          labels={"foo": "bar"},
                          name="fail",
                          task_id="task1"
                          )

failing.set_upstream(passing)
```

### This will eventually be a series of images about the running DAGs

<img
src="files/image.png">

Link to github file

## Example 2: Running a model using scipy

## Example 3: Running a kubeflow pipeline

Paragraph about kubeflow and how awesome it will be

* Developed by Google and
Bloomberg to make kubernetes data development easier

## How it Works

# Closing Statements

Final statements about all the possibilities this opens up

* Airflow Kubernetes
Executor
* Custom Deployments via python API