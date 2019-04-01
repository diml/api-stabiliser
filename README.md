An API stabilistion process
===========================

This document is a work in progress.

Abstract
--------

This document describes a process allowing to add a stable API with
clear guarantees on top of any library. The goal can be either to
provide guarantees for libraries that are inherently unstable, or to
increase the dynamism of an existing library by providing a
straightforward method to go through breaking API changes.

From the point of view of library authors, the method provides a
systematic way to go through breaking API change. It relieves the
library from having to do complex decisions when evolving the
library. It also take off the pressure from library authors of having
to come with the perfect design right from the start for new features,
given that it is often hard to do that before we have some feedback
from experience.

From the point of view of users, this method give them strong
guarantees as to how long code written against a stble API will be
buildable.

Definition of stable
--------------------

By stable we mean the following: any code that compiles without
deprecation warnings is guaranteed to continue building for the next
two years.

This guarantee only applies when all the depeendencies of the code use
the method described in this document.

With this guarantee, users always get a window of two years to upgrade
their code once a feature is deprecated.

The method
----------

The core idea is to take a snapshot of the API of the library every
two years. The library itself is unstable and provides no stability
guarantee, however the API of the snapshot is guaranteed forever
immutable. API snapshots should be distributed as separate
packages. This allows external users to take over the maintenance of
old snapshots once they stop being maintained by upstream.

Initially the snapshot will be just a thin layer on top of the main
library. As the library evolves and changes in breaking ways, the
snapshot might need to be updated in order to still provide the old
API in terms of the new one.

When creating snapshot `n+1`, we do the following: we make the
snapshot `n` depend on the `n+1` one instead of the main library. At
this point, this is a trivial operation given that `n+1` snapshot is
exactly the current API. This operation ensures that snapshot `n` will
be very-low maintenance from now on given that it is built on top of a
stable API.

Even though snapshots are meants to work forever, we recommend to
deprecate and retire them in order to avoid having an increasing
number of stable APIs to maintain. We suggest the following scheduler:
when creating snapshot `n+1`, deprecate snapshot `n` and delete
snapshot `n-1`.

API snapshots
-------------

The core of the method relies on taking API snapshots. In this section
we describe a few details that are important to respect when doing so.

### Preserving type equalities

It is important to preserve type equalities. I.e. it should almost
always be the case that a given type `M.t` should be equal in all
snapshots of the API. The exact type name might be different, as for
instance the module name might have been changed. However types that
logically represent the same thing should be exposed as equal between
snapshots.

This is to ensure that third-party libraries written against different
versions of this library can inter-operate. For instance, let's
imagine that we were applying this method to the standard library,
then it would be important that `Hashtbl.t` remains the same type at
all time.

### Making most types abstract

In order to make the previous rule viable, types that are not
completely standard and guaranteed to never changed should be made
abstract.

Types that do not need to be made abstract are types such as booleans,
the either type, the ordering type, s-expressions, ... Everything else
should be made abstract.

### Consuming abstract types

When all the types are abstract, we loose pattern matching. This can
be a problem. In the long term, we expect that a general solution to
this problem will be developed and integrated properly in the
language. In the meantime we propose to rely on the external
`ppx_view` rewriter which provides a large enough subset of pattern
matching features for abstract types.
