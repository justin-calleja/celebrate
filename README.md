![Celebrate](https://github.com/continuationlabs/celebrate/raw/master/images/logo.png)

<sub>Logo design by chris.ruppert@gmail.com</sub>

[![Current Version](https://img.shields.io/npm/v/celebrate.svg)](https://www.npmjs.org/package/celebrate)
[![Build Status](https://travis-ci.org/continuationlabs/celebrate.svg?branch=master)](https://travis-ci.org/continuationlabs/celebrate)

[![belly-button-style](https://cdn.rawgit.com/continuationlabs/belly-button/master/badge.svg)](https://github.com/continuationlabs/belly-button)

`celebrate` is an Express middleware function that wraps the [joi](https://github.com/hapijs/joi) validation library. This allows you to use this middleware in any single route, or globally, and ensure that all of your inputs are correct before any handler function. The middleware allows you to validate `req.params`, `req.headers`, `req.query` and `req.body` (provided you are using `body-parser`).

`celebrate` uses ["peerDependencies"](https://docs.npmjs.com/files/package.json#peerdependencies) to manage the required version of `joi` it will use. This means that if you're using npm@3, *you must* install a compatible version of `joi` (currently *9.x.x*) as a top level dependency for `celebrate` to work correctly. `celebrate` does *not* install its own copy of `joi` when using npm@3. This is to maximize compatibility and to keep the number of `joi` version mismatch bugs to a minimum.

Wondering why *another* joi middleware library for Express? Full blog post [here](https://blog.continuation.io/time-to-clelebrate/).

## Usage

Example of using `celebrate` on a single POST route to validate `req.body`.
```js
const express = require('express');
const BodyParser = require('body-parser');
const Joi = require('joi');
const Celebrate = require('celebrate');

const app = express();
app.use(BodyParser.json());

app.post('/signup', Celebrate({
  body: Joi.object().keys({
    name: Joi.string().required(),
    age: Joi.number().integer(),
    role: Joi.string().default('admin')
  }),
  query: {
    token: Joi.string().token().required()
  }
}), (req, res) => {
  // At this point, req.body has been validated and 
  // req.body.name is equal to req.body.name if provided in the POST or set to 'admin' by joi
});

// By default, Express will try to send our errors back as HTML, if you want the JSON, add an error handler here
app.use((err, req, res) => {
  if (err.isJoi) {
    return res.status(400).send(err);
  }
  res.status(500).send('Some other error');
});
``` 

Example of using `celebrate` to validate all incoming requests to ensure the `token` header is present and mathes the supplied regular expression.
```js
const express = require('express');
const Joi = require('joi');
const Celebrate = require('celebrate');
const app = express();

// valide all incoming request headers for the token header
// if missing or not the correct format, respond with an error
app.use(Celebrate({
 headers: Joi.object({
   token: Joi.string().required().regex(/abc\d{3}/)
 }).unknown()
}));
app.get('/', (req, res) => { res.send('hello world'); });
app.get('/foo', (req, res) => { res.send('a foo request'); });
app.use((err, req, res) => {
  if (err.isJoi) {
    return res.status(400).send(err);
  }
  res.status(500).send('Some other error');
});
```

## API

### `celebrate(schema, [options])`

Returns a `function` with the middleware signature (`(req, res, next)`).

- `schema` - a object where `key` can be one of `'params', 'headers', 'query', and 'body'` and the `value` is a [joi](https://github.com/hapijs/joi/blob/master/API.md) validation schema. Only the `key`s specified will be validated against the incomming `req` object. If you omit a key, that part of the `req` object will not be validated. A schema must contain at least one of the valid keys. 
- `[options]` - `joi` [options](https://github.com/hapijs/joi/blob/master/API.md#validatevalue-schema-options-callback) that are passed directly into the `validate` function.

## Order

`celebrate` validates `req` values in the following order:

1. `req.headers`
2. `req.params`
3. `req.query`
4. `req.body`

If any of the configured validation rules fail, the entire request will be considered invalid and the rest of the validation will be short-circuited and the validation error will be passed into `next`. 

## Issues

*Before* opening issues on this repo, make sure your joi schema is correct and working as you intended. The bulk of this code is just exposing the joi API as Express middleware. All of the heavy lifting still happens inside joi. 
