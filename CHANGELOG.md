# Changelog

All notable changes to this project will be documented in this file.

The format is roughly based on [Keep a
Changelog](http://keepachangelog.com/en/1.0.0/).

This project intends to inhere to [Semantic
Versioning](http://semver.org/spec/v2.0.0.html), but has not yet reached 1.0 so
all APIs might be changed.

## Unreleased - xxxx-xx-xx

### Breaking Changes

- InputObject no longer has a serialize method - this is now handled by a
  SerializableArgument impl instead, which is generated by the InputObject
  derive.
- `Query` has been renamed to `Operation` to make it clear it's used for both
  queries & mutations.
- `Query::new` is now `Operation::query`

### New Features

- InputObjects can now be derived and will be generated by querygen.
- Querygen output is now tested more thoroughly - should be less changes
  required by users just to get it to compile.
- Cynic now supports running & generating code for mutations.

### Removed Features

- Removed the `optimised_query_modules` feature from codegen, as it invovled
  more code than it was worth to keep it around. Functionally this should make
  no difference, though it may change performance characteristics of compiling
  cynic code. Didn't seem to make a significant difference when I was using it
  though.

### Bug Fixes

- Fixed a compile issue in the generated `query_dsl` for schemas with fields
  with > 1 required argument.
- Fixed an issue that required users to add `serde_json` to their dependencies.
  We now re-export it as `cynic::serde_json` and use that in our derive output.
- querygen now adds `rename_all="SCREAMING_SNAKE_CASE"` to Enums by default -
  the GQL convention is to have them in this format and querygen was already
  doing the transformation into the `PascalCase` rust usually uses so this
  should make things more likely to work by default.
- Removed fontawesome from the querygen HTML. Think I added this along with
  bulma but it's not being used, and adds 400kb to the payload.
- Fixed a bug where querygen would not snake case field names when generating
  `QueryFragment`s.
- querygen will now take references to arguments rather than ownership (which
  didn't work for most non-enum types).
- Fixed an issue where querygen was adding ID literals as Strings in arguments,
  rather than IDs.

## v0.8.0 - 2020-08-16

### Breaking Changes

- Integer fields are now i32 rather than i64 inline with the GraphQL spec. If
  larger integers are required a custom scalar should be used.
- The `cynic_arguments` attribute for passing arguments to GraphQL fields is
  now named `arguments`

### New Features

- querygen-web now incorporates graphiql & graphiql explorer, to make testing &
  building queries easier.
- querygen now supports bare selection sets, they're assumed to be queries.
  Quite easy to create these in GraphiQL/GraphqlExplorer, and they work for
  queries so seemed important.
- Arguments are now converted using the `IntoArgument<T>` trait - default
  conversions are provided for `Option` and reference types, so users don't
  always have to wrap Options in `Some` or explicitly clone their arguments.
- As a result of the above, cynic will no longer stop compiling when a schema
  changes a required argument to optional.

### Bug Fixes

- Fixed an issue with cynic-querygen where it guessed the name for the root of
  a query and crashed out if it was wrong (which was often).
- Fixed an issue where querygen would fail if given a query with a hardcoded
  enum value (#33)
- Integers are now i32 rather than i64, inline with the GraphQL spec. If
  larger integers are required a custom scalar should be used.
- Querygen now puts `argument_struct` attrs on types that have arguments rather
  than just types that have children with arguments. (#37)
- Fixed an issue where querygen would use the name of the query as the
  `graphql_type` on the root struct of named queries.
- Fixed a bunch of broken links in the book.

## v0.7.0 - 2020-06-23

### Breaking Changes

- `SerializableArgument`s are now required to be `Send`. Found this was
  required for using cynic in an async context. May revisit at some point to
  see if it's 100% required.

## v0.6.0 - 2020-06-17

### New Features

- `cynic::Id` now derives PartialEq, Hash & Eq
- Added `cynic::Id::new` function

### Bug Fixes

- Using a `query_module` should no longer cause errors on an individual derive
  to be attributed to the `query_module` span - the error information should
  now be associated with the derive it originated from.
- Fixed some dead code warnings in the selection builders output by query DSL

## v0.5.0 - 2020-06-14

### Breaking Changes

- `Query::body` no longer exists, the `Query` itself is now directly
  serializable, and exposes the `query` type itself. Errors that were
  previously exposed by `Query::body()` will now be surfaced when serializing a
  Query.
- `Argument::new` has been updated to take a `SerialiableArgument` itself.
- Removed `selection_set::Error`.
- `SerializableArgument::serialize` & `Scalar::encode` now return
  `Box<std::error::Error>` errors rather than `()`

### Bug Fixes

- `cynic-codegen` will now build with the rustfmt feature disabled.
- Removed some unwraps that I lazily put in and forgot to remove.

## v0.4.0 - 2020-06-12

### Breaking Changes

- `schema_path` parameters are now relative to `CARGO_MANFIEST_DIR` rather than
  the current working directory. This fixes an issue with cargo workspace
  projects where doc-tests would be relative to the sub-crate but cargo builds
  were relative to the workspace root. For projects not using workspaces this
  will probably make no difference
- The `query_dsl` has been reworked. Before each field with arguments had one
  or two structs: one for optional arguments & one for required arguments, and
  these were passed to the selector function before the selection set argument.
  Now each selector function takes the required arguments, and then returns a
  struct that follows the builder pattern to allow for optional arguments to be
  added. This fixes a few issues and is a bit more ergonomic.

### Bug Fixes

- Fixed a bug where any type used as an optional argument needed to implement
  Default. This was fixed by the `query_dsl` rework.
- Fixed a bug where optional arguments that were enums or interfaces had to be
  provided or type inference problems occurred. This was also fixed by the
  `query_dsl` rework.

## v0.3.0 - 2020-06-12

### New Features

- Added chrono::DateTime scalar support behind a chrono feature flag.
- The `cynic::selection_set` module and all it's contents are now documented.
- `QueryBody` now exposes it's arguments & query fields for greater flexibility
  (and use in snapshot testing etc.)

### Bug Fixes

- Generated `query_dsl` now disables unused import warnings where appropriate.
- Exposed `Id::inner` & `Id::into_inner` functions - these were meant to be
  public but were not
- Cleaned up a ton of compiler warnings - mostly unused imports and a few unused
  variables
- `query_dsl` adds `allow(dead_code)` annotations so we don't get tons of dead
  code warnings when we're not exercising an entire schema.
- `query_dsl` no longer creates mutable `Vec` for fields without arguments -
  this was leading to tons of "doesn't need to be mutable" warnings.
- `ID` fields are now correctly given the `cynic::Id` type in `query_dsl` - previously
  they were being forced to String.
- `cynic::Id` is now a `cynic::Scalar`
- Fixed an issue in `derive(QueryFragment)` where Enums inside lists would not be
  treated as Enums.
- `DecodeError` now implements `std::error::Error`

## v0.2.0 - 2016-06-11

### Breaking Changes

- The generated `query_dsl` no longer contains generated enums - users should
  provide their own enums and `derive(cynic::Enum)` on them. Cynic querygen
  can be used to help with this.
- The generated `query_dsl` no longer contains generated input objects - users
  should provide their own structs and `impl cynic::InputObject` on them. A
  derive for this should be coming in the future.

### New Features

- Union types can be queried via `#[derive(InlineFragments)]` on an enum.
- Schemas that use interfaces are now supported, though interfaces are not
  yet queryable.
- `#[derive(QueryFragment)]` now explicitly checks for required/list type
  mismatches & other easy mistakes, and warns the user appropriately.
- Added an `output_query_dsl` function suitable for running inside build.rs
- Added a flatten option at the field level. When present this will flatten
  nested options & vectors into the provided type. Used to handle the common
  case in GQL where someone has defined an optional list of optionals. This is
  a pain in Rust, since the same thing can usually be represented by a
  non-optional list of non-optionals.
- Added a cynic-querygen for generating QueryFragment structs from a graphql
  schema & query. This currently has a WIP web interface and a WIP CLI, though
  neither of them are particularly user friendly at this point.
- Added a `cynic::query_module` attribute macro that can be applied to modules
  containing QueryFragments & InlineFragments. When this attribute is present
  the derive will be done for all QueryFragments & InlineFragments contained
  within. This allows users to omit some of the parameters these derives
  usually require, as the `query_module` attribute provides them and fills them
  in. These modules may be expanded in the future to provide more
  "intelligent" features.
- Added support for mapN up to N = 50, therefore adding support for GraphQL
  objects with up to 50 fields.
- Added new `cynic::Enum` derive that matches up a Rust enum with a GraphQL enum.
  `cynic-querygen` will automatically provide enums using this derive when a
  query includes an enum.
- Added `SelectionSet::and_then` for chaining decode operations on selection sets.
- Added `cynic::Scalar` derive for newtype structs so that users can easily
  define their own scalars. Also added support for this to cynic-querygen
- Added `cynic::Id` type to handle Ids in queries.
- Added `cynic::InputObject` trait to allow the `query_dsl` to handle
  InputObjects generically.

### Changed

- Split the procedural macros out into their own cynic-proc-macros crate.
  cynic-codegen now exists as a re-usable library for programatically
  doing the codegen.
- The IntoArguments trait is now named FromArguments and has had it's
  parameters switched up.
- Added a StarWars API example

### Bug Fixes

- Now supports schemas that define their root query types. Before we just
  assumed there was a type called query.
- Fixed a few things that stop the examples in the documentation from
  compiling.
- Fixed all the tests
- We now use the correct case for non built-in scalar types
- Fixed an issue that prevented propagation of argument structs into inner
  QueryFragments

## v0.1.2 - 2020-02-04

- No changes

## v0.1.1 - 2020-02-04

- Some tweaks to the documentation.

## v0.1.0 - 2020-02-03

- Initial release
