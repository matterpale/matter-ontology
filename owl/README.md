# Matter Ontology in OWL

The class hierarchy of the original version of OWL Matter Ontology designed in [Protégé](https://protege.stanford.edu/) (visualized with OntoGraf tool, blue lines represent subclass relation):

![mo_owl](https://user-images.githubusercontent.com/56684558/144194580-659915b2-3a31-4248-878c-660991979ee0.png)

## Overview

First, there is a top-level dichotomy between `Role` and `Basic Thing` (i.e., everything that is not merely a role within a context) inspired by [1]. `Basic Thing` is further classified into `Concrete Thing` and `Abstract Thing` (based on whether they're directly dependent on timespace or not), as well as `Location` and `Time` (traditional coordinates), and `Information`. `Abstract Thing` is further subclassed by `Form` and `Quality`, which should help to clarify the distinction between `Abstract Thing` and `Information`. These design choices were loosely inspired by other literature ([2], [3], [4]).

For detailed descriptions see [mo_core.owl](https://github.com/matterpale/matter-ontology/blob/main/owl/mo_core.owl).

Let us consider an example extension to illustrate the ideas of this design:

# Example Extension of Matter Ontology in OWL

A selection of the class hierarchy of an extension of OWL Matter Ontology for a toy truck production line test scenario (visualized with OntoGraf tool, blue lines represent subclass relation):

![mo_ext_owl](https://user-images.githubusercontent.com/56684558/144576832-7d73bb85-82b2-443c-825f-806325a3d382.png)

An obscurity surrounds particularly the subtree of `Abstract Thing`, i.e., `Quality`, `Form` and its subclasses. To clarify the motivation for this classification, let us compare this approach with the alternative:

To classify an instance in OWL, one typically creates a granular enough class taxonomy and instantiates a certain class. In this example, classes `Transport Operation` and `Assembly Operation` would be subclasses of `Process` (a `Concrete Thing`) rather than of `Process Form` (an `Abstract Thing`). However, in that case we lack a way of directly referencing the given type of operation within object properties (only instances can participate, not classes) and SWRL rules. We can define relationships between these classes with axiomatic restrictions, however, in terms of expressivity, that cannot fully substitute.

Also, the alternative approach leads to expansive class hierarchies as the domain granularizes - it seems more appropriate to me to try to populate primarily the set of instances rather than to bloat the taxonomy of concepts. On the other hand, this is not to argue for the extreme case of having every tangible object represented as an instance of `Object` itself and leaving classification entirely to object properties towards instances of `Form` and its subclasses. A balance of the two approaches is expected and will naturally differ depending on given domain specifics and personal preferences of the designer.

In the case of this particular extension, instances of the subclasses of `Abstract Thing` were utilized to allow Pellet reasoner to infer whether a given truck configuration `occurs assembled`, `can be assembled` or `can be delivered`, with the help of a set of SWRL rules. To check how that works, and for descriptions and more detail on the ontology, see [mo_ext_trucks.owl](https://github.com/matterpale/matter-ontology/blob/main/owl/mo_ext_trucks.owl).

##

This work was conducted using the Protégé resource, which is supported by grant GM10331601 from the National Institute of General Medical Sciences of the United States National Institutes of Health.

## Sources

[1] Riichiro Mizoguchi. Tutorial on Ontological Engineering: Part 3: Advanced
Course of Ontological Engineering. New Generation Computing, 22:193–220,
2004.

[2] Riichiro Mizoguchi. Tutorial on Ontological Egineering Part 2: Ontology
Development, Tools and Languages. New Generation Computing, 22:61–96,
2009.

[3] John F. Sowa. Knowledge Representation: Logical, Philosophical and Computational
Foundations. Brooks/Cole Publishing Co., USA, 1999.

[4] Aldo Gangemi. Ontology Design Patterns for Semantic Web Content. In
International Semantic Web Conference, 2005.
