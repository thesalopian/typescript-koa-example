# A Node - Koa - Typescript Project

This example provides a good project setup and workflow for writing a Node api rest in TypeScript using KOA and an SQL DB.

Koa provides new web framework designed by the team behind Express, which promises a smaller, more expressive, and more robust foundation for web applications and APIs. Through leveraging generators Koa allows you to ditch callbacks and greatly increase error-handling. Koa does not bundle any middleware within core, and provides an elegant suite of methods that make writing servers fast and enjoyable.

Note: You *must* set an Authorization header to pass the JWT middleware:

HEADER
```
Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

AVAILABLE ENDPOINTS

| method             | resource         | description                                                                                    |
|:-------------------|:-----------------|:-----------------------------------------------------------------------------------------------|
| `GET`              | `/`              | Simple hello world response                                                                    |
| `GET`              | `/jwt`           | Dummy endpoint to show how JWT info gets stored in ctx.state                                   |
| `GET`              | `/users`         | returns the collection of users present in the DB                                              |
| `GET`              | `/users/:id`     | returns the specified id user                                                                  |
| `POST`             | `/users`         | creates a user in the DB (send object user includued in request's body)                        |
| `PUT`              | `/users/:id`     | updates an already created user in the DB (send object user in request's body)                 |
| `DELETE`           | `/users/:id`     | deletes a user from the DB (JWT token user ID must match the user you want to delete)          |


## Pre-reqs
To build and run this app locally you will need:
- Install [Node.js](https://nodejs.org/en/)

## Features:
 * Nodemon - server auto-restarts when code changes
 * Koa v2
 * TypeORM (SQL DB) with basic CRUD included
 * Class-validator - Decorator based entities validation
 * Docker-compose - ready to go for those that like its

## Included middleware:
 * koa-router
 * koa-bodyparser
 * Winston Logger
 * JWT auth koa-jwt
 * Helmet (security headers)
 * CORS

# Getting Started

- Install dependencies
```
cd <project_name>
npm install
```
- Run the project directly in TS
```
npm run watch-server
```

- Build and run the project in JS
```
npm run build
npm run start
```

## Docker (optional)
A docker-compose file allows the easy deployment of a PostgreSQL (already setting user, pass and dbname as the ORM config permits) and an ADMINER image (easy web db client).

You can execute the command 'docker-compose up' once you have Docker installed, and both the PostgreSQL server and the Adminer client will run on ports 5432 and 8080 respectively with all the config you need to start playing around. 

If you use Docker natively, the host for the server which you will need to include in the ORM configuration file will be localhost. 

## Setting up the Database - ORM
This API works with an SQL database, using [TypeORM](https://github.com/typeorm/typeorm). In this case we use PostgreSQL, and the reason why you can find 'pg' in the package.json. If you where to use a different SQL database remember to install the correspondent driver.

The ORM configuration and connection to the database can be specified in the file 'ormconfig.json'. This depends on the connection to the database in 'server.ts' file because a environment variable containing databaseUrl used to set the connection data. 

Please note when serving the project directly with typescript files using ts-node,the configuration for the ORM should specify the typescipt files path, but once the project gets built (transpiled) and run as plain js, it will be needed to change it accordingly to find the built js files:

```
"entities": [
      "dist/entity/**/*.js"
   ],
   "migrations": [
      "dist/migration/**/*.js"
   ],
   "subscribers": [
      "dist/subscriber/**/*.js"
   ]
```

Notice that if NODE_ENV gets set to development, the ORM config won't be using SSL to connect to the DB. Otherwise it will.

```
createConnection({
    ...
    extra: {
        ssl: config.DbSslConn, // if not development, will use SSL
    }
 })
```

You can find an implemented **CRUD of the entity user** in the correspondent controller controller/user.ts and its routes in routes.ts file.

## Entities validation
This project uses the library class-validator, a decorator-based entity validation, used directly in the entities files as follows:
```
export class User {
    @Length(10, 100) // length of string email must be between 10 and 100 characters
    @IsEmail() // the string must comply with an standard email format
    @IsNotEmpty() // the string can't be empty
    email: string;
}
```
Once the decorators get set in the entity, you can validate from anywhere as follows:
```
const user = new User();
user.email = "avileslopez.javier@gmail"; // should not pass, needs the ending .com to be a valid email

