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



[ipld]: https://github.com/ipld/specs/tree/master/ipld
[schema-generation]: {{ site.baseurl }}{% link _docs/publishing/schema-generation.md %}
[self-desc-schema]: https://github.com/snowplow/iglu/wiki/Self-describing-JSON-Schemas
[jsonld]: http://json-ld.org/
[schemaver]: https://github.com/snowplow/iglu/wiki/SchemaVer
[snowplow]: http://snowplowanalytics.com/
