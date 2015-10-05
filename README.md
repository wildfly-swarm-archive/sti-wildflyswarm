Wildfly Swarm - CentOS Docker image
========================================

This repository contains the source for building various versions of
the WildFly Swarm application as a reproducible Docker image using
[source-to-image](https://github.com/openshift/source-to-image).  The
resulting image can be run using [Docker](http://docker.io).

Versions
---------------
WildFly Swarm versions currently provided are:
* Whatever version your application uses

CentOS versions currently provided are:
* CentOS7


Installation
---------------

To build a WildFly Swarm image from scratch, run:

```
$ git clone https://github.com/wildfly-swarm/sti-wildflyswarm.git
$ cd sti-wildflyswarm
$ make build
```

Standalone Usage
---------------------
To build a simple [jee application](https://github.com/bparees/openshift-jee-sample)
using standalone [S2I](https://github.com/openshift/source-to-image) and then run the
resulting image with [Docker](http://docker.io) execute:

```
$ s2i build git://github.com/bparees/openshift-jee-sample openshift/wildflyswarm-10-centos7 wildflyswarmtest
$ docker run -p 8080:8080 wildflyswarmtest
```

**Accessing the application:**
```
$ curl 127.0.0.1:8080
```

OpenShift 3 Usage
---------------------

First ensure you have a [working OpenShift 3
environment](http://www.openshift.org/) with the `oc` command in your
path.

```
$ oc create -f https://raw.githubusercontent.com/wildfly-swarm/sti-wildflyswarm/master/1.0/template.json
$ oc start-build wildflyswarm-10-centos7-build
$ oc new-app wildflyswarm-10-centos7~https://github.com/bbrowning/openshift-jee-sample
```

Visit your app's service host/port (given by `oc status`) to see the
sample app. The path `/snoop.jsp` will also test a JSP page and output
some server information.

Test
---------------------
This repository also provides a [S2I](https://github.com/openshift/source-to-image) test framework,
which launches tests to check functionality of a simple WildFly Swarm application built on top of the wildfly swarm image.

*  **CentOS based image**

    ```
    $ cd sti-wildflyswarm
    $ make test
    ```

Repository organization
------------------------
* **`<Image version>`**

    * **Dockerfile**

        CentOS based Dockerfile

    * **`s2i/bin/`**

        This folder contains scripts that are run by [S2I](https://github.com/openshift/source-to-image):

        *   **assemble**

          Is used to restore the build artifacts from the previous built (in case of
          'incremental build'), to install the sources into location from where the
          application will be run and prepare the application for deployment (eg.
          installing maven dependencies, building java code, etc..).


        *   **run**

          This script is responsible for running the application built
          using WildFly Swarm.

        *   **save-artifacts**

          In order to do an *incremental build* (iow. re-use the build artifacts
          from an already built image in a new image), this script is responsible for
          archiving those. In this image, this script will archive the
          maven dependencies and previously built java class files.

    * **`contrib/`**

        This folder contains commonly used modules

        * **`wfbin/`**

            Contains script used to launch wildfly after performing environment variable
            substitution into the standalone.xml configuration file.

    * **`test/`**

        This folder contains the [S2I](https://github.com/openshift/source-to-image)
        test framework with a simple JEE application.

        * **`test-app/`**

            A simple Node.JS echo server used for testing purposes by the [S2I](https://github.com/openshift/source-to-image) test framework.

        * **run**

            This script runs the [S2I](https://github.com/openshift/source-to-image) test framework.

* **`hack/`**

    Folder containing scripts which are responsible for the build and test actions performed by the `Makefile`.


Image name structure
------------------------
##### Structure: openshift/1-2-3

1. Platform name (lowercase) - wildflyswarm
2. Platform version(without dots) - 10
3. Base builder image - centos7

Example: `openshift/wildflyswarm-10-centos7`
Environment variables
---------------------
To set environment variables, you can place them as a key value pair into a `.sti/environment`
file inside your source code repository.

* MAVEN_ARGS

    Overrides the default arguments passed to maven durin the build process

* MAVEN_ARGS_APPEND

    This value will be appended to either the default maven arguments, or the value of MAVEN_ARGS if MAVEN_ARGS is set.

* MYSQL_DATABASE

    If set, WildFly will attempt to define a MySQL datasource based on the assumption you have an OpenShift service named "mysql" defined.
    It will attempt to reference the following environment variables which are automatically defined if the "mysql" service exists:
    MYSQL_SERVICE_PORT
    MYSQL_SERVICE_HOST
    MYSQL_PASSWORD
    MYSQL_USER

* POSTGRESQL_DATABASE

    If set, WildFly will attempt to define a PostgreSQL datasource based on the assumption you have an OpenShift service named "postgresql" defined.
    It will attempt to reference the following environment variables which are automatically defined if the "postgresql" service exists:
    POSTGRESQL_SERVICE_PORT
    POSTGRESQL_SERVICE_HOST
    POSTGRESQL_PASSWORD
    POSTGRESQL_USER


Copyright
--------------------

Released under the Apache License 2.0. See the [LICENSE](https://github.com/openshift/sti-wildfly/blob/master/LICENSE) file.
