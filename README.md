# PlanetScale Serverless Driver for JavaScript

A Fetch API-compatible PlanetScale database driver for serverless and edge compute platforms that require HTTP external connections, such as Cloudflare Workers or Vercel Edge Functions

## Installation

```
npm install @planetscale/database
```

## Usage

```ts
import { connect } from '@planetscale/database'

const config = {
  host: '<host>',
  username: '<user>',
  password: '<password>'
}

const conn = connect(config)
const results = await conn.execute('select 1 from dual where 1=?', [1])
console.log(results)
```

### Database URL

A single database URL value can be used to configure the `host`, `username`, and `password` values.

```ts
import { connect } from '@planetscale/database'

const config = {
  url: process.env['DATABASE_URL'] || 'mysql://user:pass@host'
}

const conn = connect(config)
```

### Connection factory

Use the `Client` connection factory class to create fresh connections for each transaction or web request handler.

```ts
import { Client } from '@planetscale/database'

const client = new Client({
  host: '<host>',
  username: '<user>',
  password: '<password>'
})

const conn = client.connection()
const results = await conn.execute('select 1 from dual')
console.log(results)
```

### Custom fetch function

Node.js version 18 includes a built-in global `fetch` function. When using an older version of Node.js, you can provide a custom fetch function implementation. We recommend the [`undici`][1] package on which Node's built-in fetch is based.

[1]: https://github.com/nodejs/undici

```ts
import { connect } from '@planetscale/database'
import { fetch } from 'undici'

const config = {
  fetch,
  host: '<host>',
  username: '<user>',
  password: '<password>'
}

const conn = connect(config)
const results = await conn.execute('select 1 from dual')
console.log(results)
```

### Custom query parameter format function

Query replacement parameters identified with `?` are replaced with escaped values. Named replacement parameters are supported with a colon prefix.

```ts
const results1 = await conn.execute('select 1 from dual where 1=?', [42])
const results2 = await conn.execute('select 1 from dual where 1=:id', { id: 42 })
```

Providing a custom format function overrides the built-in escaping with an external library, like [`sqlstring`](https://github.com/mysqljs/sqlstring).

```ts
import { connect } from '@planetscale/database'
import SqlString from 'sqlstring'

const config = {
  format: SqlString.format,
  host: '<host>',
  username: '<user>',
  password: '<password>'
}

const conn = connect(config)
const results = await conn.execute('select 1 from dual where 1=?', [42])
console.log(results)
```

### Custom type casting function

Column values are converted to their corresponding JavaScript data types. This can be customized by providing a `cast` function.

```ts
import { connect, cast } from '@planetscale/database'

function inflate(field, value) {
  if (field.type === 'INT64' || field.type === 'UINT64') {
    return BigInt(value)
  }
  return cast(field, value)
}

const config = {
  cast: inflate,
  host: '<host>',
  username: '<user>',
  password: '<password>'
}

const conn = connect(config)
```

## Development

```
npm install
npm test
```

## License

Distributed under the Apache 2.0 license. See LICENSE for details.
