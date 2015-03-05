####
OpenStack Ceilometer Installation Guide for Juno release
####

Welcome to OpenStack Ceilometer installation manual !

This document is based on `the OpenStack Official Documentation <http://docs.openstack.org/juno/install-guide/install/apt/content/>`_ for Juno. 

================================

.. contents::

Ceilometer Overview
===================

In this guide, we will go over the installation of OpenStack Ceilometer service !  

The Telemetry module or Ceilometer performs the following functions:

+ Efficiently collects the metering data about the CPU and network costs.

+ Collects data by monitoring notifications sent from services or by polling the infrastructure.

+ Configures the type of collected data to meet various operating requirements. It accesses and inserts the metering data through the REST API.

+ Expands the framework to collect custom usage data by additional plug-ins.

+ Produces signed metering messages that cannot be repudiated. 

 Ceilometer has five basic components that communicate by using the OpenStack messaging bus: 

A **compute agent** Runs on each compute node and polls for resource utilization statistics. 

A **central agent** Runs on a central management server to poll for resource utilization statistics for resources not tied to instances or compute nodes.

A **collector** Runs on central management serverS to monitor the message queues (for notifications and for metering data coming from the agent). Notification messages are processed and turned into metering messages, which are sent to the message bus using the appropriate topic. Telemetry messages are written to the data store without modification.

A **data store** A database capable of handling concurrent writes (from one or more collector instances) and reads (from the API server).

An **API server** Runs on one or more central management servers to provide data access from the data store.


Ceilometer Install
==================
This section describes how to install and configure the Telemetry module on the controller node and the compute nodes.

Controller Node
---------------

* Change to super user mode::

    sudo su
    
* Install the MongoDB::
    
    apt-get install -y mongodb-server mongodb-clients python-pymongo
    
* Edit the /etc/mongodb.conf file::

    vi /etc/mongodb.conf
    bind_ip = 10.0.0.11
    smallfiles = true
    
* Restart the MongoDB service::

    service mongodb stop
    rm -f /var/lib/mongodb/journal/prealloc.*
    service mongodb start

* Create the ceilometer database::

    mongo --host controller --eval '
    db = db.getSiblingDB("ceilometer");
    db.addUser({user: "ceilometer",
    pwd: "CEILOMETER_DBPASS",
    roles: [ "readWrite", "dbAdmin" ]})'
    

* Configure service user and role::
    
    vi admin_creds
    #Paste the following:
    export OS_TENANT_NAME=admin
    export OS_USERNAME=admin
    export OS_PASSWORD=admin_pass
    export OS_AUTH_URL=http://controller:35357/v2.0
    
    
    source admin_creds

    keystone user-create --name ceilometer --pass service_pass
    keystone user-role-add --user ceilometer --tenant service --role admin


* Register the service and create the endpoint::
    
    keystone service-create --name ceilometer --type metering --description "Telemetry"
    
    keystone endpoint-create \
    --service-id $(keystone service-list | awk '/ metering / {print $2}') \
    --publicurl http://controller:8777 \
    --internalurl http://controller:8777 \
    --adminurl http://controller:8777 \
    --region regionOne


* Install ceilometer packages::

    apt-get install -y ceilometer-api ceilometer-collector ceilometer-agent-central ceilometer-agent-notification ceilometer-alarm-evaluator ceilometer-alarm-notifier python-ceilometerclient


* Edit the /etc/ceilometer/ceilometer.conf file::

    vi /etc/ceilometer/ceilometer.conf
   
    [database]
    replace connection=sqlite:////var/lib/ceilometer/ceilometer.sqlite with:
    connection = mongodb://ceilometer:CEILOMETER_DBPASS@controller:27017/ceilometer
  
    [DEFAULT]  
    verbose = True
    
    rpc_backend = rabbit
    rabbit_host = controller
    rabbit_password = service_pass
    
    auth_strategy = keystone
    
    [keystone_authtoken]
    auth_uri = http://controller:5000/v2.0
    identity_uri = http://controller:35357
    admin_tenant_name = service
    admin_user = ceilometer
    admin_password = service_pass
    
    [service_credentials]
    os_auth_url = http://controller:5000/v2.0
    os_username = ceilometer
    os_tenant_name = service
    os_password = service_pass
    
    [publisher]
    metering_secret = METERING_SECRET
    

* Restart the Telemetry services::

    service ceilometer-agent-central restart
    service ceilometer-agent-notification restart
    service ceilometer-api restart
    service ceilometer-collector restart
    service ceilometer-alarm-evaluator restart
    service ceilometer-alarm-notifier restart
    
    
* Configure the Image Service for Telemetry. Edit the /etc/glance/glance-api.conf file::

    vi /etc/glance/glance-api.conf
    [DEFAULT]
    notification_driver = messaging
    rpc_backend = rabbit
    rabbit_host = controller
    rabbit_password = service_pass

* Edit the /etc/glance/glance-registry.conf file::
    
    vi /etc/glance/glance-registry.conf
    [DEFAULT]
    notification_driver = messaging
    rpc_backend = rabbit
    rabbit_host = controller
    rabbit_password = service_pass
  
* Restart the Image Service::

    service glance-registry restart
    service glance-api restart



Compute Node
------------

* Change to super user mode::

    sudo su
    
* Install the ceilometer agent package::

    apt-get install -y ceilometer-agent-compute
    
* Edit the /etc/nova/nova.conf::

    vi /etc/nova/nova.conf
    [DEFAULT]
    instance_usage_audit = True
    instance_usage_audit_period = hour
    notify_on_state_change = vm_and_task_state
    notification_driver = nova.openstack.common.notifier.rpc_notifier
    notification_driver = ceilometer.compute.nova_notifier
    
* Restart the Compute service::

    service nova-compute restart
    
* Edit /etc/ceilometer/ceilometer.conf file::

    vi /etc/ceilometer/ceilometer.conf
    
    [DEFAULT]
    verbose = True
    
    rabbit_host = controller
    rabbit_password = service_pass

    [keystone_authtoken]
    auth_uri = http://controller:5000/v2.0
    identity_uri = http://controller:35357
    admin_tenant_name = service
    admin_user = ceilometer
    admin_password = service_pass
    
    [service_credentials]
    os_auth_url = http://controller:5000/v2.0
    os_username = ceilometer
    os_tenant_name = service
    os_password = service_pass
    os_endpoint_type = internalURL
    os_region_name = regionOne

    [publisher]
    metering_secret = METERING_SECRET
    
* Restart the Telemetry service::

    service ceilometer-agent-compute restart

That's it ;) 


License
=======
Institut Mines Télécom - Télécom SudParis  

Copyright (C) 2015  Authors

Original Authors -  Marouen Mechtri and  Chaima Ghribi 

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except 

in compliance with the License. You may obtain a copy of the License at::

    http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.


Authors
========

Copyright (C) `Marouen Mechtri <https://www.linkedin.com/in/mechtri>`_ : marouen.mechtri@it-sudparis.eu

Copyright (C) `Chaima Ghribi <https://www.linkedin.com/pub/chaima-ghribi/15/b78/997/>`_ : chaima.ghribi@it-sudparis.eu
