<p align="center">
  <a href="https://www.linkedin.com/in/hannah-morten-b1218017a/"><img height="200" src="https://github.com/asos-craigmorten/superdeno/raw/main/.github/superdeno.png" alt="Super Deno standing in the rain at night – stoically facing the dark battle that is software engineering"></a>
  <h1 align="center">SuperDeno</h1>
</p>
<p align="center">
HTTP assertions for Deno made easy via <a href="https://github.com/visionmedia/superagent">superagent</a>.
</p>
<p align="center">
   <a href="https://github.com/asos-craigmorten/superdeno/tags/"><img src="https://img.shields.io/github/tag/asos-craigmorten/superdeno" alt="Current version" /></a>
   <img src="https://github.com/asos-craigmorten/superdeno/workflows/Test/badge.svg" alt="Current test status" />
   <a href="https://doc.deno.land/https/deno.land/x/superdeno/mod.ts"><img src="https://doc.deno.land/badge.svg" alt="SuperDeno docs" /></a>
   <a href="http://makeapullrequest.com"><img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs are welcome" /></a>
   <a href="https://github.com/asos-craigmorten/superdeno/issues/"><img src="https://img.shields.io/github/issues/asos-craigmorten/superdeno" alt="SuperDeno issues" /></a>
   <img src="https://img.shields.io/github/stars/asos-craigmorten/superdeno" alt="SuperDeno stars" />
   <img src="https://img.shields.io/github/forks/asos-craigmorten/superdeno" alt="SuperDeno forks" />
   <img src="https://img.shields.io/github/license/asos-craigmorten/superdeno" alt="SuperDeno license" />
   <a href="https://GitHub.com/asos-craigmorten/superdeno/graphs/commit-activity"><img src="https://img.shields.io/badge/Maintained%3F-yes-green.svg" alt="SuperDeno is maintained" /></a>
   <a href="http://hits.dwyl.com/asos-craigmorten/superdeno"><img src="http://hits.dwyl.com/asos-craigmorten/superdeno.svg" alt="SuperDeno repository visit count" /></a>
</p>

---

## Table of Contents

