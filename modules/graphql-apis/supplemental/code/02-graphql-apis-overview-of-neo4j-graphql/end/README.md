[![Open in CodeSandbox](https://img.shields.io/badge/Open%20in-CodeSandbox-blue?style=flat-square&logo=codesandbox)](https://codesandbox.io/s/github/neo4j-contrib/training-v3/modules/graphql-apis/supplemental/code/02-graphql-apis-overview-of-neo4j-graphql/end?file=/schema.graphql)

# Bookstore GraphQL API

This directory contains a Node.js GraphQL API application using [`@neo4j/graphql`](https://www.npmjs.com/package/@neo4j/graphql).

Try it live on CodeSandbox [here](https://codesandbox.io/s/github/neo4j-contrib/training-v3/modules/graphql-apis/supplemental/code/02-graphql-apis-overview-of-neo4j-graphql?file=/schema.graphql)

## Setup

First, edit `.env`, replacing the defaults with your database connection string, user, and database (optional):

```
NEO4J_URI=
NEO4J_USER=
NEO4J_PASSWORD=
```

Next, install dependencies.

```
npm install
```

Then start the API application,

```
npm run start
```

This will start a local GraphQL API server at `localhost:4000`.

## Example GraphQL Queries

```GraphQL
{
  books {
    title
    reviews {
      rating
      text
      author {
        username
      }
    }
  }
}
```

## Exercise Solutions

**Adding `Author` and `Subject` to the GraphQL schema.**

Add the `Author` and `Subject` types, using the `@relationship` directive to describe the relationships according to our data model diagram.

```GraphQL
type Author {
  name: String!
  books: [Book] @relationship(type: "AUTHOR_OF", direction: OUT)
}

type Subject {
  name: String!
  books: [Book] @relationship(type: "ABOUT", direction: IN)
}
```

Also, add `authors` and `subjects` fields to the `Book` type.

```GraphQL
type Book {
  isbn: ID!
  title: String
  price: Float
  description: String
  authors: [Author] @relationship(type: "AUTHOR_OF", direction: IN)
  subjects: [Subject] @relationship(type: "ABOUT", direction: OUT)
  reviews: [Review] @relationship(type: "REVIEWS", direction: IN)
}
```

**Creating authors and connecting to their books**

```GraphQL
mutation {
  createAuthors(
    input: [
      {
        name: "Marty Cagan"
        books: { connect: { where: { node: { title: "Inspired" } } }}
      }
      {
        name: "Winston Graham"
        books: { connect: { where: { node: { title: "Ross Poldark" } } }}
      }
      {
        name: "Mark Needham"
        books: { connect: { where: { node:{ title: "Graph Algorithms" } } }}
      }
      {
        name: "Amy E. Hodler"
        books: { connect: { where:{ node: { title: "Graph Algorithms" } }} }
      }
    ]
  ) {
    authors {
      name
      books {
        title
      }
    }
  }
}
```

**Adding subjects to the graph**

There are a few different ways to approach this one, let's see how we can accomplish this using the `updateBooks` mutation:

```GraphQL
mutation {
  inspired: updateBooks(
    where: { title: "Inspired" }
    create: {
      subjects: [
        { node: { name: "Product management" } }
        { node: { name: "Design" } }
      ]
    }
  ) {
    books {
      title
      subjects {
        name
      }
    }
  }

  poldark: updateBooks(
    where: { title: "Ross Poldark" }
    create: {
      subjects: [
        { node: { name: "Historical fiction" } }
        { node: { name: "Cornwall" } }
      ]
    }
  ) {
    books {
      title
      subjects {
        name
      }
    }
  }

  algorithms: updateBooks(
    where: { title: "Graph Algorithms" }
    create: {
      subjects: [
        { node: { name: "Graph theory" } }
        { node: { name: "Neo4j" } }
      ]
    }
  ) {
    books {
      title
      subjects {
        name
      }
    }
  }
}
```
