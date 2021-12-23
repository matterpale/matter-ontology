# Matter Ontology in TypeDB
An attempt at translating Matter Ontology from its original OWL version into Vaticle TypeDB with focus on usability.
## Summary
Here is a list of some of the things to keep in mind when adopting Matter Ontology, typically by extending it:
- Maintain a clear distinction between physically existent (`ConcreteEntity`) things and others, i.e., information, abstractions etc.
- Consider the context and use cases before picking between the two main approaches of extension:
  - define a new sub-attribute of `a_category` and then connect given data with its instances (by attribute ownership)
  - define a comprehensive subhierarchy of entities and use it to instantiate given data
- Keep in mind value uniqueness of attribute types, i.e., utilize its instances with caution, see [here](https://github.com/matterpale/matter-ontology/blob/main/vaticle/README.md#amount--amount-unit).
- Non-abstract types cannot own abstract attributes - relations can work around by setting the abstract attribute to play a role of theirs.
## Overview of the Core Schema
![mo_core](https://user-images.githubusercontent.com/56684558/145677602-384ed825-db17-4008-9010-fd709ce0a0be.png)
For the justification of design choices in Matter Ontology, see the [owl directory](https://github.com/matterpale/matter-ontology/tree/main/owl).
##
### Attributes
For easy navigation within queries, Matter Ontology uses forced *snake_case* in naming attributes, often using indefinite articles.
```typeql
a_tag sub attribute, abstract,          value string;

a_category sub attribute, abstract,     value string;

a_location sub attribute,               value string,
    owns a_location;

occurs_at sub attribute,                value datetime;

an_amount sub attribute,                value long;

amount_unit sub attribute,              value string;
```
#### Tag
An abstract string super-attribute for any sort of identifiers, e.g., names (first or last), nicknames, uuids etc.
#### Category
An abstract string super-attribute for classification of data into categories, e.g., types of payment, user access, material etc.
#### Location
Matter Ontology describes locations with a transitive (see rules) string attribute, format and uniqueness is left to the user. In an example implementation, one might introduce latitude and longitude as string attributes and have `a_location` own these.
#### Occurs at
A basic datetime attribute for description of processes, e.g. times of delivery, departure, arrival etc.
#### Amount & Amount Unit
Amounts of things (typically instances of resources) are modelled to be expressed with the integer (long) attribute `an_amount`, while also specifying given amount with another attribute `amount_unit`. For example, a bistro may have 26 litres of orange juice and 11 loafs of barley bread:
```typeql
define      ## schema write query
    food_item sub a_category, owns an_amount, owns amount_unit;

insert      ## data write query
    "orange juice" isa food_item, has an_amount 26, has amount_unit "litre";
    "barley bread" isa food_item, has an_amount 11, has amount_unit "piece";
```
IMPORTANT: In TypeDB, each attribute only has one instance of given value within the database, so we should not say `26 has amount_unit "litre"`, because if you also had 26 kilograms of sugar, there would be no direct way of determining the correct `amount_unit` for `26`. That means we cannot model `an_amount` to own `amount_unit`.
##
### Entities
For easier navigation within queries, Matter Ontology uses *PascalCase* for entity names.
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
An entity for representing all timespace-dependent physical entities of given domains. It is only abstract in the context of database, i.e., that it shall not be instantiated.
#### Object
A tangible object, typically a solid item, a piece of material or a living person. It may prove more convenient to represent fluids with the help of a sub-attribute of `a_category` and carry/contain relations for its potentially frequent changes of multitude and location, see [here]().
#### Process
An entity for representing physical occurrences of events.
#### Information
Information may or may not relate to time-space but it is not a tangible part of it.

For example, if we describe a recipe (perhaps as a relation relating ingredients with tools etc.), one of the ingredients may look something like "2 cups of wholewheat flour". Now, there may be no physical instance of this ingredient present in the described system, it is merely an idea, a description of the kind of flour (`a_category`) and its `an_amount` and `amount_unit`:
```typeql
define
bakingRecipe sub relation, relates ingredient;
BakingIngredient sub Information, owns an_amount, owns amount_unit, plays bakingRecipe:ingredient;
```
##
### Relations
For easier navigation within queries, Matter Ontology uses *camelCase* for relation names and kebab-case for roles. None of these is forced because roles only have meaning within their relation's context, so single-word naming should not lead to ambiguities.
```typeql
duration sub relation,
    owns an_amount,                     # length of time
    owns amount_unit,                   # time unit
    relates process;   
```
#### Duration
The only relation in the core Matter Ontology is the one setting basic context for time intervals. In its basic form, `duration` only relates an instance of a process and together with `occurs_at` as potential starting point allows for interval specification. Obviously, in timetable scenarios one would rather introduce a `start_date` and `end_date` - that is in no conflict with Matter Ontology.
##
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
This rule assures transitivity among locations, e.g., if a town has `a_location` a region which in turn has `a_location` a country, then the town naturally finds itself in the country as well. If a user needs to distinguish between these locations and only filter out, say, the country, one can simply introduce a `location_type sub a_category` and distinguish locations by defining them to own this new attribute.
##
### Comparison with OWL
Let's take a quick look at how this translation reflects the original class taxonomy and consider what we left behind:

![owl_to_typedb](https://user-images.githubusercontent.com/56684558/147237301-c8c9e38a-71fc-4f38-ab89-726a4a416155.png)
#### Role
Thanks to the concept of roles being integral to TypeDB, there is no need for introducing them explicitly as entities. In TypeDB, anything that participates in an instance of a relation has to play a specific role within it. We can also drop the concept of 'Basic Thing' since its only meaning is "a non-role".
#### Abstract Thing
Thanks to the instantiation of all things (attributes, entities & relations) in TypeDB, there is little reason to introduce Form or Quality explicitly as entities because those can be both represented by attributes in a clearer way. Since these attributes are instances of the database, they can be directly referred to in rules and queries and thus flexibly used for inference. Also, 'Form' translates accurately to `a_category` (as a handy alternative to large branching of schema taxonomy) and 'Quality' belongs among all other kinds of attributes.
