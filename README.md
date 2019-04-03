Immutable versioning
====================

Immutable versioning is a very simple but powerful idea that aims to
solve versioning problems. In particular, it aims at making it easy to
go through breaking changes, which are a natural part of software
development. It can apply to many systems such as library APIs or even
languages. The only requirement is the ability to represent the
difference between two states of a system.

The problem
-----------

Let's consider a system that is continously evolving. Users of this
system might want to build more complex systems on top of this system,
and other users might want to build even more complex systems on top
of all this. For instance, the systems might be libraries.

As systems evolve and change, other systems built on top need to adapt
and change as well. If the various systems are built by independent
developers, keeping everything in sync can quickly become a challenge
as the number of systems increases.

We typically have this problem with libraries: whenever a library
changes in an incompatible way, each of its user need to upgrade. As
the number of libraries increases, it can become very difficult for a
community to cope with breaking changes in widely used libraries.

The solution
------------

The solution consist on taking regular snapshots of the system. A
snapshot is simply another independent system built on top of the main
one, except that it is guaranteed to never change and always be
equivalent to the main system at the time the snapshot was taken.

Initially, because the two systems are equivalent, the snapshot
requires no extra data. Its size is 0. As the main system evolves, the
snapshot will need to adapt in order to still look the same to its
users and its size will slowly increase.

In order to keep the operation sustainable as the number of snapshot
increases, we enforce that each snapshot is built on top of the next
one. More precisiely, let `S` be a system and `S<n>` be the last
snapshot of this system. When we take the next snapshot `S<n+1>`, we
change `S<n>` so that it is built on top of `S<n+1>` rather than
`S`. At this point, this is a straightforward operation because `S`
and `S<n+1>` are equivalent.

By construction, each snapshot but the last one are completely frozen
and never need to change. That means that the maintainers of the
system only need to focus on the system itself and the very last
snapshot.

In this model, older snapshots tend to be heavier than more recent
one. Indeed, the snapshot forms a linked chain of systems, each one
being built on the next one. Users who regularly put in the effort of
upgrading to recent snapshots are rewarded by relying on a small chain
of systems, which is likely to be more efficient. Users who do not
upgrade are penalised by relying on a long chain of linked systems.

Applying this idea to API versioning
------------------------------------

Applying this idea to API versioning means the following: consider a
library `Foo`. Whenever `Foo` is ready for a new stable release, we
simply create a new library `Foo_v<n>` that depends on `Foo` and is
equivalent to it, and make `Foo_v<n-1>` depend on `Foo_v<n>` rather
than `Foo`. At this point, `Foo_v<n-1>` will never change again and we
can simply delete it from the development repository. In order for
this to work well with package managers, we should release the various
`Foo_vXXX` libraries as separate packages.

Effectively, we are creating a chain of libraries such that:

- `Foo_v1` depends on `Foo_v2`
- `Foo_v2` depends on `Foo_v3`
...
- `Foo_v<n-1>` depends on `Foo_v<n>`
- `Foo_v<n>` depends on `Foo`

In this list, `Foo` is completely unstable and can change in any way
at any point. However, all the `Foo_vXXX` libraries are completely and
forever stable. Everytime one changes `Foo`, they also have to update
the last snapshot. In a way, we took away the burden of upgrading from
users and put the pressure on the person doing the breaking change.

With this model, we can forget about deprecation warnings. Users who
make the effort to upgrade are rewarded by faster compilation and
execution times since they have to go through less compatibility
layers.

In any case, several different versions of `Foo` can be linked in the
same executable since they are effectively different libraries. The
nice property is that maximum sharing is achived as each version of
`Foo` is not a full blown copy of `Foo` but rather a diff against the
next version.

When minting a stable version, we might also want to trim a bit the
API rather than take a full snapshot. For instance to hide private
functions that are only exposed for testing purposes.

Inter-operability
-----------------

One thing to consider is that it is likely that various users of `Foo`
might need to inter-operate with each other. This might be difficult
if the values manipulated by the various versions of `Foo` are not
compatible. In strongly typed languages, it is therefore important to
expose type equalities from one version of a type to the next version
as long as they are equivalent.

This as two consequences: types shouldn't be changed between versions
as otherwise it would hurt inter-operability. To make this viable,
most types should be made abstract, except for the ones that we know
will never change.

In languages that support pattern matching, it is important that the
language also support pattern matching on absract types, for instance
via _view patterns_.