validate(user).then(errors => { // errors in an array of validation errors
    if (errors.length > 0) {
        console.log("validation failed. errors: ", errors); // code will get here, printing an "IsEmail" error
    } else {
        console.log("validation succeed");
    }
});
```

For further documentation regarding validations see [class-validator docs](https://github.com/typestack/class-validator).


## Environment variables
Create a .env file (or just rename the .example.env) containing all the env variables you want to set, dotenv library will take care of setting them. This project use four variables:

 * PORT -> port where the server will be started on, Heroku will set this env variable automatically
 * NODE_ENV -> environment, development value will set the logger as debug level, also important for CI. In addition will determine if the ORM connects to the DB through SSL or not.
 * JWT_SECRET -> secret value, JWT tokens should be signed with this value
 * DATABASE_URL -> DB connection data in connection-string format.

## Getting TypeScript
You add TypeScript to any project with `npm`.
```
npm install -D typescript
```

## Project Structure
The most obvious difference in a TypeScript + Node project becomes the folder structure.
TypeScript (`.ts`) files live in your `src` folder and after compilation are output as JavaScript (`.js`) in the `dist` folder.

The folder has the following structure:

| Name                     | Description                                                                                   |
| ------------------------ | --------------------------------------------------------------------------------------------- |
| **dist**                     | Contains the distributable (or output) from your TypeScript build. This contains the code you ship  |
| **src**                      | Contains your source code that will be compiled to the dist dir                               |
| **src**/server.ts            | Entry point to your KOA app                                                                   |
| .copyStaticAssets.ts     | Build script that copies images, fonts, and JS libs to the dist folder                        |
| package.json             | File that contains npm dependencies as well as [build scripts](#what-if-a-library-isnt-on-definitelytyped)                          |
| docker-compose.yml       | Docker PostgreSQL and Adminer images in case you want to load the db from Docker              |
| tsconfig.json            | Config settings for compiling server code written in TypeScript                               |
| tslint.json              | Config settings for TSLint code style checking                                                |
| .example.env             | Env variables file example to be renamed to .env                                              |

## Configuring TypeScript compilation
TypeScript uses the file `tsconfig.json` to adjust project compile options.
Let's dissect this project's `tsconfig.json`, starting with the `compilerOptions` which details how your project gets compiled. 

```json
    "compilerOptions": {
        "module": "commonjs",
        "target": "es2017",
        "lib": ["es6"],
        "moduleResolution": "node",
        "sourceMap": true,
        "outDir": "dist",
        "baseUrl": ".",
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true,  
        }
    },
```

| `compilerOptions` | Description |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `"module": "commonjs"`             | The **output** module type (in your `.js` files). Node uses commonjs, so we use that too               |
| `"target": "es2017"`               | The output language level. Node supports ES2017, so we can target that here                            |
| `"lib": ["es6"]`                   | Needed for TypeORM.                                             |
| `"moduleResolution": "node"`       | TypeScript attempts to mimic Node's module resolution strategy. Read more [here](https://www.typescriptlang.org/docs/handbook/module-resolution.html#node)                             |
| `"sourceMap": true`                | We want source maps to be output along side our JavaScript.     |
| `"outDir": "dist"`                 | Location to output `.js` files after compilation                |
| `"baseUrl": "."`                   | Part of configuring module resolution.                          |
| `"experimentalDecorators": true`   | Needed for TypeORM. Allows use of @Decorators                   |
| `"emitDecoratorMetadata": true`    | Needed for TypeORM. Allows use of @Decorators                   |


The rest of the file define the TypeScript project context.
The project context basically consitutes a set of options that determine which files get compiled when we invoke the compiler with a specific `tsconfig.json`.
In this case, we use the following to define our project context: 
```json
    "include": [
        "src/**/*"
    ]
