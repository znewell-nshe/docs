***************************************
Deploying a Central Measurement Archive
***************************************

Measurement results from your regular tests are stored in a **measurement archive (MA)**. This leads to the definition of two categories of hosts:

#. The **measurement host** that runs regular tests such as throughput, one-way latency, ping and traceroute measurements.
#. The **archive host** that stores the results from the measurement host(s)

By default the perfSONAR Toolkit is both measurement host AND archive host. It comes installed with both the regular testing infrastructure and the measurement archive software. The measurement archive stores all the results from measurements defined in the regular testing configuration of the local host. In many cases, this is an adequate storage strategy. 

As an alternative, you may decide to separate your measurement hosts and archive host(s). This allows a single "centralized" archive host to store results from multiple measurement hosts. A few use cases where this may be desirable are as follows:

* Your measurement hosts are on less powerful (and likely lower cost) hardware that you don't want having the overhead of running the measurement archive. The measurement archive has higher hardware requirements than the measurement tools, so removing it can result in considerable resource savings.
* Your site has the experience and the infrastructure to manage large databases that you would like to leverage for storing measurement results on a dedicated archive host.
* You want to store results in multiple places. For example, you may store your results locally but also store them in a central archive where analysis tools operate on all the data from multiple hosts in one location.

Those are just a few examples. If you think a central measurement archive is right for your measurement strategy, then read the remainder of this document for instructions on how to configure both your archive host and measurement hosts.

Installing the Measurement Archive Software
============================================
The measurement archive is implemented using a software package named *perfsonar-archive*. You can install as a standalone package on any perfSONAR supported operating system with the following:

  *CentOS/RedHat:*::

    yum install perfsonar-archive

  *Debian/Ubuntu:*::

     apt-get install perfsonar-archive

Authenticating Measurement Hosts
================================
Measurement hosts need to authenticate to the measurement archive when they register data. This is implemented using one of two methods:

#. A username and password that is passed in the HTTP header of all requests.
#. Using IP based authentication

The remainder of this section describes how to setup both type of accounts.

.. note:: You may have a combination of the above where some accounts authenticate based on IP address and others based on username/password.

.. _multi_ma_install-intro:

Authenticating by Username and Password
----------------------------------------

The username and password is set in OpenSearch. You can create your own but an account is enabled with the minimum set of permissions needed to write to the perfSONAR indices. You can find the details in `/etc/perfsonar/opensearch/logstash_login`.

.. _multi_ma_install-auth_ip:

Authenticating by IP Address
----------------------------

This is done in the Apache proxy for Logstash. See the examples in */etc/httpd/conf.d/apache-logstash.conf*.

Configuring Measurement Hosts
==============================
Each measurement host must be configured to register its data to the central archive. This is done by configuring the :doc:`pSConfig pScheduler Agent <psconfig_pscheduler_agent>` to use the archive. Specifically, it configures the HTTP archiver plugin supported by pScheduler. The full set of options for the HTTP archiver plugin are detailed :ref:`here <pscheduler_ref_archivers-archivers-http>`. The exact approach depends largely on how you are reading your pSConfig template and which tasks you want to send to the central archive. 

Some common scenarios are listed below:

Scenario 1: Writing to a Local Archive
-----------------------------------------------------------------------
Note that by default, a perfSONAR Toolkit writes all tests to a local archive so no additional steps are needed. The archive definition can be found in `/etc/perfsonar/psconfig/archives.d/http_logstash.json`. If you need to regenerate that configuration, you can run the following command::

    /usr/lib/perfsonar/archive/perfsonar-scripts/psconfig_archive.sh

The output of the command will look similar to the following::

    {
        "archiver": "http",
        "data": {
            "schema": 2,
            "_url": "https://{% scheduled_by_address %}/logstash",
            "op": "put",
            "_headers": {
                "x-ps-observer": "{% scheduled_by_address %}",
                "content-type": "application/json", 
                "Authorization":"Basic eXamPleTokEn"
            }
        }
    }

Copy the above output to `/etc/perfsonar/psconfig/archives.d/http_logstash.json` and all pSConfig tests will write to the local archive.

Scenario 2: Writing to a Remote Archive with Username and Password Authentication
--------------------------------------------------------------------------------------------

