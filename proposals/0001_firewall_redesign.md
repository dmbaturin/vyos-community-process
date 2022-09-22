# VyOS 1.4 Firewall Redesign

## Introduction

### Interface-based model is flawed

The current firewall syntax was designed in the early days of Vyatta, seemingly with a goal to mimic Cisco IOS:
every network interface in the config has subcommands for assigning firewall rulesets for different directions (in, out, local).

There are multiple problems with that approach:

First, It isn’t how netfilter works, so configuration scripts need to generate intentionally unidiomatic rules (as in, hardly any Linux user would write firewall rules that way).

Second, that model breaks down when it comes to interfaces created dynamically or on demand. Cisco solves that problem by allowing the user to create "virtual templates"
for dynamically created interfaces, but we certainly don’t want to do that — it’s both ugly and hard to implement.

Third, that model also doesn’t fit netfilter’s “two-axis” model where directions (in, out, forward) and tables/hooks (filter, mangle, bridge…) are independent.

### Interface-based model is already violated in the current CLI

Even in Vyatta times, people created commands outside of the interface-based model to work around its limitations:

```
set firewall ping/all-ping/broadcast-ping
set firewall state-policy
set interfaces … adjust-mss
```

Does anyone need any more proof that the interface-based model is flawed and needs to be replaced completely? ;)

But it gets worse.

### `set policy route` is a misnomer

Until Vyatta 6.4, there were “firewall name” and “firewall modify” subtrees. The architects at Vyatta inc. were likely aware how weird it was starting to sound and decided to rename “set firewall modify” to “set policy route”.
That made things much worse:

* The subtree that is allegedly about routing policy includes commands for TCP MSS clamping and setting netfilter marks — features that have nothing to do with routing whatsoever.
* The fact that it’s named “policy route” discourages adding support for more packet mangling features.
* Users need to learn that part of the functionality is in a completely different subtree.

### Zone-based firewall is a workaround for a self-inflicted problem

Now this may be a controversial idea, but there's a good chance that zone-based firewall is a wrong abstraction.
It's better than the "ACL on every interface" model for sure.

* Most setups only have a few zones, often just "LAN", "DMZ", and "WAN".
* Multi-tenant setups should better use VRFs for isolation — it's both easier and allows many more possibilities (like overlapping subnets for different tenants).
* Interface-less, zone-less firewall allows expressing every configuration as easily.

Thus zone-based firewall should be removed and migrated to the new syntax on upgrade.

## Goals

* Get rid of the leaky abstraction of the interface-based firewall.
* Allow users to express a full range of filtering criteria (now you can’t say "any traffic to destination port 80", for example — you have to assign a ruleset with that rule to every network interface where you want it to apply).
* Open up a path to supporting all netfilter features in our CLI.
* Merge different subsystems that duplicate each other (“firewall” and “policy route”).
* Define correct place and structure for upcoming new features, so cli can get reacher and mor flexible, without another big re-writting.
* Be able to write more optimal configuration, with less jumps (for example apply all filtering in forward hook).

## Configuration commands

```
firewall
    group [address|domain|ipv6-address|ipv6-network|mac|network|port|interface]
    state-policy
    filter
        ipv4-filter
            name <name>
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            forward
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            input
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            output
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            flowtables|offloads|acceleration…
        ipv6-filter
            name <name>
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            forward
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            input
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            output
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
        Inet-filter|ipv4_and_ipv6_filter
            name <name>
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            forward
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            input
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            output
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
        bridge-filter
            name <name>
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            forward
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            input
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            output
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            flowtables|offloads|acceleration…
        arp-filter
            ruleset <name>
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            input
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
            output
                default-action
                default-jump-target
                description
                enable-default-log
                rule <number> …
        netdev|ingress
        .
        .
        .
    policy|modify|mangle
    .
    .
    .
```

## Migration

Migration scripts will have to do quite a lot of work to translate old interface-based and zone-based firewalls
but so far I cannot see any non-migratable configurations.

## Operational commands

`* firewall name/ipv6 name` command must be renamed to "ruleset" instead.
