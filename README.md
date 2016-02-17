# open-stream

`open-stream` is a number of different things all rolled into one:

- An open standard for defining composable transformations that can be performed on streams of data, as well as for how those modules can be chained together to create powerful transformations.
- A reference implementation of a `runner` that can process those steps in order.
- A way of storing these definitions in GitHub so they can be easily versioned, shared, and audited

## Inspirations

- [RecordStream](https://github.com/benbernard/RecordStream) for inspiring the record oriented, streaming architecture
- [Bundler for Ruby](http://bundler.io/) for how it manages dependency versions
- [Travis CI](https://travis-ci.org/) as a model for sandbox environments

## Architecture

`open-stream` is made up of different components, each defined in its own GitHub repo. These components communicate in very simple ways using `datastreams`.

### Data Streams

In `open-stream`, inspired by [RecordStream](https://github.com/benbernard/RecordStream), data streams are simply the *nix standard input or output streams from a collection of scripts.

A stream starts when standard out is opened, and completes when the `end of file` is seen. Each record is a single line, containing a JSON object. Example:

```javascript
{ "type" : "BURGLARY", "timestamp" : "2016-02-17T12:51:49", ... }
{ "type" : "GRAND THEFT AUTO", "timestamp" : "2016-02-17T01:12:31", ... }
<EOF>
```

#### Datatypes

Data Streams are not explicitly typed, but transforms can require particular datatypes, and those types can be enforced lazily as the transformers process the stream.

Implementers are free to introduce their own rich datatypes, but the following built in types are recomended:

- `text`: simple UTF-8 text strings
- `number`: simple floating point numbers. Transforms should use at least 64-bit floating points to represent numbers
- `timestamp`: date and time represented in a [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format. If no time zone is specified, transforms should assume the times are in local time
- `point`: A [GeoJSON](http://geojson.org/)-formated location on the Earth
- `line`: A [GeoJSON](http://geojson.org/)-formated line or multi-line
- `polygon`: A [GeoJSON](http://geojson.org/)-formated polygon or multi-polygon

### Transforms

A transform is, at its simplest implementation, a script that can take data streams as input or output. It lives in its own little GitHub repository, and can be composed with other transforms to form a data pipeline.

Each transform has, defined in its YAML config file:

- A name
- A description
- A set of inputs, which can be fed either static values or field names to pull data from the stream
- A set of outputs they will provide. For things like aggregation transforms, the outputs can replace the output stream entirely, or transforms can modify, add, or remove fields
