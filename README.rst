====================================================
`c-bastion` -- Cloud Bastion Host (a.k.a. jump-host)
====================================================

About
=====

``c-bastion`` allows you to create users in a docker container via a CLI tool
and then log in with these formerly created users. The CLI tool is called
``cbas`` and can be found at:

https://github.com/immobilienscout24/cbas

The purpose of this project is to provide an EC2 instance with a Docker
container that can be used as a jump-host with dynamically created user. After
creation, the user is able to log in into the Docker container by using his/her
given SSH key.


Architecture
============

The main server-code is written Python and everything is packaged as a Docker
container. Inside the Docker container we use 
`Supervisor <http://supervisord.org/>`_ to run multiple processes. You may
inspect the file ``supervisord.conf`` to see the how it has been configured
(and where the log files are).

Configuration
=============

To run the jump-host you will need a so called auth-server that can provide
`OpenID Connect <http://openid.net/connect/>`_
`Json Web Tokens <http://jwt.io/>`_. To configure this server, please supply
the URL via an environment variable called ``AUTH_URL``. Note, that is
configured as a full url *including* the protocol and *without* a trailing
slash, e.g. ``http://auth-server.example``.

Docker Image Availability
=========================

The project is built with continuous integration on `Travis CI
<https://travis-ci.org/>`_.  This tests the server code, builds the Docker
image and uploads it to `Docker Hub <https://hub.docker.com/>`_ via Travis.
Hence you may obtain the Docker image from our organization on Docker Hub:

https://hub.docker.com/r/immobilienscout24/cbastion/

Local Testing
=============

Either pull the image from Docker Hub:

.. code-block::

    $ docker pull immobilienscout24/cbastion

You can then launch the Docker image using, note how the ``AUTH_URL`` is
supplied:

.. code-block::

    $ docker run -p 127.0.0.1:8080:8080 -e AUTH_URL=http://auth-server.example immobilienscout24/cbastion:latest

And finally, check that the container has started and that the jump-host server
has come up:

.. code-block::

   $ curl http://127.0.0.1:8080/status
   OK

API
===

There are a total of three endpoints:

:``status``: Check if the server is up and running.
:``create``: Upload ssh-key-file and create user.
:``delete``: Delete the user again.

Note however, that the preferred way to interact with the server is ``cbas``.

CFN STACK
=========

The repo holds a ``test-stacks.yml`` file to create an AWS CFN stack with using
an EC2 instance with ``taupage``, in which a Docker container is set up to serve
as the jump-host. Make sure to edit both the ``test-stacks.yml`` and
``template.yml`` (included by the former) before syncing it. The templates are in
YAML format, understood by ``cfn-sphere``.

Create/sync your AWS CFN stack:

- edit `test-stacks.yml` and `template.yml` to fit your needs
- sync the stack with `cf sync --confirm --parameter <your-preferred-stack-name>.dockerImageVersion=$VERSION test-stacks.yml`


License
=======

Copyright 2016 Immobilien Scout GmbH

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License. You may obtain a copy of the
License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed
under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.