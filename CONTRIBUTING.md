# Contributing to WordPress JSON Schemas

Bug fixes and new schema contributions are very welcome.

There is no fully automatic process to create these schemas. A schema for a PHP object is created from the public properties of its class, and a schema for a REST API response is created from an OPTIONS request to its REST API endpoint.

* [Creating a REST API response schema](#creating-a-rest-api-response-schema)
* [Creating a PHP object schema](#creating-a-php-object-schema)
* [Releasing a new version](#releasing-a-new-version)

## Setting up Locally

### Prerequisites

* [Composer](https://getcomposer.org/)
* [Node](https://nodejs.org/)

### Setup

1. Install the PHP dependencies:

       composer install

2. Install the Node dependencies:

       npm install

## Creating a REST API response schema

The WordPress REST API response doesn't fully adhere to the JSON schema spec, so we need to tweak its output in order to generate a valid schema and to increase its specificity.

* Perform an `OPTIONS` request to an endpoint that returns the object (or array of the objects) that you want to create a schema for. For example `/wp-json/wp/v2/users`.
* Use the `schema` property from the response as a basis for the schema in a new file in `schemas/rest-api` for a single entity and `schemas/rest-api/collections` for collections.
* Set the `$schema`, `$id`, `title`, and `description` properties using the same format as existing schemas.
* Remove any properties that have a `context` of `[]` as these are write-only and not exposed. So far I've only seen this for the `password` property of a user.
* Remove any `properties` properties that have a value of `[]`.
* All properties that have a `context` of `view` should be added to the `required` property.
* Remove the `context` and `required` properties from each property as these are not valid JSON schema properties.
* Rename any `readonly` properties to `readOnly`.
* Add `additionalProperties` information to any `object` properties as appropriate.
* Cross-reference the properties with those in the `get_item_schema()` method of the controller class. There may be properties that are conditionally added. Add them to the schema if so.
* Add the schema to the `REST_API` property in the root `schema.json` using a `$ref` to the schema file.
* Add the new schema property key to the `required` property in the root `schema.json`.
* In `tests/output` create a file for the new object type
  - Start by copying an existing file such as `post.php` which is for `/wp/v2/posts`
  - The command should perform one or more REST API requests to the endpoint and pass the responses to the `save_rest_array()` function which saves them as JSON during the tests
* Run `composer run test` to validate and test the schemas.
* Run `npm run build-wp-types` and check the output of `packages/wp-types/index.ts`.
* Add documentation for the schema in both `readme.md` and `packages/wp-types/readme.md`.

## Creating a PHP object schema

The schema for a PHP object is created using the docblocks from its class properties and a lot of manual adjustment.

* JSON encode an instance of the class you're interested in. For example:

      echo json_encode( get_post( 123 ), JSON_PRETTY_PRINT );

* Make a note of all of its property names.
* Open the corresponding PHP file for the class.
* Correlate this public property list with those defined in the class.
  - Why? An object can have a public property that's not declared on the class. In this case, you'll need to provide the type and description yourself.
* Wave your hands over your keyboard.
* The root `additionalProperties` property should be set to `false`.
* The `readOnly` property should be set to `true` for the unique ID property of the object if appropriate.
* Add the schema to the root `schema.json` using a `$ref` to the schema file.
* Add the new schema property key to the `required` property in the root `schema.json`.
* In `tests/output` create a file for the new object type
  - Start by copying an existing file such as `error.php`
  - The file should pass an array of one or more objects of this type to the `save_object_array()` function which saves it as JSON during the tests
* Run `composer run test` to validate and test the schemas.
* Run `npm run build-wp-types` and check the output of `packages/wp-types/index.ts`.
* Add documentation for the schema in both `readme.md` and `packages/wp-types/readme.md`.

## Updating schemas for a new WordPress release

*     composer require --dev roots/wordpress-full=dev-main
*     composer update
*     composer test

Address any test failures by updating the corresponding schema files.

Address any newly added REST API routes by adding them to both readme files and adding a schema if necessary or documenting them as "Todo".

For release, update the WordPress version number in composer.json and the readme files.

## Releasing a new version

*     npm install
*     composer install
*     composer run test
* Ensure the tests pass
* Ensure there are no changed files in git
*     npm version <major|minor|patch>
*     git push
*     git push --tags
* Publish a release with a nice changelog

When a tag is pushed, the following packages will be deployed to npm automatically via GitHub Actions:

* [wp-json-schemas](https://www.npmjs.com/package/wp-json-schemas)
* [wp-types](https://www.npmjs.com/package/wp-types)
