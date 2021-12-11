# Matter Ontology in TypeDB
An attempt at translating Matter Ontology from its original OWL version into Vaticle TypeDB with usability in mind.

## Summary

// TODO: A list of main ideas concerning classification and schema design pitfalls. 

## Overview of the Core Schema
![mo_core](https://user-images.githubusercontent.com/56684558/145677602-384ed825-db17-4008-9010-fd709ce0a0be.png)

For the justification of design choices in Matter Ontology, see the [owl directory](https://github.com/matterpale/matter-ontology/tree/main/owl).

### Attributes
For easy navigation within queries, Matter Ontology uses forced snake_case in naming attributes, often using indefinite articles.

```typeql
a_tag sub attribute, abstract,          value string;

a_type sub attribute, abstract,         value string;

a_location sub attribute,               value string,
    owns a_location;

occurs_at sub attribute,                value datetime;

an_amount sub attribute,                value long;

amount_unit sub attribute,              value string;
```

#### Tag
An abstract string super-attribute for any sort of identifiers, e.g., names (first or last), nicknames, uuids etc.

#### Type (CATEGORY?)
An abstract string super-attribute for classification of data into categories, e.g., types of payment, user access, material etc.

#### Location
Matter Ontology describes locations with a transitive (see rules) string attribute, format and uniqueness is left to the user. In an example implementation, one might introduce latitude and longitude as string attributes and have `a_location` own these.

#### Occurs at
A basic datetime attribute for description of processes, e.g. times of delivery, departure, arrival etc.

#### Amount & Amount Unit
Amounts of things (typically instances of resources) are modelled to be expressed with the integer (long) attribute `an_amount`, while also specifying given amount with another attribute `amount_unit`. For example, a bistro may have 26 litres of orange juice and 11 loafs of bread:


```typeql
insert
$j isa Juice, has an_amount 26, has amount_unit "l";   # L for litre
$b isa Bread, has an_amount 11, has amount_unit "pc";  # PC for piece, as in a loaf
```

IMPORTANT: In TypeDB, each attribute only has one instance of given value within the database. That means we cannot model `an-amount` to own `amount-unit` since then we would bind the `26` instance of `an_amount` to the `"l"` instance of `amount-unit` - then, if you also had 26 kilograms of sugar, there would be no direct way of determining the correct `amount_unit` for `26`.

### Entities
For easier navigation within queries, Matter Ontology uses PascalCase for entity names.
```typeql
ConcreteEntity sub entity, abstract,    # timespace-dependent physical entities
    owns a_name,
    owns a_location;

    Object sub ConcreteEntity;

    Process sub ConcreteEntity,         # a physical occurrence of an event
        owns occurs_at,
        plays duration:process;

Information sub entity, abstract,       # entities with loose timespace dependence               
    owns a_name;       
```

#### Concrete Entity
An abstract entity for representing all timespace-dependent physical entities of given domains.

#### Object
A tangible object, typicaly a solid item. It may prove more convenient to represent fluids with the help of `a_type` attribute and carrying/containing relations for its potentially frequent changes of multitude and location.

#### Process
An entity for representing physical occurrences of events.

#### Information
Information may or may not relate to time-space but it is not a tangible part of it in any case. For example, if we describe a recipe (perhaps as a relation relating ingredients with tools etc.), one of the ingredients may look something like "2 cups of wholewheat flour". Now, there may be no physical instance of this ingredient present in the described system, it is merely an idea, a description of the kind of flour (`a_type`) and its `an_amount` and `amount_unit`:
```typeql
define
bakingRecipe sub relation, relates ingredient;
BakingIngredient sub Information, owns an_amount, owns amount_unit, plays bakingRecipe:ingredient;
```

### Relations
For easier navigation within queries, Matter Ontology uses camelCase for relation names and kebab-case for roles. None of these is forced because roles only have meaning within their relation's context, so single-word naming should not lead to ambiguities.

```typeql
duration sub relation,
    owns an_amount,                     # length of time
    owns amount_unit,                   # time unit
    relates process;   
```

#### Duration
The only relation in the core Matter Ontology is the one setting basic context for time intervals. In its basic form, `duration` only relates an instance of a process and together with `occurs_at` as potential starting point allows for interval specification. Obviously, in timetable scenarios one would rather introduce a `start_date` and `end_date` - that is in no conflict with Matter Ontology.

### Rules
Matter Ontology uses kebab-case for rules. Once defined, rules are never explicitly operated, so confusion with relation roles should never arise.

```typeql
rule location-transitivity:
when {
    $x has a_location $y;
    $y has a_location $z;
} then {
    $x has $z;
};
```

#### Location Transitivity
This rule assures transitivity among locations, e.g., if a town has `a_location` a region which in turn has `a_location` a country, then the town naturally finds itself in the country as well. If a user needs to distinguish between these locations and only filter out, say, the country, one can simply introduce a `location_type sub a_type` and distinguish locations using this new attribute.

### Left Behind

#### Abstract Things
Thanks to the instantiation of all things (attributes, entities & relations) in TypeDB, there is little reason to introduce Form or Quality explicitly because those can be both easily represented by attributes. Since these attributes are instances of the database, they can be directly referred to in rules and queries and thus used for inference.

#### Roles
Thanks to the concept of roles being integral to TypeDB, there is no need for introducing them explicitly as entities. In TypeDB, anything that participates in an instance of a relation has to play a specific role within it.