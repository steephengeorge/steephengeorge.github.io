---
layout: inner
position: center
title: Apache Kafka Connectors
date: 2022-11-01 14:15:00
categories: Apache-Kafka 
tags: Apache-Kafka Kafka-Connect
lead_text: "Apache Kafka connector ecosystem is an integration layer with different systems and Apache Kafka. Connectors are categorized as sink and source connectors. Sink connectors move data from Apache Kafka to other data stores like databases and cloud storage solutions like Amazon Simple Storage Service(s3). Source connectors move data from external systems to Apache Kafka."
project_link: 'http://steephengeorge.github.io/apache-kafka/2022/11/01/Apache-Kafka-Connectors.html'
---

#  Apache Kafka Connectors



Apache Kafka connector ecosystem is an integration layer with different
systems and Apache Kafka. Connectors are categorized as sink and source
connectors. Sink connectors move data from Apache Kafka to other data
stores like databases and cloud storage solutions like Amazon Simple
Storage Service(s3). Source connectors move data from external systems
to Apache Kafka.

There are a considerable number of connectors available as open-source
solutions and quite a lot are commercial versions. It is now a common
practice to build a connector for new data stores by vendors. For
example, Snowflake built a sink connector.

The Confluent team, who built Apache Kafka at LinkedIn, built a
considerable number of connectors. They have a huge number of commercial
connectors along with free ones. Confluent hosted a
[hub](https://www.confluent.io/hub/) for connectors and collected
quite a lot of connectors authored by Confluent engineers as well as
others. It looks like this is a central repository for all connectors
available across the world, but that is not true. But I recommend this
as the [first place](https://www.confluent.io/hub/) to look for
an Apache Kafka connector. 

Another interesting place to find a bunch of connectors is the
[aiven.io
website](https://docs.aiven.io/docs/products/kafka/kafka-connect/concepts/list-of-connector-plugins.html).
They listed a bunch of connectors built by their engineers as well as
others. I'm not sure what their relationship is with lenses.io, but they
listed the connectors from the GitHub repository of lenses.io also in
their list. Most of the connectors listed by aiven.io are open-source
with Apache 2.0 license.

Apache Kafka Sink connectors for Amazon S3

There are at least three Apache Kafka sink connectors available for
Amazon Simple Storage Service(s3). All of them are available for free
even though there is a subtle difference for one of them in its license.
As per my preference, I listed them as follows. I will explain the
reason for that preference in detail later.

-   [Aiven s3 sink
    connector](https://github.com/aiven/s3-connector-for-apache-kafka)

-   [Confluent s3 sink
    connector](https://github.com/confluentinc/kafka-connect-storage-cloud)

-  [Lenses s3 sink
    connector](https://github.com/lensesio/stream-reactor/tree/master/kafka-connect-aws-s3)

I was trying to move data from my Apache Kafka cluster data to Amazon
s3. More specifically, in my cluster, a Confluent Schema Registry
instance is running for schema management. The Schema Registry by
default creates a topic called \_schemas. It is recommended to keep a
backup of \_schemas topic to manage the disaster recovery. And Confluent
recommendation is to use ByteArrayConverter as a serialization mechanism
to backup \_schemas topic.

## Aiven s3 sink connector

This is my favorite connector of all. I found this one the last though.
I initially tried the Confluent s3 sink connector, then the lenses s3
sink connector.

##### Pros: 

1.  It creates a file with the name of the topic and partition combined
    as a file name. You can modify this pattern as you want. 

2.  It supports multiple compression types including gzip, zstd and more

3.  It can write the key, value, offset, timestamp, and headers as a
    single line in the destination s3 file

4.  It supports ByteArrayConverter as a serialization mechanism.

5.  It supports s3 authentication by key_id and secret key as
    configurable as well as supports Security Token service

6.  It has a clear documentation

7.  It has Apache 2.0 license, the code is publicly available and easy
    to make your build.

##### Cons:

1.  If your cloud setup does not support configurable authentication as
    in pros \#5 and prefers profile-based authentication, you have to
    make code changes. My
    [[PR]{.ul}](https://github.com/steephengeorge/s3-connector-for-apache-kafka/tree/provider-chain-enablement)
     is on the way.

2.  There is no source connector available right now. 

## Confluent s3 sink connector

This is the elephant in the room. Wherever you search, you find a lot of
noise about this connector if you have to move data to AWS s3 from your
Apache Kafka. Probably it started early and the solution is convoluted
in several ways. 

##### Pros: 

1.  It supports ByteArrayConverter

2.  It is free according to the Confluent hub and supports Confluent
    Community License

3.  It supports profile-based authentication as well as configurable s3
    authentication by key_id and secret_key.

##### Cons: 

1.  It supports only compression type gzip

2.  It writes keys, values, and headers in different files. This is a
    big mess and I did not have any clue how to restore this data in
    case of a disaster. You can build a custom transform to overcome
    this scenario by ignoring the configuration for the key and headers.

3.  If I configure to backup headers and one of the messages does not
    have a header, this will throw NullPointerException. 

4.  In code and configuration instructions, I noted security token
    service references but did not find clear instructions on how to use
    them.

5.  I could not build the source code even though it is publicly
    available. It has a dependency on other confluent repositories. I
    did not look into the availability of other dependent Confluent
    repositories though.

6.  Confluent source connector is only available as an evaluation
    version. On top, they don't sell connectors piecemeal. If you need
    their commercial connectors, you have to go for the Confluent
    platform.

## Lenses s3 sink/source connector

I am not sure if the GitHub repo is active or not. This solution is
built using Scala. 

##### Pros:

1.  It holds the Apache 2.0 license, so if you find a bug I hope you can
    go and fix it if you have the skill to write Scala code. 

2.  It supports profile-based authentication as well as configurable s3
    authentication by key_id and secret_key.

3.  There is an s3 source connector available among lenses connectors

##### Cons:

1.  I noted lenses slack group is almost inactive and almost 75+ issues
    are piled up on this repo. The repo holds the code of all connectors
    from Lenses, not only the s3 sink connector.

2.  It failed when I used ByteArrayConverter and JsonConverter and only
    succeeded with StringConverter

3.  It supports only gzip as a compression type

4.  It does not support moving keys or headers. There is a PR in line to
    move keys. You may need to build a custom transform to achieve it if
    that is your requirement
