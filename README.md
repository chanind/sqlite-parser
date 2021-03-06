# sqlite-parser

[![NPM Version Image](https://img.shields.io/npm/v/sqlite-parser.svg)](https://www.npmjs.com/package/sqlite-parser)
[![dependencies Status Image](https://img.shields.io/david/codeschool/sqlite-parser.svg)](https://github.com/codeschool/sqlite-parser/)
[![devDependencies Status Image](https://img.shields.io/david/dev/codeschool/sqlite-parser.svg)](https://github.com/codeschool/sqlite-parser/)
[![License Type Image](https://img.shields.io/github/license/codeschool/sqlite-parser.svg)](https://github.com/codeschool/sqlite-parser/blob/master/LICENSE)

This library parses SQLite queries, using JavaScript, and generates
_abstract syntax tree_ (AST) representations of the input strings. A
syntax error is produced if an AST cannot be generated.

**This parser is written against the [SQLite 3 spec](https://www.sqlite.org/lang.html).**

## Install

```
npm install sqlite-parser
```

## Demo

There is an interactive demo of the parser hosted
[at this location](http://codeschool.github.io/sqlite-parser/demo/). You 
can run a copy of the demo on your local machine by cloning this repository 
and then using the command `grunt live`.

## Usage

The library exposes a function that accepts two arguments: a string
containing SQL to parse and a callback function.

If invoked without a callback function the parser will runs synchronously and
return the resulting AST or throw an error if one occurs.

``` javascript
var sqliteParser = require('sqlite-parser');
var query = 'select pants from laundry;';
// sync
var ast = sqliteParser(query);
console.log(ast);

// async
sqliteParser(query, function (err, ast) {
  if (err) {
    console.log(err);
    return;
  }
  console.log(ast);
});
```

## Syntax Errors

This parser will try to create _smart_ error messages when it cannot parse
some input SQL. In addition to an approximate location for the syntax error,
the parser will attempt to describe the area of concern
(e.g.: `Syntax error found near Column Identifier (WHERE Clause)`).

## AST

**NOTE: The SQLite AST is a work-in-progress and subject to change.**

### Example

You can provide one or more SQL statements at a time. The resulting AST object
has, at the highest level, a `statement` key that consists of an array containing
the parsed statements.

#### Input SQL

``` sql
SELECT
 MIN(honey) AS "Min Honey",
 MAX(honey) AS "Max Honey"
FROM
 BeeHive
```

#### Result AST

``` json
{
  "statement": [
    {
      "type": "statement",
      "variant": "select",
      "result": [
        {
          "type": "function",
          "name": "min",
          "args": [
            {
              "type": "identifier",
              "variant": "column",
              "name": "honey"
            }
          ],
          "alias": "Min Honey"
        },
        {
          "type": "function",
          "name": "max",
          "args": [
            {
              "type": "identifier",
              "variant": "column",
              "name": "honey"
            }
          ],
          "alias": "Max Honey"
        }
      ],
      "from": [
        {
          "type": "identifier",
          "variant": "table",
          "name": "beehive"
        }
      ]
    }
  ]
}
```

## Contributing

Once the dependencies are installed, start development with the following command:

```
grunt test-watch
```

which will automatically compile the parser and run the tests in
`test/core/**/*-spec.js` each time a change is made to the tests and/or
the source code.

Optionally, run `grunt debug` to get AST output from each test in addition to
live reloading.

Finally, you should run `grunt release` before creating any PR. **Do not** change
the version number in `package.json` inside of the pull request.

### Writing tests

Each test refers to a SQL input file in `test/sql/` and an expected output
JSON AST file.

For example a `describe()` block with the title `parent block` that contains an
`it()` block named `super test 2` will look for the SQL input at
`test/sql/parent-block/super-test-2.sql` and the JSON AST at
`test/json/parent-block/super-test-2.json`.

There are three options for the test helpers exposed by `tree`:
- Assert that the test file successfully generates _any_ valid AST
  ``` javascript
  tree.ok(this, done);
  ```

- Assert that the test file generates an AST that exactly matches the expected output JSON file
  ``` javascript
  tree.equals(this, done);
  ```

- `tree.error()` to assert that a test throws an error
  - Assert a specific error `message` for the thrown error
    ``` javascript
    tree.error('My error message.', this, done);
    ```
  - Assert an object of properties that all exist in the thrown error object
    ``` javascript
    tree.error({
      message: 'You forgot to add a boop to the beep.',
      location: {
        start: { offset: 0, line: 1, column: 1 },
        end: { offset: 0, line: 1, column: 1 }
      }
    }, this, done)
    ```

``` javascript
/* All the helper functions in `/test/helpers.js` are already available
 * and do not need to be explicitly imported.
 */
describe('select', function () {
  /* Test SQL: test/sql/select/basic-select.sql
   * Expected JSON AST: test/json/select/basic-select.json
   */
  it('basic select', function (done) {
    tree.equals(this, done);
  });
});

describe('parse errors', function (done) {
  /* Test SQL: test/sql/parse-errors/parse-error-1.sql
   * Expected JSON AST: test/json/parse-errors/parse-error-1.json
   */
  it('parse error 1', function(done) {
    tree.error({
      'message': 'Syntax error found near Column Identifier (WHERE Clause)'
    }, this, done);
  });
});
```
