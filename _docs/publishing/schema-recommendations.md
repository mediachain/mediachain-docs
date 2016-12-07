---
title: Schema Recommendations
layout: page
category: publishing
order: 10
---

* TOC
{:toc #table_of_contents}

Mediachain supports publishing JSON data objects, which are converted to [CBOR](http://cbor.io)
for efficiency and compatibility with [IPLD][ipld].  While it's possible to publish any valid JSON
object, we highly recommend defining a schema for your data to catch any invalid data and make your
records understandable to other developers.

There are two kinds of schema supported by mediachain, [JSON Schema][jsonschema], and the
Linked Data schemas provided by [schema.org](http://schema.org), which you can use by structuring
your data as [JSON-LD][jsonld], a "superset" of JSON that adds semantic links between objects.  

Which to use depends on what kind of data you have and where it's from.  For existing collections of
JSON data, we recommend [using JSON Schema](#using-json-schema), which requires no transformations
to your input data, and gives consumers a useful set of expectations for the "shape" of your
records.

When writing "mediachain first" data that doesn't come from an existing dataset, you can take
advantage of the flexibility of JSON-LD and the broad vocabulary provided by
[schema.org](http://schema.org).

## Using JSON Schema

When publishing objects from an existing,
well-defined dataset, we recommend [generating a JSON Schema][schema-generation] from your data.

Mediachain uses ["self-describing"][self-desc-schema] JSON Schemas, which add a small bit of
metadata to the schema object to so that the schema can identify itself, without needing any
"out of band" context.  

Self-describing schemas use a semantic versioning convention called [SchemaVer][schemaver] to
track the evolution of schemas over time.  SchemaVer is based on the widely used
["semver"](http://semver.org) convention used for software packages, with a few adaptations.  The
most obvious difference is that SchemaVer uses a `-` character as its separator instead of a `.`,
so a SchemaVer looks like this: `1-0-0`.  Where semantic versioning for software divides changes
into `MAJOR.MINOR.PATCH`, SchemaVer uses `MODEL-REVISION-ADDITION`.  Your schema should start at
version `1-0-0` (SchemaVer does not have a notion of "pre-release" schemas), and be sure to read
the [SchemaVer docs][schemaver] as your schema evolves.

When you publish a record with a schema, a reference to the schema object is embedded in the record,
so your data will always link to the version of the schema used when the record was published.

### Creating the schema
The simplest way to create a schema from an existing dataset is to follow our
[schema generation guide][schema-generation], which walks you through using the
[schema-guru][schema-guru] tool to generate a schema from a collection of input records.

If you don't want to use `schema-guru`, you can always define your own JSON Schema.  To make it
compatible with mediachain, you just need to add a small `self` object within the schema definition.
Here's a simple example schema that includes the `self` object:

```json
{
  "self": {
    "vendor": "org.json-schema",
    "name": "basic-example",
    "format": "jsonschema",
    "version": "1-0-0"
  },
  "title": "Example Schema",
  "type": "object",
  "properties": {
    "firstName": {
      "type": "string"
    },
    "lastName": {
      "type": "string"
    },
    "age": {
      "description": "Age in years",
      "type": "integer",
      "minimum": 0
    }
  },
  "required": ["firstName", "lastName"]
}
```

The `self` object contains four required fields, `vendor`, `name`, `format`, and `version`.  

Since this example was adapted from the [JSON Schema website](http://json-schema.org/examples.html),
the `vendor` is set to `"org.json-schema"`.  It's recommended that you use a "reverse DNS" style
naming convention for the `vendor` field, ideally for a domain that you control, since you want the
vendor to be unique across the mediachain network.  

The `name` field identifies the schema, and must be unique per-vendor.  

The only supported value for `format` is currently the string `"jsonschema"`.

The `version` field must be a string in [SchemaVer][schemaver] format.

### Publishing With a JSON Schema

Once you have your schema, you can publish it to your mediachain node with the
`mcclient publishSchema` command:

```
$ mcclient publishSchema org.json-schema-basic-example.json
Published schema with wki = schema:org.json-schema/basic-example/jsonschema/1-0-0 to namespace mediachain.schemas
Object ID: QmcfAkjHsJbbqcbEhetrvenewxCLTm9c1PrhAjGeZBxAst
Statement ID: 4XTTM6XcpMB8N8Tnkhc66DqvAotgKzRJgrZbF7SBByihtPnV8:1481062812:0
```

Make a note of the 'Object ID' in the output; this is what we'll use to refer to the schema when
we publish records.

Now that the schema is published to the node, we can publish statement that fit the schema.

```
$ cat example-records.json
{"firstName": "Harry", "lastName": "Potter"}
{"firstName": "Lisa", "lastName": "Simpson", "age": 10}

$ mcclient publish --idFilter '"example:" + .firstName + ":" + .lastName' --namespace scratch.example --schemaReference QmcfAkjHsJbbqcbEhetrvenewxCLTm9c1PrhAjGeZBxAst example-records.json
statement id: 4XTTM6XcpMB8N8Tnkhc66DqvAotgKzRJgrZbF7SBByihtPnV8:1481066109:1
[ { object: 'QmaQbDazmD1wG3nYHxsRBR3mJbeDsLaQeuqSg9DPg96J5b',
    refs: [ 'example:Harry:Potter' ],
    tags: [],
    deps: [ 'QmcfAkjHsJbbqcbEhetrvenewxCLTm9c1PrhAjGeZBxAst' ] } ]

statement id: 4XTTM6XcpMB8N8Tnkhc66DqvAotgKzRJgrZbF7SBByihtPnV8:1481066109:2
[ { object: 'QmRTUmijH7WH3nC4dmbaprrAYf6Xrp6RoBsJ92wpoNXeZ1',
    refs: [ 'example:Lisa:Simpson' ],
    tags: [],
    deps: [ 'QmcfAkjHsJbbqcbEhetrvenewxCLTm9c1PrhAjGeZBxAst' ] } ]
All statements published successfully
```

If we try to publish a record that doesn't fit the schema (for example, omitting the required
`lastName` field), we'll get a validation error:

```
$ cat nope.json
{"firstName": "Yoda"}

$ mcclient publish --idFilter '"example:" + .firstName + ":" + .lastName' --namespace scratch.example --schemaReference QmcfAkjHsJbbqcbEhetrvenewxCLTm9c1PrhAjGeZBxAst nope.json
Record failed validation: Schema validation failed:
root_object should have required property 'lastName': {"missingProperty":"lastName"}. Failed object: {
  "firstName": "Yoda"
}
```

## Using JSON-LD

[JSON-LD][jsonld] is a linked data format that uses JSON to represent objects, their properties,
and links between related objects.  We recommend using JSON-LD to represent "green field" or
"mediachain native" data because it offers great flexibility, and there are a lot of existing
type definitions that your data can extend.  The [schema.org project](http://schema.org) has a
great collection of types that you can either use directly, or extend for your own purposes.
By using well-known types, your data is much more easily understood and indexed by other
mediachain apps and users.

### Data Model
"Linked Data" is a term that emerged out of the [semantic web][semantic-web-wiki] movement, and it
has taken many forms over the years.  JSON-LD uses a data model that's compatible with
[RDF][rdf], one of the most widely used linked data formats.  

The RDF and JSON-LD data model represents data as a graph of connected objects, where each object is
identified with a unique ID.  In most semantic web contexts, the unique id is a URI, and it's often
a URL that can be resolved to some kind of definition.  For example, the URL
[http://schema.org/Person](http://schema.org/Person) uniquely identifies the `Person` type, and if
you view the HTML source returned by that URL, you can see some special markup that embeds property
definitions into the page.  See the section below on [identifiers](#identifiers) for guidance on
defining ids for mediachain objects.

An RDF object is actually a graph connecting a set of properties to the object's identifier, but
JSON-LD lets us represent this very open-ended graph using familiar JSON conventions.

This `Person` example from the [JSON-LD Playground][jsonld-playground] shows how to define a
JSON-LD object:

```json
{
  "@context": "http://schema.org/",
  "@type": "Person",
  "name": "Jane Doe",
  "jobTitle": "Professor",
  "telephone": "(425) 123-4567",
  "url": "http://www.janedoe.com"
}
```

There are a few key things to notice in that example.

First, the `@context` tells the JSON-LD processor where to find definitions for object and property
types.  Using `"@context": "http://schema.org"` gives meaning to the fields below; without the
`@context`, the fields `name`, `telephone`, etc have no intrinsic meaning.  With the context,
they have the meaning defined by schema.org.

The `@type` field defines the type of object, in this case a [Person](http://schema.org/Person).
While it's technically possible to define any properties you want on any type of object, using the
properties that "belong" to the type of object that you're representing will give you the best
interoperability with other systems, and make your data easier to index.

The schema.org types are arranged in an inheritance hierarchy, and properties from base types are
availble to sub-types.  [Person](http://schema.org/Person) is derived from [Thing](http://schema.org/Thing),
which is where the [`name`](http://schema.org/name) property is inherited from.

You may have noticed that type names (`Person`, `Thing`, etc) begin with a capital letter, while
property names begin with lowercase letters.  This is a convention used throughout the schema.org
definitions, and we recommend using it when defining your own types as well.

### Identifiers

Mediachain stores data in a content addressed object store, which presents a bit of a
"chicken and egg" problem for unique IDs.  It's tempting to try to identify objects solely by the
hash of their contents, which is in fact how mediachain references objects in its datastore.

However, mediachain also uses a database of [statements][glossary-statement] to index and
contextualize the data store, and statements can use "well-known identifiers" (or WKIs) to identify
what a given object is "about".  This gives us a way to model objects from existing data sources,
e.g. we can point to an object from the MoMA collection with the WKI `moma:artworks:11` if we
want to refer to [a specific MoMA artwork](https://www.moma.org/collection/works/11).

In the case of objects without an existing external ID, we can generate our own to allow us to
track the object in the statement db, and give us the ability to model updates to the object
over time.

We intend to add unique "shared nothing" id generation to an upcoming release of `mcclient`, but
our current recommendation is to simply use UUIDs for each object you create.  An object's id
can be embedded in the object definition with the `@id` field:

```json
{
  "@context": "http://schema.org",
  "@type": "Person",
  "@id": "uuid:56D1A4AB-8574-48EC-B419-1AACB58A7A82",
  "name": "Roald Dahl"
}
```

Using the `@id` field, we can also link between related objects.

```json
{
  "@context": "http://schema.org",
  "@type": "Book",
  "@id": "uuid:558BE88C-7933-4B9B-8B31-FD6B3E50706F",
  "title": "Charlie and the Chocolate Factory",
  "author": "uuid:56D1A4AB-8574-48EC-B419-1AACB58A7A82"
}
```

### Publishing JSON-LD Objects

Validation of JSON-LD data is an ongoing effort; we're planning to provide semantic validation of
object and property types in an upcoming release.  At the moment, you can validate *structurally*
using the `mcclient validate --jsonld` command, which will ensure that your data is valid JSON-LD,
but not necessarily that your types are correct.

Publishing JSON-LD data is just like publishing any other JSON data:

```
$ mcclient publish --idFilter '.["@id"]' --namespace scratch.mydata mydata.json
```

As we add more tools for validating and publishing JSON-LD, the publication flow may change somewhat!
Please check back here for the latest usage instructions and recommendations.

[ipld]: https://github.com/ipld/specs/tree/master/ipld
[schema-generation]: {{ site.baseurl }}{% link _docs/publishing/schema-generation.md %}
[self-desc-schema]: https://github.com/snowplow/iglu/wiki/Self-describing-JSON-Schemas
[schemaver]: https://github.com/snowplow/iglu/wiki/SchemaVer
[snowplow]: http://snowplowanalytics.com/
[jsonld]: http://json-ld.org/
[jsonld-playground]: http://json-ld.org/playground/
[rdf]: https://en.wikipedia.org/wiki/Resource_Description_Framework
[semantic-web-wiki]: https://en.wikipedia.org/wiki/Semantic_Web
[glossary-statement]: {{ site.baseurl }}{% link _docs/arch/glossary.md %}#statement
