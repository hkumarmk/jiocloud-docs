# Why Zmq
Zmq has brokerless messaging architecture, where individual services talk to
each other directly in peer-to-peer model.

This brokerless architecture benifit us by:

1. No central broker, so no single point of failure
2. Reduced hope on messaging as each services talk to each other directly
without need to go through central broker.
3. Scalable - peer to peer messaging is more scalable than broker based
messaging.

# How openstack implementation work

Every openstack node have one service called zmq_receiver installed, which
listen on the tcp port 9501 and accept all the messages on different topics and
send it to appropriate channels - there are separate unix sockets implemented
per channel. In jiocloud implementation, the sockets are kept under
/var/run/openstack directory.

Each openstack service has zmq driver implementation by impl_zmq driver for
oslo.messaging (impl_zmq.py). So each service will cast the messages using this
driver.

Zmq need to know what all nodes listen on different topics, openstack is
implemented this by using matchmaker. There are two matchmaker drivers for now -

1. ring driver - use json file installed on each node which resolve the topics
and the nodes
2. redis - use redis as backend for matchmaking.

Note: I am working on a driver for consul which facilitates consul based
matchmaking in which case zmq driver will talk to consul directly to do
matchmaking. (Blueprint for that -
https://blueprints.launchpad.net/oslo.messaging/+spec/matchmaker-consul-driver)

# Jiocloud implementation

We use zmq for all services except neutron (and contrail) because contrail
doesnt support zeromq and they use rabbitmq. Also in case of undercloud, ironic
is also using rabbitmq.

We use ring driver for matchmaker. We have written puppet code to read consul
service catalog and update /etc/oslo/matchmaker_ring.json with matchmaking data.
So the matchmaker ring was updated as and when the consul service catalog got
changed (add/remove services to consul). We kept the puppet module which does
this here - https://github.com/JioCloud/puppet-openstack_zeromq.

