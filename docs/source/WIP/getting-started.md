---
title: Getting Started with Apollo Server
---

This tutorial helps you:

* Obtain a basic understanding of GraphQL principles
* Define a GraphQL **schema** that represents the structure of your data set
* Run an instance of Apollo Server that lets you execute queries against your schema

This tutorial assumes that you are familiar with the command line and
TypeScript or JavaScript, and that you have a recent version of Node.js (14+) installed.

If you do not already have TypeScript installed on your computer, install it now:
```bash
npm install -g typescript
```

> This tutorial walks you through installing and configuring Apollo Server.
> For a more complete introduction to the entire Apollo platform, check out [Odyssey](https://www.apollographql.com/tutorials/lift-off-part1), Apollo's new learning platform.

## Step 1: Create a new project

1. From your preferred development directory, create a directory for a new project and `cd` into it:

  ```bash
    mkdir graphql-server-example
    cd graphql-server-example
  ```

2. Initialize a new Node.js project with `npm` (or another package manager you
prefer, such as Yarn):

  ```bash
    npm init --yes
  ```

Your project directory now contains a `package.json` file.

## Step 2: Install dependencies
Applications that run Apollo Server require two top-level dependencies:
<!-- TODO: CHECK THIS LINK once packaged published -->
* [`@apollo/server`](https://www.npmjs.com/package/@apollo/server) is the core library for Apollo Server itself, which helps you define the shape of your data and how to fetch it.
* [`graphql`](https://npm.im/graphql) is the library used to build a GraphQL schema and execute queries against it.

Run the following command to install both of these dependencies and save them in
your project's `node_modules` directory:

```bash
npm install @apollo/server graphql
```

Also create an empty `index.ts` file in your project's root directory:

```bash
touch src/index.ts
```

To keep things simple, `index.ts` will contain **all** of the code for this example application.

### Set up TypeScript
Next we'll set up and configure TypeScript to compile our code. We'll install `typescript`:

```bash
npm install typescript
```

Now run the `tsc --init` command which will create a `tsconfig.json` file:

```bash
tsc --init
```

Inside your new `tsconfig.json` file add the following configuration:

```json title="tsconfig.json"
{
  "compilerOptions": {
    "rootDirs": ["src"],
    "outDir": "dist",
    "lib": ["es2019"],
    "target": "es2019",
    "module": "esnext",
    "moduleResolution": "node",
    "esModuleInterop": true
  }
}
```

For more information on any of the compiler options above check out the [TypeScript Compiler docs](https://www.typescriptlang.org/tsconfig).

Finally we'll add a `type` and `script` to the `package.json` file you previously created:
```json title="package.json"
{
  // ...etc.
  "type": "module",
  "scripts": {
    "start": "tsc && node ./dist/index.js"
  },
  // other dependencies
}
```

Setting your project's type to `module` will enable you to use a top-level [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), which can make your code easier to read.

## Step 3: Define your GraphQL schema

Every GraphQL server (including Apollo Server) uses a **schema**
to define the structure of data that clients can query.
In this example, we'll create a server for querying a collection
of books by title and author.

Open `index.ts` in your preferred editor and paste the following into it:

```js title="index.ts"
const { ApolloServer, standaloneServer, gql } = require('@apollo/server');

// A schema is a collection of type definitions (hence "typeDefs")
// that together define the "shape" of queries that are executed against
// your data.
const typeDefs = gql`
  # Comments in GraphQL strings (such as this one) start with the hash (#) symbol.

  # This "Book" type defines the queryable fields for every book in our data source.
  type Book {
    title: String
    author: String
  }

  # The "Query" type is special: it lists all of the available queries that
  # clients can execute, along with the return type for each. In this
  # case, the "books" query returns an array of zero or more Books (defined above).
  type Query {
    books: [Book]
  }
`;
```

This snippet defines a simple, valid GraphQL schema. Clients will be able to execute a query named `books`, and our server will return an array of zero or more `Book`s.

## Step 4: Define your data set

Now that we've defined the _structure_ of our data, we can define the data itself.

Apollo Server can fetch data from any source you connect to (including
a database, a REST API, a static object storage service, or even another GraphQL
server). For the purposes of this tutorial, we'll just hardcode some example data.

Add the following to the bottom of `index.ts`:

```js title="index.ts"
const books = [
  {
    title: 'The Awakening',
    author: 'Kate Chopin',
  },
  {
    title: 'City of Glass',
    author: 'Paul Auster',
  },
];
```

This snippet defines a simple data set that clients can query. Notice that the two objects in the array each match the structure of the `Book` type we defined in our schema.

## Step 5: Define a resolver

We've defined our data set, but Apollo Server doesn't know that it should _use_ that data set when it's executing a query. To fix this, we create a **resolver**.

Resolvers tell Apollo Server _how_ to fetch the data associated with a particular type. Because our `Book` array is hardcoded, the corresponding resolver is straightforward.

Add the following to the bottom of `index.ts`:

```js title="index.ts"
// Resolvers define the technique for fetching the types defined in the
// schema. This resolver retrieves books from the "books" array above.
const resolvers = {
  Query: {
    books: () => books,
  },
};
```

## Step 6: Create an instance of `ApolloServer`

We've defined our schema, data set, and resolver. Now we just need to provide
this information to Apollo Server when we initialize it.

Add the following to the bottom of `index.ts`:

```js title="index.ts"
// The ApolloServer constructor requires two parameters: your schema
// definition and your set of resolvers and returns an ApolloServerStandalone instance.
const apolloServerInstance = new ApolloServer({
  typeDefs,
  resolvers,
});

// Passing in your newly created ApolloServerStandalone instance to the
// standaloneServer function will enable you to start listening to your server
const { url } = await standaloneServer(apolloServerInstance, {
  async context() {
    return { token: 'hello' };
  },
}).listen({ port: 4000 });

console.log(`🚀  Server ready at: ${url}`);
```

## Step 7: Start the server

We're ready to start our server! Run the following from your project's root
directory:

```bash
npm start
```

After TypeScript finishes compiling, you should see the following output:

```
🚀 ApolloServer listening at: http://localhost:4000/
```

We're up and running!

## Step 8: Execute your first query

We can now execute GraphQL queries on our server. To execute our first query, we can use [**Apollo Sandbox**](/studio/explorer/sandbox/).

Visit `http://localhost:4000` in your browser. Apollo Server's default landing page appears:

<!-- TODO add and double check all these images before publishing -->
<img class="screenshot" src="./images/as-landing-page.jpg" alt="Apollo Server default landing page" width="350"/>

Click **Query your server** to open Sandbox.

> You can also:
>
> * Select **Automatically redirect to Studio next time** if you want to open Sandbox automatically whenever you visit `localhost:4000`
> * Open Sandbox directly at [studio.apollographql.com/sandbox](https://studio.apollographql.com/sandbox)

<!-- TODO check all the below images -->
<img class="screenshot" src="./images/sandbox.jpg" alt="Apollo Sandbox"/>

The Sandbox UI includes:

* An Operations panel for writing and executing queries (in the middle)
* A Response panel for viewing query results (on the right)
* Tabs for schema exploration, search, and settings (on the left)
* A URL bar for connecting to other GraphQL servers (in the upper left)

Our server supports a single query named `books`. Let's execute it!

Here's a GraphQL **query string** for executing the `books` query:

```graphql
query GetBooks {
  books {
    title
    author
  }
}
```

Paste this string into the Operations panel and click the blue button in the upper right. The results (from our hardcoded data set) appear in the Response panel:

<img class="screenshot" src="./images/sandbox-response.jpg" width="400" alt="Sandbox response panel"/>

<!-- TODO: check if this is true -->
> **Note:** If your server is deployed to an environment where `NODE_ENV` is set to `production`, introspection is **disabled** by default. This prevents Apollo Sandbox from working properly. To enable introspection, set `introspection: true` in [the options to `ApolloServer`'s constructor](./api/apollo-server/#constructor).

One of the most important concepts of GraphQL is that clients can choose to query _only for the fields they need_. Delete `author` from the query string and execute it again. The response updates to include only the `title` field for each book!

## Combined example

You can view and fork this complete example on CodeSandbox:
<!-- TODO: Recreate this sandbox -->
<a href="https://codesandbox.io/s/github/apollographql/docs-examples/tree/main/apollo-server/v3/getting-started?fontsize=14&hidenavigation=1&theme=dark">
  <img alt="Edit server-getting-started" src="https://codesandbox.io/static/img/play-codesandbox.svg" />
</a>

## Next steps

This example application is a great starting point for working with
Apollo Server. Check out the following resources to learn more about the basics
of schemas, resolvers, and deployment:

* [Schema basics](./schema/schema/)
* [Resolvers](./data/resolvers/)
* [Deploying with Heroku](./deployment/heroku/)

Want to learn how to modularize and scale a GraphQL API?  Check out the [Apollo Federation Docs](/federation) to learn how a federated architecture can create a unified *supergraph* that combines multiple GraphQL APIs.



