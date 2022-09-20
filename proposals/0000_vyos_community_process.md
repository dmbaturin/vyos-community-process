# VCP0: VyOS Community Process

## Introduction

The number of people working on VyOS is growing. That's a great thing,
but scaling a "small community" process to a "big community" one
requires more than just more participants: the process itself also needs improvement.

There are more and more cases when different people are working on the same area of code.
That can lead to massive waste of time and effort if two people send patches
with incompatible ideas or implementations.

Another factor is that design decisions in a project like VyOS are very difficult to change
after the fact — if something is in an LTS release, it stays there for years until its EOL.
To make better decisions, we need to give our ideas a broader discussion.

Another issue is that if a change comes as code, it's more difficult for non-programmers
to assess it. If a change is only presented in the form of code, then network admins
who aren't Python programmers and aren't familiar with our
command definition syntax have fewer chances to notice issues and help the developer fix them
before the code is merged.

Other projects where the community is large and where decisions have far-reaching
and long-lasting effects usually require contributors to submit a proporsal first.
Proposals are then publicly discussed, refined, and either accepted by the steering committee
or rejected.

Examples include Python PEPs, the Java Community Process and Scheme SRFIs.
I believe VyOS can benefit from a similar idea — let's tentatively call it
VyOS Community Processes.

## Goals

* Improve coordination and help people avoid wasting time on conflicting ideas or implementations.
* Make it easier for non-programmers to assess proposals.
* Ensure that the decision process and design decisions are recorded and documented.

## Proposal structure

Proposals should have following secsions:

* Introduction.
* Goals.
* Configuration commands (if any).
* Compatibility and migration considerations (if applicable).
* Operational mode commands (if any).
* Implementation considerations.
* Security implications.

Of course, proposals may deviate from that structure and omit sections or include additional sections.
Meta-proposals (like this one) certainly _will_ deviate from it — it's a guideline, not a requirement.

Introduction — describe your motivation for the proposal and outline the idea briefly
(or verbosely, if you are so inclinded ;).
If you are proposing a new feature, describe its purpose and usage context or provide links.

Goals — a short list of benefits to the VyOS project and its community that implementing your proposal
can bring.

Configuration commands — if your proposal adds any configuration commands, describe them in a textual format.
For example: 

```
system
  options
    # Automatically reboot if a kernel panic occurs
    reboot-on-panic [valueless]

system
  config-management
    # Number of config revisions to store on the local drive
    commit-revisions [int]
```

Compatibility and migration — if you propose to change any existing syntax,
there will need to be a migration script to convert old configs to it.
List affected commands and outline the migration logic.

Operational mode commands — list all operational mode commands that you want to add or change.

Implementation considerations — some things to think of include:

* interactions with other components of the system,
* possible difficulties and unknowns,
* performance considerations.

Security implications — describe how your change will affect security and what measures users may need
to keep their system secure.

## Proposal lifecycle

* Find the highest existing proposal number in the `proposals` directory of [vyos/vyos-community_process] (suppose it's 8999).
* Write your proposal in Markdown and save it to a file named `$number_$description.md` (like `9999_rewrite_vyos_in_visual_basic.md`).
* Submit a pull request to [vyos/vyos-community-process].
* Community members and maintainers will leave their comments and discuss the proposal.
* If the pull request is merged, that's a green light for implementing the proposal.

Note: it would be nice to allow people to participate without having to use GitHub. We'll need to figure out the best way to do it.

## When you don't need a proposal?

You certainly don't need to make a proposal if your change:

* fixes a bug that has has a clear cause and no side effects;
* exposes a new option in an already-implemented feature;
* improves performance without making any functional changes;
* doesn't have any observable effects and only involves refactoring.

If so, just go ahead and make a patch.
