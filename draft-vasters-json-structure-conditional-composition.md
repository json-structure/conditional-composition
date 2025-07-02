---

title: "JSON Structure: Conditional Composition"
category: std

docname: draft-vasters-json-structure-conditional-composition-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date: 2025-07-02
consensus: true
v: 3
area: Web and Internet Transport
workgroup: Building Blocks for HTTP APIs
keyword:
 - JSON
 - schema
venue:
  group: TBD
  type: Working Group
  mail: TBD
  arch: TBD
  github: "json-structure/conditional-composition"
  latest: "https://json-structure.github.io/conditional-composition/draft-vasters-json-structure-conditional-composition.html"

author:
 -
    fullname: Clemens Vasters
    organization: Microsoft Corporation
    email: clemensv@microsoft.com

normative:
  RFC2119:
  RFC4646:
  RFC8174:
  JSTRUCT-CORE:
    title: "JSON Structure Core"
    author:
      - fullname: Clemens Vasters
    target: https://json-structure.github.io/core/draft-vasters-json-structure-core.html

informative:


--- abstract


This document specifies JSON Structure Conditional Composition, an extension to
JSON Structure Core that introduces composition constructs for combining multiple
schema definitions. In particular, this specification defines the semantics,
syntax, and constraints for the keywords `allOf`, `anyOf`, `oneOf`, and `not`,
as well as the `if`/`then`/`else` conditional construct.


--- middle

# Introduction {#introduction}

This document specifies JSON Structure Conditionals, an extension to JSON Structure Core {{JSTRUCT-CORE}} that introduces conditional composition constructs for combining multiple schema definitions. In particular, this specification defines the semantics, syntax, and constraints for the keywords `allOf`, `anyOf`, `oneOf`, and `not`, as well as the `if`/`then`/`else` conditional construct.

# Terminology and Conventions {#terminology-and-conventions}

The key words MUST, MUST NOT, SHALL, SHALL NOT, REQUIRED, SHOULD, and OPTIONAL are to be interpreted as described in {{RFC2119}} and {{RFC8174}}.

Unless otherwise specified, all references to "non-schema" refer to a JSON Structure Core non-schema object, which is an inert JSON object that does not declare a type constraint.

# Composition and Evaluation Model {#composition-and-evaluation-model}

The keywords introduced in this document extend the set of keywords allowed for non-schemas and schemas as defined in JSON Structure Core. In particular, the keywords defined herein MAY extend all non-schema and schema definitions.

The focus of JSON Structure Core is on data definitions. The conditional composition keywords introduced in this document allow authors to define conditional matching rules that use these fundamental data definitions.

A schema document using these keywords is not a data definition but a rule set for evaluating JSON node instances against schema definitions and lays the groundwork for validation.

Fundamentally, evaluating a JSON node against a schema involves matching the node against the schema's constraints.

The outcome of evaluating a JSON node against a schema is ultimately a boolean value that states whether the node met all constraints defined in the schema. The evaluation also creates an understanding of which constraint was met for each subschema during evaluation.

A schema evaluation engine traverses the given JSON node and the schema definition, evaluating the node and the schema recursively. When a conditional composition keyword is encountered, the engine evaluates each subschema independently against the current node and then combines the results as specified by the composition keyword.

# Conditional Composition Keywords {#conditional-composition-keywords}

This section defines several composition keywords that combine schema definitions with evaluation rules. Each keyword has a specific evaluation semantics that determines the outcome of the validation process.

## `allOf` {#allOf}

The value of the `allOf` keyword MUST be a type-union array containing at least one schema object. A JSON node is valid against `allOf` if and only if it is valid against every schema in the array.

Consider the following non-schema, which does not define its own type but rather contains an `allOf` keyword with three subschemas:

~~~json
{
  "allOf": [
    {
      "type": "object",
      "properties": {
        "a": { "type": "string" }
      },
      "required": ["a"],
      "additionalProperties": true
    },
    {
      "type": "object",
      "properties": {
        "b": { "type": "number" }
      },
      "required": ["b"],
      "additionalProperties": true
    },
    {
      "type": "object",
      "properties": {
        "c": { "type": "boolean" }
      },
      "required": ["c"],
      "additionalProperties": true
    }
  ]
}
~~~

Here, a JSON node evaluates to `true` if it is an object with at least three properties `a`, `b`, and `c`, where `a` is a string, `b` is a number, and `c` is a boolean:

~~~json
{
  "a": "string",
  "b": 42,
  "c": true
}
~~~

The JSON node satisfies all constraints defined by all subschemas. Conflicting constraints among subschemas result in an unsatisfiable schema—for example, if two subschemas require the same property to have different types or if one of the subschemas has `additionalProperties` set to `false`.

## `anyOf` {#anyOf}

