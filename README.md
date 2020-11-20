<h3 align="center">fauda</h3>
<h4 align="center">Don't worry about configuration anymore.</h4>

<p align="center">
  <a href="https://zapier.slack.com/archives/CD4LXLTK5">
    <img src="https://img.shields.io/badge/team-foundations-brightgreen?style=flat-square" />
  </a>
  <a href="https://zapier.slack.com/archives/CD4LXLTK5">
    <img src="https://img.shields.io/badge/maintained-yes-brightgreen?style=flat-square" />
  </a>
</p>

<p align="center">
  <a href="#features">Features</a> •
  <a href="#getting-started">Getting Started</a> •
  <a href="#how-does-it-work">How does it work?</a> •
  <a href="#usage">Usage</a> •
  <a href="#api">API</a> •
  <a href="#cli">CLI</a> •
  <a href="#faq">FAQ</a>
</p>

---

Fauda **loads** and **normalizes** your configuration from multiple sources at the same time, giving flexibility to the users of your library / application to configure it in various ways.

## Features

- **One dependency** to load and validate your configuration.
- **Load from multiple sources** such as JSON, YAML, JavaScript, Typescript, command-line arguments, and environment variables.
- **Validate your configuration** with a JSON schema. BONUS: Your configuration files have auto-completion in VSCode!
- **Generate types** for your Typescript code and for Typescript configuration files.
- **Expand environment variables** in any configuration value.

## Getting Started

<details>
<summary><b>1️⃣ Install</b> Fauda</summary><br>

```sh
npm install fauda
```

</details>

<details>
<summary><b>2️⃣ Set up</b> your JSON schema</summary><br>

