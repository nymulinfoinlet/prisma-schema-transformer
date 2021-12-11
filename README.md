# prisma-schema-transformer

**This is a temporary patch**

Patch for support prisma 3

You can look for the [source repository](https://github.com/IBM/prisma-schema-transformer), maybe it has a new release.

Test on Prisma 3.6.0 only

Thank source repository contributors and [Marc-Oros fork](https://github.com/Marc-Oros/prisma-schema-transformer)

## Install

```bash
yarn add --dev https://github.com/allyusd/prisma-schema-transformer.git
```

## Usage

```bash
yarn prisma-schema-transformer prisma/schema.prisma
```

## More about my solution

Write a `prisma/preSchema.prisma` and add it to git

Add `prisma/schema.prisma` to `.gitignore`

Add `package.json` scripts

```json
  "scripts": {
    "schema": "cp prisma/preSchema.prisma prisma/schema.prisma && yarn prisma-schema-transformer prisma/schema.prisma",
    "migrate": "yarn schema && yarn prisma migrate dev --name"
  }
```

transformer schema only

```bash
yarn schema
```

transformer schema and migrate

```bash
yarn migrate init
```

## Example

write `prisma/preSchema.prisma` with `snake_case`

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model user {
  id    Int     @default(autoincrement()) @id
  email String  @unique
  name  String?
  hello_world  String?
  posts post[]
}

model post {
  id        Int      @default(autoincrement()) @id
  title     String
  content   String?
  published Boolean? @default(false)
  author    user?    @relation(fields: [author_id], references: [id])
  author_id  Int?
}
```

# Source content

> **EXPERIMENTAL FOR PRISMA V3**

This project utilizes the [getDMMF](https://github.com/prisma/prisma/blob/023249752380976d797518e1350199895246d099/src/packages/sdk/src/engineCommands.ts#L45) method from `@prisma/sdk` to perform some post-processing work on generated Prisma schema, including the following.

- Transform `snake_case` to `camelCase`, 
- Properly singularize or pluralize model and field name.
- Add `@updatedAt` attribute to field in the event of column name is `updated_at`
- Ignore models from the schema.

__TODO__

- [ ] Auto generate the `generator` and `datasource` nodes.

## Install

```bash
$ yarn global add prisma-schema-transformer
```

## Usage

```bash
$ prisma-schema-transformer prisma/schema.prisma
```

```
Usage
  $ prisma-schema-transformer [options] [...args]

Specify a schema:
  $ prisma-schema-transformer ./schema.prisma

Instead of saving the result to the filesystem, you can also print it
  $ prisma-schema-transformer ./schema.prisma --print

Exclude some models from the output
  $ prisma-schema-transformer ./schema.prisma --deny knex_migrations --deny knex_migration_lock

Options:
  --print   Do not save
  --deny    Exlucde model from output
  --help    Help
  --version Version info
```

## Motivation

Using `snake_case` in database and automatically transform generated Prisma schema to `camelCase` with `@map` and `@@map` as needed to map the new name back to the database.

There is a [snippet](https://github.com/prisma/prisma/issues/1934#issuecomment-618063631) provided by @TLadd, but I found regex a bit unreliable.

## How does it work

`getDMMF` parses the Prisma schema file into dmmf(datamodel meta format), which we can use to do some post-processing on the Prisma internal data structure.

### Deserializer

There does not seem to be a printer or a deserializer for `DMMF`, [source](https://github.com/prisma/prisma/issues/515#issuecomment-619999178). You can learn more about the implementation at [deserializer.ts](./src/deserializer.ts). It is responsible for converting a serialized Prisma schema, DMMF, back to plain text file.

It's hacky, but it works. Some test fixtures are taken from the `@prisma/sdk` repository for testing. We use the `getDMMF` method to compare the serialized structure of original and transformed Prisma schema, make sure they are identical.

### Transformer

Manipulate the naming of Model and Field to follow the `camelCase` naming convention.

- Model name is always singular.
- Field name is singular by default with the execption of `many-to-many` relation.

## License

This project is [MIT licensed](./LICENSE).