As an example, let's say we want all our measurement hosts to write results to an archive with the hostname `example.archive`. We can generate a configuration with the following command where the hostname is given as a parameter to the script::

    /usr/lib/perfsonar/archive/perfsonar-scripts/psconfig_archive.sh -n example.archive

The output looks as follows::

    {
        "archiver": "http",
        "data": {
            "schema": 3,
            "_url": "https://example.archive/logstash",
            "verify-ssl": false,
            "op": "put",
            "_headers": {
                "x-ps-observer": "{% scheduled_by_address %}",
                "content-type": "application/json", 
                "Authorization":"Basic eXamPleTokEn"
            },
            "_meta": {
                "esmond_url": "https://example.archive/esmond/perfsonar/archive/"
            }
        }
    }

Note some differences include the URL pointing at the provided host, SSL verification is disabled by default (assumes a self-signed certificate), and an `esmond_url` is generated which can be used by tools like MaDDash to grab data using the legacy Esmond interface. Note this `esmond_url` will never be used by measurement clients to write results, it is only for querying using the backward compatibility interface. 

The simplest method for using the above output is to add it to a new JSON file in the `/etc/perfsonar/psconfig/archives.d/` directory.  See :ref:`psconfig_pscheduler_agent-templates-local` for more information on how to setup local templates.

**It is only recommended you set the** ``Authorization`` **field if your template is a file on the local system that will never be published to the web.** This is to protect your credentials from being exposed. If you would like to use username and password authentication while still publishing to a public pSConfig template, you can remove the Authorization field from the definition published either by hand or re-running the above command with `/usr/lib/perfsonar/archive/perfsonar-scripts/psconfig_archive.sh -a none -n example.archive`. You can then add it back locally using a transform script. See  :ref:`psconfig_pscheduler_agent-modify-transform_all` and :ref:`psconfig_pscheduler_agent-modify-transform_one` for examples of how to do this.

Scenario 3: Writing to a Remote Archive with IP Authentication
---------------------------------------------------------------------------

In this scenario you have configured a list of allowed IPs in */etc/httpd/conf.d/apache-logstash.conf*. You can safely publish to the web without authentication info. Generate the archiver configuration with the following command::

    /usr/lib/perfsonar/archive/perfsonar-scripts/psconfig_archive.sh -a none -n example.archive

The output should look something like the following::

    {
        "archiver": "http",
        "data": {
            "schema": 3,
            "_url": "https://example.archive/logstash",
            "verify-ssl": false,
            "op": "put",
            "_headers": {
                "x-ps-observer": "{% scheduled_by_address %}",
                "content-type": "application/json"
            },
            "_meta": {
                "esmond_url": "https://example.archive/esmond/perfsonar/archive/"
            }
        }
    }

Copy above to your central pSConfig template and your measurement hosts should begin archiving.


Legacy Installation: Writing to both Esmond and OpenSearch
==================================================================

In perfSONAR 5.0, the legacy archive named Esmond is dropped in favor of a new default archiver using OpenSearch. Since Esmond data is not migrated to OpenSearch, some users may find it beneficial to write to both archivers in parallel for a period of time. Assuming you have an existing Esmond installation and a new OpenSearch installation, you can continue to have measurement hosts write to each. **Given that Esmond will not be supported long-term by the perfSONAR project, we recommend you consider this a temporary measure and have a plan to retire your Esmond installation after a few months.** 

The biggest decision point for this is where you would like existing MaDDash instances to retrieve data that it displays. Generally, if you have both defined in the same pSConfig Template, then MaDDash will pick the first it encounters when it displays data. If you desire to exercise greater control of the archiver MaDDash chooses, then you have multiple options. A few different common scenarios are detailed below.

Scenario 1: MaDDash Reads from Legacy Esmond 
----------------------------------------------------
In this case you will write to both Esmond and OpenSearch, but MaDDash will only read from Esmond. This may be a good choice if you want to display historical data prior to the 5.0 release alongside new data while simulataneously building a new (albeit undisplayed in MaDDash) history in OpenSearch.

