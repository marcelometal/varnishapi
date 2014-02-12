Varnish service API for tsuru PaaS
==================================

.. image:: https://travis-ci.org/globocom/varnishapi.png?branch=master
   :target: https://travis-ci.org/globocom/varnishapi
Deploying the API
-----------------

First, let's create an app in tsuru, from the project root, execute the following:

.. highlight: bash

::

    % tsuru app-create varnishapi python
    % git remote add tsuru git@remote.sbrubles.com # the returned remote ;)
    % git push tsuru master

The push will return an error telling you that you can't push code before the
app unit is up, wait until your unit is in service, you can check with:


.. highlight: bash

::

    % tsuru app-list

When you get an output like this you can proceed to push.

.. highlight: bash

::

    +-------------+-------------------------+--------------------------------------+
    | Application | Units State Summary     | Address                              |
    +-------------+-------------------------+--------------------------------------+
    | your-app    | 1 of 1 units in-service | your-app.sa-east-1.elb.amazonaws.com |
    +-------------+-------------------------+--------------------------------------+

Now if you access our app endpoint at "/" (you can check with `tsuru app-info`
cmd) you should get a 404, which is right, since the API does not respond
through this url.

Alright, let's configure the application, it'll need to talk with EC2 API, and
it does so by using environment variables. Here's what you need:

.. highlight: bash

::

    % tsuru env-set EC2_ENDPOINT=https://ec2.amazonaws.com EC2_ACCESS_KEY=your-access-key EC2_SECRET_KEY=your-secret-key

In order to get Varnish running, you can provide an AMI or a list of packages
to install via user data. The AMI is specified via the ``AMI_ID`` environment
variable, while the packages are specified by the ``API_PACKAGES`` environment
variable. Users may specify both variables.

.. highlight: bash

::

    % tsuru env-set AMI_ID=your-ami-id API_PACKAGES=varnish vim-nox

Users may also specify a subnet for running with VPC. You can specify the
subnet ID via the ``SUBNET_ID`` environment variable.

.. highlight: bash

::

    % tsuru env-set SUBNET_ID=your-subnet-id

The API also needs a SSH key in order to comunicate with the service instances'
VMs. You will need to store the key in the application machine and export the
environment variable ``KEY_PATH``:

.. highlight: bash

::

    % tsuru env-set KEY_PATH=/home/ubuntu/.ssh/id_rsa.pub

One more thing: this API will use MongoDB to store information about instances,
the MongoDB endpoint and the database name is also controlled via environment
variables:

We're done with our API! Let's create the service in Tsuru.

Creating the Service
--------------------

First you'll have to change the ``manifest.yaml`` file located at the project
root of our application. Change the production endpoint to point to the
application address, your yaml should look like this:

.. highlight: yaml

::

    id: varnish
    endpoint:
      production: varnishapi-endpoint.com

Now let's tell tsuru it needs to registrate a new service, from the project
root run, using `crane
<http://godoc.org/github.com/globocom/tsuru/cmd/crane>`_:

.. highlight: bash

::

    % crane create manifest.yaml

And we're done!