```
`include` takes an array of glob patterns of files to include in the compilation.
This project has a fairly simple nature and all of our .ts files are under the `src` folder.
For more complex setups, you can include an `exclude` array of glob patterns that removes specific files from the set defined with `include`.
You can also use a `files` option which takes an array of individual file names which overrides both `include` and `exclude`.


## Running the build
All the different build steps are orchestrated via [npm scripts](https://docs.npmjs.com/misc/scripts).
Npm scripts basically allow us to call (and chain) terminal commands via npm.
This is nice because most JavaScript tools have easy to use command line utilities allowing us to not need grunt or gulp to manage our builds.
If you open `package.json`, you will see a `scripts` section with all the different scripts you can call.
To call a script, simply run `npm run <script-name>` from the command line.
You will notice that npm scripts can call each other which makes it easy to compose complex builds out of simple individual build scripts.
Below you can find a list of all the scripts this template has available:


| Npm Script | Description |
| ------------------------- | ------------------------------------------------------------------------------------------------- |
| `start`                   | Does the same as 'npm run serve'. Can be invoked with `npm start`                                 |
| `build`                   | Full build. Runs ALL build tasks (`build-ts`, `tslint`, `copy-static-assets`)                     |
| `serve`                   | Runs node on `dist/server/server.js` which defines the apps entry point                                |
| `watch-server`            | Nodemon, process restarts if crashes. Continuously watches `.ts` files and re-compiles to `.js`   |
| `build-ts`                | Compiles all source `.ts` files to `.js` files in the `dist` folder                               |
| `tslint`                  | Runs TSLint on project files                                                                      |
| `copy-static-assets`      | Calls script that copies JS libs, fonts, and images to dist directory                             |

# TSLint
TSLint provides a code linter which mainly helps catch minor code quality and style issues.
TSLint seems very similar to ESLint or JSLint but specifically targets TypeScript.

## TSLint rules
Like most linters, TSLint has a wide set of configurable rules as well as support for custom rule sets.
All rules get configured through editing `tslint.json`.
In this project, we use a fairly basic set of rules with no additional custom rules.

## Running TSLint
Like the rest of our build steps, we use npm scripts to invoke TSLint.
To run TSLint you can call the main build script or just the TSLint task.
```
npm run build   // runs full build including TSLint
npm run tslint  // runs only TSLint
```
Notice that TSLint does not form Codepart of the main watch task to keep things simple.

# Logging
Winston provides a simple and universal logging library with support for multiple transports.

The project uses a "logger" middleware passing a winstonInstance. Current configuration of the logger gets defined in the file "logging.ts". It will log 'error' level to an error.log file and 'debug' or 'info' level (depending on NODE_ENV environment variable, debug if == development) to the console.

```
// Logger middleware -> use winston as logger (logging.ts with config)
app.use(logger(winston));
```

# Authentication - Security
To keep the API as clean as possible, we use a client that in turn uses an auth provider such as Auth0. The client making requests to the API should include the JWT in the Authorization header as "Authorization: Bearer <jwt_token>". This HS256 secret allows your api and your client to sign the token, so make sure you keep it hidden.

As seen within the server.ts file, a JWT middleware passes the secret from an environment variable. The middleware will validate that every request to the routes below, MUST include a valid JWT signed with the same secret. The middleware will automatically set the payload information in ctx.state.user.

```
// JWT middleware -> below this line, routes are only reached if you have a valid JWT toke, secret as env variable
app.use(jwt({ secret: config.jwtSecret }));
```
Go to the website [https://jwt.io/](https://jwt.io/) to create JWT tokens for testing/debugging purposes. Select algorithm HS256 and include the generated token in the Authorization header to pass through the jwt middleware.

Custom 401 handling -> if you don't want to expose koa-jwt errors to users:
```
app.use(function(ctx, next){
  return next().catch((err) => {
    if (401 == err.status) {
      ctx.status = 401;
      ctx.body = 'Protected resource, use Authorization header to get access\n';
    } else {
      throw err;
    }
  });
});
```

If you want to authenticate from the API, and you like the idea of an auth provider like Auth0, have a look at [jsonwebtoken — JSON Web Token signing and verification](https://github.com/auth0/node-jsonwebtoken)

## CORS
This boilerplate uses @koa/cors, a simple CORS middleware for koa. If you are not sure what this does, click [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

```
// Enable CORS with default options
app.use(cors());
```
Have a look at [Official @koa/cors docs](https://github.com/koajs/cors) in case you want to specify 'origin' or 'allowMethods' properties.


## Helmet
This boilerplate uses koa-helmet, a wrapper for helmet to work with koa. It provides important security headers to make your app more secure by default. 

We use the best practices as defined in [helmet](https://github.com/helmetjs/helmet). Helmet offers 11 security middleware functions (clickjacking, DNS prefetching, Security Policy...), using *good* defaults.

```
// Enable helmet with default options
app.use(helmet());
```

Have a look at [Official koa-helmet docs](https://github.com/venables/koa-helmet) in case you want to customize which security middlewares get used.


# Dependencies
Dependencies are managed through `package.json`.
In that file you'll find two sections:
## `dependencies`

| Package                         | Description                                                           |
| ------------------------------- | --------------------------------------------------------------------- |
| dotenv                          | Loads environment variables from .env file.                           |
| koa                             | Node.js web framework.                                                |
| koa-bodyparser                  | A bodyparser for koa.                                                 |
| koa-jwt                         | Middleware to validate JWT tokens.                                    |
| koa-router                      | Router middleware for koa.                                            |
| koa-helmet                      | Wrapper for helmet, important security headers to make app more secure| 
| @koa/cors                       | Cross-Origin Resource Sharing(CORS) for koa                           |
| pg                              | PostgreSQL driver, needed for the ORM.                                |
| reflect-metadata                | Used by typeORM to implement decorators.                              |
| typeorm                         | A very cool SQL ORM.                                                  |
| winston                         | Logging library.                                                      |
| class-validator                 | Decorator based entities validation.                                  |
| pg-connection-string            | Parser for database connection string                                 |


## `devDependencies`

| Package                         | Description                                                           |
| ------------------------------- | --------------------------------------------------------------------- |
| @types                          | Dependencies in this folder are `.d.ts` files used to provide types   |
| nodemon                         | Utility that automatically restarts node process when it crashes      |
| ts-node                         | Enables directly running TS files. Used to run `copy-static-assets.ts`|
| tslint                          | Linter (similar to ESLint) for TypeScript files                       |
| typescript                      | JavaScript compiler/type checker that boosts JavaScript productivity  |
| shelljs                         | Portable Unix shell commands for Node.js                              |

To install or update these dependencies you can use `npm install` or `npm update`.

