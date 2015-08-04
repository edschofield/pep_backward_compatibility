PEP: A standard mechanism for backward compatibility
====================================================

PEP:	 NUMBER
Title:	 A standard mechanism for backward compatibility
Author:	 Ed Schofield <ed at pythoncharmers.com>
Status:	 Draft
Type:	 Process
Created: 04-Aug-2015



Scope and relation to other PEPs
--------------------------------

This PEP is complementary to PEPs 5, 236, and 387, and shares similar goals.

PEP 5, "Guidelines for Language Evolution", notes that "This PEP does not replace or preclude other compatibility strategies ..."

This PEP introduces an additional compatibility mechanism to that of PEP 236 in support of PEP 5. PEP 236, "Back to the __future__", introduced a mechanism for forward compatibility but noted that a new mechanism for backward compatibility was outside the scope of that PEP. This PEP introduces such a mechanism for backward compatibility.

PEP 387, "Backwards Compatibility Policy", 

Context
-------

From PEP 236: "From time to time, Python makes an incompatible change to the advertised semantics of core language constructs, or changes their accidental (implementation-dependent) behavior in some way. While this is never done capriciously, and is always done with the aim of improving the language over the long term, over the short term it's contentious and disrupting. PEP 5, Guidelines for Language Evolution, suggests ways to ease the pain, and this PEP [236] introduces some machinery in support of that."

Also from PEP 236: "The purpose of future_statement is to make life easier for people who keep current with the latest release in a timely fashion. We don't hate you if you don't, but your problems are much harder to solve, and somebody with those problems will need to write a PEP addressing them. future_statement is aimed at a different audience."


Motivation
----------

When an incompatible change to core language syntax or semantics is being made, Python currently provides the future_statement mechanism for providing forward compatibility until the release that enforces the new syntax or semantics, but provides no corresponding standard mechanism for providing backward compatibility after this release.


Problem
-------

A consequence of this asymmetry is that, with respect to a breaking change, the older (pre-breaking) version of the Python interpreter is more capable than the newer (breaking) version; the older interpreter can use both modules designed prior to the change and newer modules, whereas the newer interpreter is only capable of using modules that have been upgraded to support the changed feature.

As an example, consider the changes to the division operator introduced in PEP 238 in 2001, soon after PEP 236 introduced the future_statement mechanism. PEP 238 outlines a suite of useful forward-compatibility mechanisms for "true division" in the Python 2.x series but omits to include any backward-compatibility mechanisms for after "true division" is first enforced in Python 3.0. Python versions since 3.0 do not provide a backward compatibility mechanism such as ``from __past__ import divisio`` for code that expects the old "classic division" semantics, whereas Python versions prior to 3.0 do support both "classic division" code and also forward compatibility with code expecting "true division". A further consequence of this is that the "most compatible" interpreter with respect to the variety of division-related Python code contained in books, blog posts, PyPI packages, and in-house codebases, is the one before the breaking change is first enforced.


Backward compatibility as enabler for "downhill upgrades"
---------------------------------------------------------

In contrast to this situation, newer versions of application software such as office suites tend to be more capable than earlier versions with respect to their support for loading different versions of their data file formats. The pattern is usually that the newer application versions can transparently load data from either their newer or their older data formats, and that the newer version defaults to saving data in the newer format. Newer application software versions tend to be backward-compatible by default. Forward compatibility is relatively rare.

This policy puts the user of the newer application software at an advantage over the user of the older software, which is usually incapable of loading data in the newer format. Sometimes it is possible for a user of a newer software application version to export data in an older version by choosing this option explicitly. In these cases, the forward-compatibility this enables may or may not be perfect; some features may be missing or the results may be otherwise suboptimal. Upgrading is therefore easy, whereas downgrading is harder.

The emergent behaviour from a policy of backward compatibility over many users is that a natural pressure builds up on each individual user to upgrade his or her own application version, and, the more other users an individual exchanges data files with, the more acute this pressure becomes.


Proposal - part 1
-----------------

This PEP makes two specific proposals. The first is:

1. PEP 5 be augmented with a 6th step in the section "Steps for Introducing Backwards-Incompatible Features" to indicate that, when an incompatible change to core language syntax or semantics is being made, Python-dev's policy is to prefer and expect that, wherever possible, a mechanism for backward compatibility be provided for future versions after the breaking change is adopted by default, in addition to any mechanisms for forward compatibility such as the future_statement. (PEP 387 could also be augmented with the same 6th step.)

Example
~~~~~~~