The value of the `anyOf` keyword MUST be a type-union array containing at least one schema object. A JSON node is valid against `anyOf` if and only if it is valid against at least one of the schemas in the array.

Consider the following schema:

~~~json
{
  "anyOf": [
    {
      "type": "object",
      "properties": {
        "a": { "type": "string" }
      },
      "required": ["a"],
      "additionalProperties": true
    },
    {
      "type": "object",
      "properties": {
        "b": { "type": "number" }
      },
      "required": ["b"],
      "additionalProperties": true
    },
    {
      "type": "object",
      "properties": {
        "c": { "type": "boolean" }
      },
      "required": ["c"],
      "additionalProperties": true
    }
  ]
}
~~~

Here, a JSON node evaluates to `true` if it is an object with at least one of the properties `a`, `b`, or `c`, where `a` is a string, `b` is a number, and `c` is a boolean:

~~~json
{
  "a": "string"
}
~~~

or

~~~json
{
  "b": 42,
  "c": true
}
~~~

Both JSON nodes satisfy the constraints defined by at least one subschema.

## `oneOf` {#oneOf}

The value of the `oneOf` keyword MUST be a type-union array containing at least one schema object. A JSON node is valid against `oneOf` if and only if it is valid against exactly one of the schemas in the array.

Consider the following schema:

~~~json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "a": { "type": "string" }
      },
      "required": ["a"],
      "additionalProperties": true
    },
    {
      "type": "object",
      "properties": {
        "b": { "type": "number" }
      },
      "required": ["b"],
      "additionalProperties": true
    },
    {
      "type": "object",
      "properties": {
        "c": { "type": "boolean" }
      },
      "required": ["c"],
      "additionalProperties": true
    }
  ]
}
~~~

Here, a JSON node evaluates to `true` if it is an object with exactly one of the properties `a`, `b`, or `c`, where `a` is a string, `b` is a number, and `c` is a boolean:

~~~json
{
  "a": "string"
}
~~~

The following JSON node evaluates to `false` because it matches two subschemas:

~~~json
{
  "a": "string",
  "b": 42
}
~~~

## `not` {#not}

The value of the keyword `not` is a single schema object, which MAY be a type union. A JSON node is valid against `not` if it is not valid against the schema. For example, the schema is written as follows:

~~~json
{
  "not": { "type": "string" }
}
~~~

Here, a JSON node evaluates to `true` if it is not a string:

~~~json
42
~~~

## `if`/`then`/`else` {#if-then-else}

The values of the keywords `if`, `then`, and `else` are schema objects. If the processed JSON node is valid against the `if` schema, the `then` schema further constrains the JSON node and MUST match the input. If the processed JSON node is not valid against the `if` schema, the `else` schema further constrains the JSON node and MUST match the input.

Consider the following schema:

~~~json
{
  "if": {
    "properties": {
      "a": { "type": "string" }
    },
    "required": ["a"]
  },
  "then": {
    "properties": {
      "b": { "type": "number" }
    },
    "required": ["b"]
  },
  "else": {
    "properties": {
      "c": { "type": "boolean" }
    },
    "required": ["c"]
  }
}
~~~

Here, a JSON node evaluates to `true` if it is an object with a property `a` that is a string; then it must also have a property `b` that is a number:

~~~json
{
  "a": "string",
  "b": 42
}
~~~

Otherwise, if the JSON node does not have a property `a` that is a string, it must have a property `c` that is a boolean:

~~~json
{
  "c": true
}
~~~

or

~~~json
{
  "a": 42,
  "c": false
}
~~~

## Enabling the Extensions {#enabling-the-extensions}

The conditional composition extensions can be enabled in a schema or meta-schema
by adding the `JSONSchemaConditionalComposition` key to the `$uses` clause when
referencing the extended meta-schema:

~~~ json
{
  "$schema": "https://json-structure.org/meta/extended/v0/#",
  "$id": "myschema",
  "$uses": [
    "JSONSchemaConditionalComposition"
  ],
  "oneOf" : [
    { "type": "string" },
    { "type": "number" }
  ]
}
~~~

Conditional composition is enabled by default in the validation meta-schema:

~~~ json
{
  "$schema": "https://json-structure.org/meta/validation/v0/#",
  "$id": "myschema",
  "type": "object",
  "oneOf" : [
    { "type": "string" },
    { "type": "number" }
  ]
}
~~~


# Security Considerations {#security-considerations}

- The use of composition keywords does not alter the security model of JSON Structure Core; however, excessive nesting or overly complex compositions may impact performance and resource usage.
- Implementations MUST ensure that all subschema references resolve within the same document or trusted sources to prevent external schema injection.

# IANA Considerations {#iana-considerations}

This document does not require any IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
