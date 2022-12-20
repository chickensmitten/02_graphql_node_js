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
- GraphQL workflow: client -> POST request to graphQL -> Route operations like Query, Mutation, Subscription in schema to define the data structure -> Controller like functions are resolvers that contain server side logic to resolve queries
- In using GraphSQL, Apollo server is also a famous alternative to Express server.


## Setup
- Install graphql and express graphql `npm install --save graphql express-graphql`
- Install validator `npm install --save validator`
- Create "/graphql/schema.js" "/graphql/resolvers.js"
- Following is an exampl code setup for schema.js
```
const { buildSchema } = require("graphql");

module.exports = buildSchema(`
  type TestData {
    text: String!
    views: Int!
  }

  type RootQuery { // or any other name like Query
    hello: TestData! // graphQL recognizes different query types like String, integer etc
  }

  schema {
    query: RootQuery
  }
`);
```
- Following is an exampl code setup for resolvers.js
```
module.exports = {
  hello() {
    return {
      text: "hello World!",
      views: 12345
    }
  }
};
```
- in "app.js" add the following code
```
const { graphqlHttp } = require("express-graphql");
const graphqlSchema = require("./graphql/schema");
const graphqlResolver = require("./graphql/resolvers");
...
app.use(
  '/graphql', 
  graphqlHttp({
    schema: graphqlSchema,
    rootValue: graphqlResolver
  })
);
```
- to access the code above, can go to postman then query "http://localhost:8080/graphql" with body
```
{
  "query": "{ hello { text views } }"
}
```
which will return
```
{ 
  "data": { 
    "hello": {
      "text": "Hello world!", 
      "views": 12345
    }
  }
}
```

## Mutations
- Example in using mutations in graphql
- example code in schema to first declare the allowed query schema
```
// /graphql/schema.js
module.exports = {
  type Post {
      _id: ID!
      title: String!
      content: String!
      imageUrl: String!
      creator: User!
      createdAt: String!
      updatedAt: String!
  }

  type User {
      _id: ID!
      name: String!
      email: String!
      password: String
      status: String!
      posts: [Post!]!
  }

  input UserInputData {
      email: String!
      name: String!
      password: String!
  }

  type RootMutation {
      createUser(userInput: UserInputData): User!
  }

  type RootQuery {
      login(email: String!, password: String!): AuthData!
      posts(page: Int): PostData!
      post(id: ID!): Post!
      user: User!
  }

  schema {
      query: RootQuery
      mutation: RootMutation
  }
};    
```
- then, in example code in resolver handle the incoming query
```
const User = require("../models/user");

module.exports = {
  createUser: async function({ userInput }, req) {
    // const email = args.userInput.email;
    const errors = [];
    if (!validator.isEmail(userInput.email)) {
      errors.push({ message: 'E-Mail is invalid.' });
    }
    if (
      validator.isEmpty(userInput.password) ||
      !validator.isLength(userInput.password, { min: 5 })
    ) {
      errors.push({ message: 'Password too short!' });
    }
    if (errors.length > 0) {
      const error = new Error('Invalid input.');
      error.data = errors;
      error.code = 422;
      throw error;
    }
    const existingUser = await User.findOne({ email: userInput.email });
    if (existingUser) {
      const error = new Error('User exists already!');
      throw error;
    }
    const hashedPw = await bcrypt.hash(userInput.password, 12);
    const user = new User({
      email: userInput.email,
      name: userInput.name,
      password: hashedPw
    });
    const createdUser = await user.save();
    return { ...createdUser._doc, _id: createdUser._id.toString() };    
  }
};
```
- to test for mutations, set `graphiql` to `true`. then go to "localhost:8080/graphql" in the browser to see the output. This makes it easier to play around with graphql compared with postman
```
app.use(
  '/graphql', 
  graphqlHttp({
    schema: graphqlSchema,
    rootValue: graphqlResolver,
    graphiql: true
  })
);
```
- to create a user based on the code above, run the following code in graphiql
```
mutation {
  createUser(userInput: {email: "test@gmail.com", name: "Max", password: "test"}) {
    _id
    email
  }
}

// data will be created in MongoDB and in the graphql, the output returns
{
  "data": {
    "createUser": {
      "_id": "322332423ee233242342".
      "email": "test@gmail.com"
    }
  }
}
```


## Validation
- validating graphql query with validator `npm install --save validator`
```
const validator = require("validator);

module.exports = {
  createUser: async function({ userInput }, req) {
    
    ...
    
    if (!validator.isEmail(userInput.email)) {
      errors.push({ message: 'E-Mail is invalid.' });
    }

    ...

  }
};  

```


## Handling Errors
- to handle error responses with `formatError` in "app.js"
```
// /app.js
app.use(
  '/graphql', 
  graphqlHttp({
    schema: graphqlSchema,
    rootValue: graphqlResolver,
    graphiql: true,
    formatError(err) {
      if (!err.originalError) {
        return err;
      }
      const data = err.originalError.data;
      const message = err.message || 'An error occurred.';
      const code = err.originalError.code || 500;
      return { message: message, status: code, data: data };
    }
  })
);
```
- then extract useful error information in "resolvers.js" with example code below
```
module.exports = {
  createUser: async function({ userInput }, req) {

    ...

    const errors = [];
    if (!validator.isEmail(userInput.email)) {
      errors.push({ message: 'E-Mail is invalid.' });
    }
    if (
      validator.isEmpty(userInput.password) ||
      !validator.isLength(userInput.password, { min: 5 })
    ) {
      errors.push({ message: 'Password too short!' });
    }
    if (errors.length > 0) {
      const error = new Error('Invalid input.');
      error.data = errors;
      error.code = 422;
      throw error;
    }

    ...

  }
};     
```


## Connecting with Frontend
- refer to front end project "02_graphql_node_js_frontend"
- example code implementation in the front end. Key code to take note is `query`, `JSON.stringify(graphqlQuery)`, `resData.errors[0].status`
```
// frontend project folder /app.js
    event.preventDefault();
    const graphqlQuery = {
      query: `
        query UserLogin($email: String!, $password: String!) {
          login(email: $email, password: $password) {
            token
            userId
          }
        }
      `,
      variables: {
        email: authData.email,
        password: authData.password
      }
    };
    this.setState({ authLoading: true });
    fetch('http://localhost:8080/graphql', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(graphqlQuery)
    })
      .then(res => {
        return res.json();
      })
```
- there is an error in the front end where the OPTIONS method is Not Allowed. OPTION are sent before the GET, POST, PATCH, PUT, DELETE requests. Express graphql automatically declines anything that is not POST or GET request . To fix this, make `req.method` for `OPTIONS` equal 200.
```
// backend project folder /app.js
app.use((req, res, next) => {
  
  ...

  if (req.method === 'OPTIONS') {
    return res.sendStatus(200);
  }
  
  next();
});

```


