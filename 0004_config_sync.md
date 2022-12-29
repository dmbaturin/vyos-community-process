# VCP4: primary-replica config synchronization

Config sync is a frequently requested feature. The standard answer is that full-blown synchronziation
will be handled by a controller appliance. However, in small setups where configs are unique to the site
and that only have a few VyOS instances, a controller appliance is likely an overkill.

Thus there is a bitter spot where configs are large and fast-changing enough to make keeping them in sync
a hard problem but the network is not large enough to justify using a controller appliance.

It's obvious that _bi-directional_ synchronization is an undecidable problem. The proof is trivial:
suppose that since the last synchronization, Alice set the protocol in a firewall rule to TCP
and Bob now wants to set it to UDP. The only way to _meaningfully_ reconcile such changes
is for Alice and Bob to talk and decide what they want the rule to do.

The role of the controller appliance is to serve as a single source of truth in such situations
and instances managed by a controller must not be configured by hand
or the controller will override the local changes.

However, one possible solution for a controller-less _partial unidirectional_ synchronization
is to make _both_ the primary host and replicas aware of the paths that are synchronized.

## Example

On the primary:

```
set high-availability config-replication mode primary

set high-availability config-replication replicate path "firewall name WAN"
set high-availability config-replication replicate path "interfaces openvpn"

set high-availability config-replication replica Secondary address 192.0.2.2

set high-availability config-replication authentication pre-shared-secret qwerty
```


On the replica:

```
set high-availability config-replication mode replica

set high-availability config-replication replicate path "firewall name WAN"
set high-availability config-replication replicate path "interfaces openvpn"

set high-availability config-replication authentication pre-shared-secret qwerty

set high-availability config-replication primary address 192.0.2.1
```

Now trying to manually change anything under `firewall name WAN` on the replica will fail:

```
vyos@replica:~# set firewall name WAN default-action accept
vyos@replica:~# commit

Error: path "firewall name WAN" is managed by config replication and must not be modified locally.
```

## Unreachable replica problem

The main reason why the primary instance address is configured on replicas is so that they
can request the synchronized paths on boot to make sure that even in the worst case
when the primary repeatedly failed to push the config to them they will be eventually consistent.