- [Getting Started](#getting-started)
- [About](#about)
- [Installation](#installation)
- [Example](#example)
- [Documentation](#documentation)
- [API](#api)
- [Notes](#notes)
- [Contributing](#contributing)
- [License](#license)

## Getting Started

```ts
import { superdeno } from "https://deno.land/x/superdeno@main/mod.ts";
import { opine } from "https://deno.land/x/opine@0.8.0/mod.ts";

const app = opine();

app.get("/user", (req, res) => {
  res.setStatus(200).json({ name: "deno" });
});

superdeno(app)
  .get("/user")
  .expect("Content-Type", /json/)
  .expect("Content-Length", "15")
  .expect(200)
  .end((err, res) => {
    if (err) throw err;
  });
```

## About

The motivation of this module is to provide a high-level abstraction for testing HTTP in Deno, while still allowing you to drop down to the lower-level API provided by [superagent](https://visionmedia.github.io/superagent/).

## Installation

This is a [Deno](https://deno.land/) module available to import direct from this repo and via the [Deno Registry](https://deno.land/x).

Before importing, [download and install Deno](https://deno.land/#installation).

You can then import SuperDeno straight into your project:

```ts
import { superdeno } from "https://deno.land/x/superdeno@main/mod.ts";
```

If you want to use a specific version of SuperDeno, just modify the import url to contain the version:

```ts
import { superdeno } from "https://deno.land/x/superdeno@0.3.0/mod.ts";
```

Or if you want to use a specific commit of SuperDeno, just modify the import url to contain the commit hash:

```ts
import { superdeno } from "https://deno.land/x/superdeno@c21f8d6/mod.ts";
```

## Example

You may pass a url string, [`http.Server`](https://doc.deno.land/https/deno.land/std/http/mod.ts#Server), a request handling function, or an object that implements an `app.listen()` method (which mirrors the [`http.serve`](https://doc.deno.land/https/deno.land/std/http/mod.ts#serve) interface) to `superdeno()` - if SuperDeno identifies that a server is not already listening for connections, then one is bound to an ephemeral port for you so there is no need to keep track of ports.

SuperDeno works with any Deno test framework. Here's an example with Deno's built-in test framework, note how you can pass `done` straight to any of the `.expect()` calls:

```ts
Deno.test("GET /user responds with json", async () => {
  await superdeno(app)
    .get("/user")
    .set("Accept", "application/json")
    .expect("Content-Type", /json/)
    .expect(200);
});
```

Here's an example of SuperDeno working with the Opine web framework:

```ts
import { opine } from "https://deno.land/x/opine@0.8.0/mod.ts";
import { expect } from "https://deno.land/x/expect@9effa6/mod.ts";

const app = opine();

app.get("/", (req, res) => {
  res.send("hey");
});

Deno.test("it should support regular expressions", async () => {
  await superdeno(app)
    .get("/")
    .expect("Content-Type", /^application/)
    .end((err) => {
      expect(err.message).toEqual(
        'expected "Content-Type" matching /^application/, got "text/html; charset=utf-8"'
      );
    });
});
```

Here's an example of SuperDeno working with the Oak web framework:

```ts
import { Application, Router } from "https://deno.land/x/oak@v5.0.0/mod.ts";

const router = new Router();
router.get("/", (ctx) => {
  ctx.response.body = "hello";
});

const app = new Application();
app.use(router.routes());
app.use(router.allowedMethods());

Deno.test("it should support the Oak framework", () => {
  const controller = new AbortController();
  const { signal } = controller;

  app.addEventListener("listen", async ({ hostname, port, secure }) => {
    const protocol = secure ? "https" : "http";
    const url = `${protocol}://${hostname}:${port}`;

    await superdeno(url)
      .get("/")
      .expect("hello", () => {
        controller.abort();
      });
  });

  await app.listen({ port: 0, signal });
});
```

If you are using the [Oak](https://github.com/oakserver/oak/) web framework then it is recommended that you use the specialised [SuperOak](https://github.com/asos-craigmorten/superoak) assertions library for reduced bootstrapping.

For further examples, see the [tests](./test) or the [supertest examples](https://github.com/visionmedia/supertest#example) for inspiration.

## Documentation

- [SuperDeno Deno Docs](https://doc.deno.land/https/deno.land/x/superdeno/mod.ts)
- ~[SuperDeno Type Docs](https://asos-craigmorten.github.io/superdeno/)~ - Unavailable while awaiting GitHub's support of alternatively named default branches.
- [License](https://github.com/asos-craigmorten/superdeno/blob/main/LICENSE.md)
- [Changelog](https://github.com/asos-craigmorten/superdeno/blob/main/.github/CHANGELOG.md)

## API

You may use any [superagent](http://github.com/visionmedia/superagent) client (browser) methods and perform assertions in the `.end()` callback for lower-level needs.

### .expect(status[, fn])

Assert response `status` code.

### .expect(status, body[, fn])

Assert response `status` code and `body`.

### .expect(body[, fn])

Assert response `body` text with a string, regular expression, or parsed body object.

### .expect(field, value[, fn])

Assert header `field` `value` with a string or regular expression.

### .expect(function(res) {})

Pass a custom assertion function. It'll be given the response object to check. If the check fails, throw an error.

```ts
superdeno(app).get("/").expect(hasPreviousAndNextKeys).end(done);

function hasPreviousAndNextKeys(res) {
  if (!("next" in res.parsedBody)) throw new Error("missing next key");
  if (!("prev" in res.parsedBody)) throw new Error("missing prev key");
}
```

### .end(fn)

Perform the request and invoke `fn(err, res)`.

## Notes

This is a port (not fork) of [supertest](https://github.com/visionmedia/supertest) to TypeScript + Deno, which fulfills this motivation currently for Node. This module also includes a XHR sham so [superagent](https://visionmedia.github.io/superagent/) client mode can be used directly.

## Contributing

[Contributing guide](https://github.com/asos-craigmorten/superdeno/blob/main/.github/CONTRIBUTING.md)

---

## License

This library is a port of [supertest](https://github.com/visionmedia/supertest) whose license and copyrights are available at [SUPERTEST_LICENSE](./SUPERTEST_LICENSE.md) in the root of this repository, and covers all files within the [source](./src) directory which detail that the file is a port.

SuperDeno is licensed under the [MIT License](./LICENSE.md).

Icon designed and created by [Hannah Morten](https://www.linkedin.com/in/hannah-morten-b1218017a/).
