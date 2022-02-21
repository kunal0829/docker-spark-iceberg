# syntax=docker/dockerfile:1
FROM ubuntu:20.04

ENV PYTHON_VERSION=${PYTHON_VERSION:-"3.9"}

RUN apt-get update && \
    apt-get install -y software-properties-common && \
    add-apt-repository ppa:deadsnakes/ppa && \
    apt-get update && \
    apt-get -y install \
    sudo \
    curl \
    vim \
    unzip \
    openjdk-8-jdk \
    build-essential \
    python3.9 \
    python3.9-dev \
    python3.9-distutils \
    ssh

# Remove python 3.5 that comes with ubuntu 16.04 and symlink downloaded python
RUN apt remove python3.5-minimal -y \
 && ln -sf /usr/bin/python3.9 /usr/bin/python3

# Install Jupyter and other python deps
RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py \
 && python3 get-pip.py \
 && rm get-pip.py \
 && pip3 install jupyter pandas prettytable

## Download spark and hadoop dependencies and install

# Optional env variables
ENV SPARK_HOME=${SPARK_HOME:-"/opt/spark"}
ENV HADOOP_HOME=${HADOOP_HOME:-"/opt/hadoop"}

ENV AWS_ACCESS_KEY_ID="ENTER HERE"
ENV AWS_TEST_BUCKET="ENTER HERE"
ENV AWS_SECRET_ACCESS_KEY="ENTER HERE"
ENV AWS_TEST_ACCOUNT_ID="ENTER HERE"
ENV AWS_REGION="ENTER HERE"

RUN mkdir -p ${HADOOP_HOME} && mkdir -p ${SPARK_HOME}
WORKDIR ${SPARK_HOME}

RUN curl https://dlcdn.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz -o hadoop-3.3.1.tar.gz
RUN tar xvzf hadoop-3.3.1.tar.gz --directory /opt/hadoop --strip-components 1 \
 && rm -rf hadoop-3.3.1.tar.gz

RUN curl https://dlcdn.apache.org/spark/spark-3.1.2/spark-3.1.2-bin-hadoop3.2.tgz -o spark-3.1.2-bin-hadoop3.2.tgz
RUN tar xvzf spark-3.1.2-bin-hadoop3.2.tgz --directory /opt/spark --strip-components 1 \
 && rm -rf spark-3.1.2-bin-hadoop3.2.tgz

RUN for pkg in core auth utils http-client-spi regions aws-core sdk-core url-connection-client profiles aws-json-protocol metrics-spi protocol-core json-utils third-party-jackson-core aws-xml-protocol aws-query-protocol glue s3 kms dynamodb sqs sns; do curl https://repo1.maven.org/maven2/software/amazon/awssdk/${pkg}/2.17.130/${pkg}-2.17.130.jar -o aws-${pkg}.jar && cp aws-${pkg}.jar /opt/spark/jars; done

COPY spark-defaults.conf /opt/spark/conf
COPY iceberg-spark-runtime-3.1.jar /opt/spark/jars/iceberg-spark3-runtime.jar
RUN curl https://repo1.maven.org/maven2/org/reactivestreams/reactive-streams/1.0.3/reactive-streams-1.0.3.jar -o reactive-streams-1.0.3.jar && cp reactive-streams-1.0.3.jar /opt/spark/jars

RUN mkdir -p /home/iceberg/notebooks /home/iceberg/spark-events
COPY ["notebooks/spark-sql-demo.ipynb", "/home/iceberg/notebooks"]

# Add a pyspark-notebook command for convenience
RUN echo '#! /bin/sh' >> /bin/pyspark-notebook \
 && echo 'export PYSPARK_DRIVER_PYTHON=jupyter-notebook' >> /bin/pyspark-notebook \
 && echo "export PYSPARK_DRIVER_PYTHON_OPTS=\"--notebook-dir=/home/iceberg/notebooks --ip='*' --NotebookApp.token='' --NotebookApp.password='' --port=8888 --no-browser --allow-root\"" >> /bin/pyspark-notebook \
 && echo "pyspark" >> /bin/pyspark-notebook \
 && chmod u+x /bin/pyspark-notebook

RUN mkdir -p /root/.ipython/profile_default/startup
COPY ipython/startup/00-prettytables.py /root/.ipython/profile_default/startup
COPY ipython/startup/README /root/.ipython/profile_default/startup

COPY entrypoint.sh .

ENTRYPOINT ["./entrypoint.sh"]