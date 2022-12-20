# GraphQL with ExpressJS


## About
- REST API: Stateless, client-independent.
- GraphQL API: Stateless, client-independent API for exchanging data with higher query flexibility
- API Cons. Require lots and lots of endpoints. API with params becomes hard to understand.
- GraphQL only send a POST request even if getting data. The POST request contains query expression to define the data that should be returned.
JSON object like structure
```
{ query 
  { user 
    { 
      name, 
      age 
    } 
  } 
}
```
- Three operations types: Query to retrieve Data with GET, Mutation to manipulate Data with (POST, PUT, PATCH, DELETE), Subscription by setting up realtime connection via Websockets
- GraphQL workflow: client -> POST request to graphQL -> Route operations like Query, Mutation, Subscription -> Controller like resolvers that contain server side logic


## Setup
- Install graphql and express graphql `npm install --save graphql express-graphql`
- In using GraphSQL, Apollo server is also a famous alternative to Express server.
- 
