# Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

## Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid.
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on
your command line run:

```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:

```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:

```bash
npm run format-examples
```

## Adding a new provider

1. Add documentation in a Markdown file to the folder `platforms`
2. Add the provider to the table in the `README.md`, see chapter "type"
3. Add a JSON Schema to the folder `json-schema/platforms`
4. Add the schema to the extension schema in file `json-schema/schema.json` (search for `allOf` below the definition of `storage:schemes`)
5. Add the newly created schema to the `validator-config.json`

Use the same file names (excluding the extension) for documentation and schema.
