# Why Zmq
Zmq has brokerless messaging architecture, where individual services talk to
each other directly in peer-to-peer model.

This brokerless architecture benifit by

1. No central broker, so no single point of failures
2. Reduced hope as each service directly to other

# How openstack implementation work

Every openstack node have one service called zmq_receiver, which listen on the
tcp port 9501 and accept all the messages on different topics and send it to
appropriate channels which is implemented as separate unix sockets - there are
separate unix sockets implemented per channel. 

Each openstack service has zmq driver implementation by impl_zmq driver for
oslo.messaging (impl_zmq.py). So each service will cast the messages using this
driver.

Zmq need to know what all nodes listen on different topics, openstack is
implemented this by using matchmaker. There are two matchmaker drivers for now -

1. ring driver - use json file installed on each node which resolve the topics
and the nodes
2. redis - use redis as backend for matchmaking.

# Jiocloud implementation

We use zmq for all services except neutron (and contrail) because contrail
doesnt support zeromq and they use rabbitmq. Also in case of undercloud, ironic
is also using rabbitmq.

We use ring driver for matchmaker. We have written puppet code to read consul
service catalog and update /etc/oslo/matchmaker_ring.json with matchmaking data.
So the matchmaker ring was updated as and when the consul service catalog got
changed (add/remove services to consul). We kept the puppet module which does
this here - https://github.com/JioCloud/puppet-openstack_zeromq.