Create a `schema.json` file:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema",
  "title": "My awesome app configuration",
  "type": "object",
  "properties": {
    "$schema": {
      "description": "Path to my app's schema.",
      "type": "string"
    },
    "port": {
      "description": "The port the server listens to.",
      "type": "number",
      "default": 3000
    },
    "open": {
      "description": "Open in a browser tab if true.",
      "type": "boolean",
      "default": false
    },
    "mode": {
      "description": "Mode of the app.",
      "type": "string",
      "enum": ["development", "production"],
      "default": "${NODE_ENV}"
    },
    "publicPages": {
      "description": "A list of public pages.",
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": ["publicPages"]
}
```

Fauda relies on a [JSON schema](https://json-schema.org/) to load and validate your configuration, but also to generate types.

For more information on how to creat this schema, please take a look at the [Getting Started](https://json-schema.org/learn/getting-started-step-by-step.html) guide.

</details>

<details>
<summary><b>3️⃣ Generate</b> Typescript typings <i>optional</i></summary><br>

Generate a `src/configuration.ts` file:

```sh
$ npx fauda types
```

For more information, please take a look at [CLI](#cli).

```ts
export interface Configuration {
  port?: number
  open?: boolean
  mode?: 'development' | 'production'
  publicPages: string[]
}
```

</details>

<details>
<summary><b>4️⃣ Load & validate</b> your configuration.</summary><br>

Assuming your package is named `my-app`:

```ts
import { fauda } from 'fauda'
import { Configuration } from './configuration'

async function loadConfiguration() {
  try {
    const configuration = await fauda<Configuration>('my-app')
  } catch (err) {
    console.error(err.message)
  }
}
```

</details>

## How does it work?

Fauda loads your configuration from several sources using the following order of precedence: `environment variables > command-line arguments > configuration files`.

Option names are inflected according the source's typical naming convention:

| Source                | Casing        | Example               |
| --------------------- | ------------- | --------------------- |
| Configuration file    | `camel`       | `publicPages`         |
| Command-line argument | `kebab`       | `--public-pages`      |
| Environment variable  | `upper+snake` | `MY_APP_PUBLIC_PAGES` |

Once your configuration loaded, it is normalized into a valid configuration object that your library / application can use safely. The normalization process validates your configuration using the provided JSON schema. It checks that the type of options are valid, required options are specified, sets default values, and also expand environment variables that are referenced!

<details>
<summary>🙋🏻‍♂️ <i>What is environment variable expansion?</i></summary><br>

You can reference an environment variable's name as your option's value. Fauda will replace it by the variable's value when loading the configuration.

Here's an example of an option referencing a environment variable:

```json
{
  "mode": "${NODE_ENV}"
}
```

</details>

## Usage

### Configuration files

Fauda first tries to find a `config.${myApp}` property in the `package.json` of your users.

It then searches for a configuration file starting from the current directory up to the root.

Several configuration file names and formats are supported:

| File                               | Format |
| ---------------------------------- | ------ |
| `.${myPackage}rc`                  | `json` |
| `${myPackage}.config.js`           | `js`   |
| `${myPackage}.config.json`         | `json` |
| `${myPackage}.config.ts`           | `ts`   |
| `${myPackage}.config.yaml`         | `yaml` |
| `${myPackage}.config.yml`          | `yaml` |
| `.config/${myPackage}rc`           | `json` |
| `.config/${myPackage}.config.js`   | `js`   |
| `.config/${myPackage}.config.json` | `json` |
| `.config/${myPackage}.config.ts`   | `ts`   |
| `.config/${myPackage}.config.yaml` | `yaml` |
| `.config/\${myPackage}.config.yml` | `yaml` |

### Command-line arguments

Fauda parses command-line arguments as you can expect from any other argument parsers!

Options are "kebab-"cased. For instance, the `publicPages` option is transposed as the `--public-pages` command-line argument.

<details>
<summary>🙋🏻‍♂️ <i>What about arrays?</i></summary><br>

Arrays are supported in two ways.

1. Declare a JSON-compatible array as value.
2. Use the same argument multiple times.

Here's an example that gives the same result:

```sh
$ my-app --public-pages=/home --public-pages=/about
$ my-app --types="['/home', '/about']"
```

</details>

### Environment variables

Fauda parses environment variables prefixed with your package's name. This is precaution to avoid name clashes with other application or system-wide environment variables.

For instance, if your package's name is `my-app`, Fauda will parse variables with the `MY_APP_` prefix.

<details>
<summary>🙋🏻‍♂️ <i>What about arrays?</i></summary><br>

Arrays are supported! You simply need to declare a JSON-compatible array wrapped between quotes.

Here's an example:

```sh
$ MY_APP_PUBLIC_PAGES="['/home', '/about']"
```

</details>

## API

### [fauda](src/loader.ts#45)

Load, normalize, and return a configuration object from multiple sources.

**Params**

- **namespace** `string`: String used to prefix environment variables and namespace your configuration in the `package.json`. It's typically your package's name.
- **schema** `string | JsonObject`: Path to a JSON schema, or the schema itself, used to normalize your configuration.
- **options** `Partial<FaudaOptions>`: See availabble [options](#options).

### [FaudaOptions](src/types.ts#1)

| Option | Type                | Default         | Description                                                                    |
| ------ | ------------------- | --------------- | ------------------------------------------------------------------------------ |
| `args` | `string[]`          | `process.argv`  | Array of command-line arguments, used by the command-line arguments loader.    |
| `env`  | `NodeJS.ProcessEnv` | `process.env`   | Dictionary of environment variables, used by the environment variables loader. |
| `cwd`  | `string`            | `process.cwd()` | Array of command-line arguments, used by the configuration files loader.       |

### [generateTypes](src/generator.ts#36)

Return the `Configuration` type definition inferred from the passed JSON schema.

**Params**

- **schema** `string | JsonObject`: Path to a JSON schema, or the schema itself, used to generate the `Configuration ` type.
- **options** `Partial<Options>`: See available [options of `json-schema-to-typescript`](https://github.com/bcherny/json-schema-to-typescript#options).

## FAQ

### Why not supporting nested options?

Fauda only supports a flat options object, simply because it's easier to manipulate and to reason about. In my opinion, nested options are usually neither a good idea as it makes merging default options harder, nor necessary as one can express a sense of hierarchy using "dotted names" instead (eg. `cooking.time`) or just camel case (eg. `cookingTime`).