The first step is to put both archiver definitions in your pSConfig template. This will vary depending on the authentication mechanism you use for each, but will likely be similar to the following::

    {
      "archives":{
          "esmond_example":{
            "archiver":"esmond",
            "data":{
                "measurement-agent":"{% scheduled_by_address %}",
                "url":"https://example.esmond/esmond/perfsonar/archive/"
            }
          },
          "opensearch_example":{
            "archiver":"http",
            "data":{
                "schema":3,
                "_url":"https://example.archive/logstash",
                "verify-ssl":false,
                "op":"put",
                "_headers":{
                  "x-ps-observer":"{% scheduled_by_address %}",
                  "content-type":"application/json"
                }
            }
          }
      },
      "addresses":{
          "ADDRESS1":{
            "address":"ADDRESS1"
          },
          "ADDRESS2":{
            "address":"ADDRESS2"
          }
      },
      "groups":{
          "group_example":{
            "type":"mesh",
            "addresses":[
                {
                  "name":"ADDRESS1"
                },
                {
                  "name":"ADDRESS2"
                }
            ]
          }
      },
      "tests":{
          "test_example":{
            "type":"throughput",
            "spec":{
                "source":"{% address[0] %}",
                "dest":"{% address[1] %}",
                "duration":"PT30S"
            }
          }
      },
      "schedules":{
          "schedule_example":{
            "repeat":"PT4H",
            "sliprand":true,
            "slip":"PT4H"
          }
      },
      "tasks":{
          "throughput_example":{
            "group":"group_example",
            "test":"test_example",
            "schedule":"schedule_example",
            "archives":[
                "esmond_example",
                "opensearch_example"
            ],
            "_meta":{
                "display-name":"Example Throughput Tests"
            }
          }
      }
    }

The critical piece is that you don't define an `esmond_url` in the `_meta` section for the HTTP archiver definition used to write to OpenSearch. This will force MaDDash to use the native Esmond archiver since it will not have a compatible way to communicate with the OpenSearch host.

Scenario 2: Having a MaDDash Read from Esmond and from OpenSearch
-----------------------------------------------------------------------------------------------------------
If you would like to have a single MaDDash instance that reads from both archives, you can do so but will need a second pSConfig template file. There are multiple variations to achieve this, but if you already have a pSConfig template like in Scenaro 1, you can point measurement hosts at that file as well as your MaDDash instance to get the data from Esmond. 

You can now define a second pSConfig template that is a copy of the first, but has the following changes:

1. The task name and `display-name` is unique to distinguish from the other meshes. For example, add a 'pS5' prefix.
2. Only define the HTTP archiver and make sure it has an `esmond_url` in the `_meta` section.
3. DO NOT point measurement hosts at this pSConfig temple or you will get duplicate testing. Just point MaDDash at this second template file.

An example that modifies the template from Scenario 1 is below::

    {
      "archives":{
          "opensearch_example":{
            "archiver":"http",
            "data":{
                "schema":3,
                "_url":"https://example.archive/logstash",
                "verify-ssl":false,
                "op":"put",
                "_headers":{
                  "x-ps-observer":"{% scheduled_by_address %}",
                  "content-type":"application/json"
                }
            },
            "_meta": {
                "esmond_url": "https://example.archive/esmond/perfsonar/archive/"
            }
          }
      },
      "addresses":{
          "ADDRESS1":{
            "address":"ADDRESS1"
          },
          "ADDRESS2":{
            "address":"ADDRESS2"
          }
      },
      "groups":{
          "group_example":{
            "type":"mesh",
            "addresses":[
                {
                  "name":"ADDRESS1"
                },
                {
                  "name":"ADDRESS2"
                }
            ]
          }
      },
      "tests":{
          "test_example":{
            "type":"throughput",
            "spec":{
                "source":"{% address[0] %}",
                "dest":"{% address[1] %}",
                "duration":"PT30S"
            }
          }
      },
      "schedules":{
          "schedule_example":{
            "repeat":"PT4H",
            "sliprand":true,
            "slip":"PT4H"
          }
      },
      "tasks":{
          "pS5_throughput_example":{
            "group":"group_example",
            "test":"test_example",
            "schedule":"schedule_example",
            "archives":[
                "opensearch_example"
            ],
            "_meta":{
                "display-name": "pS5 - Example Throughput Tests"
            }
          }
      }
    }

Run an additional `psconfig remote add` command or copy to `/etc/perfsonar/psconfig/maddash.d` on the MaDDash host to point at this mesh, and you should see results from both instances displayed. 