As an example of how this PEP is to be applied, if the latest revision of the "true division" PEP (238) were proposed today, it would be considered incomplete. PEP 238 describes several measures for forward compatibility in the Abstract and API Changes sections, and mentions some backward compatibility ideas raised on c.l.py, including "Use `from __past__ import division` to use classic division semantics in a module", but it does not put forward as part of the proposal  any mechanisms for backward compatibility after the incompatible change lands in Python 3.0.


Proposal - part 2
-----------------

The second proposal is that:

2. Python provide a standard backward compatibility mechanism in parallel to the __future__ module mechanism for forward compatibility.

For reference, this document will refer to this as a "__past__ mechanism" hereon, although it need not have all the characteristics of the ``__future__`` module and ``future_statement`` mechanism.

This PEP further proposes a design for the ``__past__`` mechanism be designed around similar criteria to those outlined in PEP 296. Specifically:

a. As with the ``__future__`` module, it should enable individual modules to specify obsolete behaviours to re-enable from older Python versions on a module-by-module basis.

b. It should allow both Python 3.6+ and point releases of earlier versions where appropriate to reintroduce backward compatibility for older syntax or semantics.

c. It should be possible to run code that specifies such behaviours on older Python versions that have no knowledge of the specific ``__past__`` features invoked, or even that the ``__past__`` mechanism for backward-compatibility exists.

Counter-Examples
~~~~~~~~~~~~~~~~

The specific form and implementation of the ``__past__`` mechanism is out of
scope for this PEP. (Another PEP is in progress for this.) However, some counter-examples which violate these criteria might be mechanisms based on:

a. Import hooks. These would normally fail to work on a module-by-module basis, and would instead apply recursively to all new modules imported from within a module.

b. A new piece of syntax or new semantics for Python 3.6 that is incompatible with prior versions.

c. Functionality added to an existing module in the Python standard library.


Proposal - part 3 - recommended practices for supporting backward compatibility
-------------------------------------------------------------------------------



Benefits
--------

One benefit to Python-dev of adopting this policy is that proposed backward-incompatible features for which a corresponding ``__past__`` feature has been implemented need not wait as many years to become active by default in a new Python version as when no such ``__past__`` feature is available, because they are less disruptive to the community. Adopting this policy could prevent Python from ever needing a 4.0 release that breaks compatibility with the 3.x series in an irreversible way. [Ref: Py4k]

The benefit to the conservative or "lazy" user is obvious: they can add support for the latest shiny compatibility-breaking Python version to their code merely by adding a ``__past__`` incantation (perhaps a single line) to each module, and that this can be automated. They can then upgrade their interpreter to the latest version and gain access to the latest shiny Python features.

The benefit to the community is that, if ten thousand users rely on package XYZ, and package XYZ can trivially add support for the latest Python version, those ten thousand users can also upgrade to the latest Python version quickly, without being held back waiting for package XYZ to do this.


Questions and answers
---------------------

Q1: Won't backward compatibility features lead to lots of cruft and bloat and baggage in Python?

A1: Not necessarily. First, proposals for new compatibility-breaking features in Python could be evaluated partly on the simplicity and maintainability of the implementation of their associated ``__past__`` feature up-front.

Second, some old features are simple to provide backward compatibility for. Consider the "classic division" behaviour before Python 3.0. The ``python-future`` project contains a compatible implementation of classic division in the function ``future.utils.old_div``:

```
def old_div(a, b):
    """
    Equivalent to ``a / b`` on Python 2 without ``from __future__ import
    division``.
    """
    if isinstance(a, numbers.Integral) and isinstance(b, numbers.Integral):
        return a // b
    else:
        return a / b
```

Bundling such a function with Python 3.x versions, together with providing a simple mechanism to invoke it for every appearance of ``a / b`` would not be difficult.

Third, this PEP does not require that a feature once supported must be supported forever. Legacy features can be phased out when appropriate. Notice that reintroducing compatibility for non-nested scopes or classic classes or into Python 3.6 would likely help nobody.


Q2: But Python-dev is already overwhelmed and doesn't have the bandwidth to implement / maintain the additional complexity!

A2: Python-dev can ask the community of developers to step up and maintain backward compatibility in Python for legacy language features they care about. When the community stops caring, Python-dev can stop caring too. The ``__past__`` mechanism could also be designed to be extensible by the community to take the load off the core developers.

References
----------

[Py4k]: http://www.curiousefficiency.org/posts/2014/08/python-4000.html

[PyFut]: http://python-future.org

Source
------

https://github.com/edschofield/pep_backward_compatibility

