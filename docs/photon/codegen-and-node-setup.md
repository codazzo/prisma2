# Code generation & Node.js setup

## TLDR

In order to use Photon.js in your application, you must install the `@prisma/photon` package in your application:

```
npm install @prisma/photon
```

The `@prisma/photon` package itself is a [_facade package_](https://github.com/prisma/photonjs/issues/261) (basically a _stub_) that doesn't contain any functional code, such as types or the Photon.js runtime. When installing the `@prisma/photon` package, its `postinstall` hook is being executed to invoke the `prisma2 generate` command and generate the actual Photon.js code into the facade package at `node_modules/@prisma/photon`.

This means the `prisma2` CLI needs to be available as well. It is typically installed as a development dependency:

```
npm install prisma2 --save-dev
```

## Why is the facade package needed if Photon.js is generated?

The facade package is necessary to enable typical build and deployment workflows of Node.js applications. As an example, the facade package ensures that Photon.js survives the ["pruning"](https://docs.npmjs.com/cli/prune.html) that's often employed by Node.js package managers.

Note that you'll need to re-execute `prisma2 generate` whenever you make changes to your [Prisma schema](../prisma-schema-file.md) (or perform the changes while are you're running Prisma's [development mode](../development-mode.md). 

> **Note**: While this approach has a number of [benefits](#why-is-photon-js-generated-into-node_modulesgenerated-by-default), it is also unconventional and can be a source confusion for developers new to Photon.js. Using `node_modules/@prisma/photon` as the default `output` for Photon.js is still experimental. Please share your feedback and tell us whether you think this is a good idea or any other thoughts you have on this topic by joining the [discussion on GitHub](https://github.com/prisma/photonjs/issues/88).

## Specifying the target location for Photon.js

`prisma2 generate` invokes the [generators](../prisma-schema-file.md#generators-optional) specified in the [Prisma schema file](../prisma-schema-file.md) and generates the respective packages on the respective output path(s). 

The default Photon.js generator can be specified as follows in your schema file:

```groovy
generator photonjs {
  provider = "photonjs"
}
```

Note that this is equivalent to specifying the default `output` path:

```groovy
generator photonjs {
  provider = "photonjs"
  output   = "./node_modules/@prisma/photon"
}
```

When running `prisma2 generate` for either of these schema files, Photon.js package will be located in:

```
node_modules/@prisma/photon
```

You can also specify a custom `output` path on the `generator` configuration, for example:

```groovy
generator photonjs {
  provider = "photonjs"
  output   = "./src/generated/photon"
}
```

## Photon.js should be viewed as an npm package

Node.js libraries are typically installed as npm dependencies using `npm install`. The respective packages are then located inside the [`node_modules`](https://docs.npmjs.com/files/folders#node-modules) directory from where they can be imported into application code.

Because Photon.js is a custom API for _your specific database setup_, it can't follow that model. It needs to be generated locally instead of being installed from a central repository like npm. However, the mental model for Photon.js should still be that of an Node.js module.

## Why is Photon.js generated into `node_modules/@prisma/photon` by default?

### Importing Photon.js

By generating Photon.js into `node_modules/@prisma/photon`, you can import it into your code:

```js
import { Photon } from '@prisma/photon'
```

or

```js
const { Photon } = require('@prisma/photon')
```

### Keeping the query engine binary out of version control by default

Photon.js is based on a _query engine_ that's running as a binary alongside your application. This binary is downloaded when `prisma2 generate` is invoked and stored in the `output` path (right next to the generated Photon API).

By generating Photon.js into `node_modules`, the query engine is kept out of version control by default (since `node_modules` is typically ignored for version control). If it was not generated into `node_modules`, then developers would need to explicitly ignore it, e.g. for Git they'd need to add the `output` path to `.gitignore`.

## Generating Photon.js in the `postinstall` hook of `@prisma/photon`

The `@prisma/photon` package defines its own `postinstall` hook that's being executed whenever the package in being installed. This hook invokes the `prisma2 generate` command which in turn generates the Photon.js code into the default location `node_modules/@prisma/photon`. Notice that this requires the `prisma2` CLI to be available, either as local dependency or as a global installation (it is recommended to always install the `prisma2` package as a development dependency, using `npm install prisma2 --save-dev`, to avoid versioning conflicts though).

