= Couchbase Elasticsearch Connector

https://docs.couchbase.com/elasticsearch-connector/4.4/release-notes.html[*Download*]
| https://docs.couchbase.com/elasticsearch-connector/4.4/index.html[*Documentation*]
| https://issues.couchbase.com/projects/CBES[*Issues*]
| https://forums.couchbase.com/c/elasticsearch-connector[*Discussion*]

The Couchbase Elasticsearch Connector replicates your documents from Couchbase Server to Elasticsearch in near real time.
The connector uses the high-performance Database Change Protocol (DCP) to receive notifications when documents change in Couchbase.

NOTE: If you're looking for the Elasticsearch Plug-in flavor of the connector, that's in a https://github.com/couchbase/couchbase-elasticsearch-connector/tree/release/cypress[different branch].

[small]_This product is neither affiliated with nor endorsed by Elastic.
Elasticsearch is a trademark of Elasticsearch BV, registered in the U.S. and in other countries._

== Building the connector from source

The connector distribution may be built from source with the command:

    ./gradlew build

The distribution archive will be generated under `build/distributions`.
During development, it might be more convenient to run:

    ./gradlew installDist

which creates `build/install/couchbase-elasticsearch-connector` as a `$CBES_HOME` directory.


=== Running the integration tests

A local Docker installation is required for these tests.
To quickly test using only the latest Couchbase and Elasticsearch:

    ./gradlew integrationTest


To test against _all_ supported versions of Couchbase and Elasticsearch:

    ./gradlew exhaustiveTest


=== IntelliJ IDEA setup
Because the project uses annotation processors, some link:INTELLIJ-SETUP.md[fiddly setup] is required when importing the project into IntelliJ IDEA.


=== Building a Docker image

Use `Dockerfile` to build a Docker image from source using Gradle.
The version should be set in `build.gradle` before running.

    docker build -t imagename:tag .

Use `Dockerfile.download` to build a Docker image from released binaries hosted at packages.couchbase.com.

    docker build -f Dockerfile.download -t imagename:tag --build-arg VERSION=<version>

where `<version>` is the latest https://github.com/couchbase/couchbase-elasticsearch-connector/tags[tag] from the connector's GitHub repo.

=== Running a Docker image

The built docker image can be configured using volume mounts.
The `/opt/couchbase-elasticsearch-connector/config` directory should contain the configuration files, and the `/opt/couchbase-elasticsearch-connector/secrets` directory should contain the secrets.

Find example configuration files in the `src/dist` directory.
Be sure to rename `example-connector.toml` to `default-connector.toml`.

    docker run -p 31415:31415 -v ./config:/opt/couchbase-elasticsearch-connector/config -v ./secrets:/opt/couchbase-elasticsearch-connector/secrets -e CBES_GROUPNAME=groupname image:tag

It is also valid to pass environment variables in via the Docker command line, which can then be used to substitute values in `default-connector.toml`.
Port 31415 can be accessed via HTTP to get metrics.

=== Running in Kubernetes

The connector can run in Kubernetes.
See the examples in the `examples/kubernetes` directory, and https://docs.couchbase.com/elasticsearch-connector/current/kubernetes.html[the documentation] for more details.