# GraphQL with ExpressJS


## About
- REST API: Stateless, client-independent API for exchanging data.
- GraphQL API: Stateless, client-independent API for exchanging data with **higher query flexibility**
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
- This project is associated to another react front end project [02_graphql_node_js_frontend](https://github.com/chickensmitten/02_graphql_node_js_frontend)

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
- in "schema.js" there are three types `input`, `types` and `schema`.
- `schema` normally contains `RootQuery` to GET data, while `RootMutation` is to manipulate (PATCH, PUT, POST, DELETE) data.
- `input` is used as inputs. see `input UserInputData { ... }` in `createUser(userInput: UserInputData): User!`
- `type` is used as outputs. see `type PostData { ... }` in `posts(page: Int): PostData!`
```
    type PostData {
        posts: [Post!]!
        totalPosts: Int!
    }

    input UserInputData {
        email: String!
        name: String!
        password: String!
    }

    type RootQuery {
        login(email: String!, password: String!): AuthData!
        posts(page: Int): PostData!
        post(id: ID!): Post!
        user: User!
    }

    type RootMutation {
        createUser(userInput: UserInputData): User!
        createPost(postInput: PostInputData): Post!
        updatePost(id: ID!, postInput: PostInputData): Post!
        deletePost(id: ID!): Boolean
        updateStatus(status: String!): User!
    }

    schema {
        query: RootQuery
        mutation: RootMutation
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
- there is an error in the front end where the OPTIONS method is Not Allowed. OPTION are sent before the GET, POST, PATCH, PUT, DELETE requests. Express graphql automatically declines anything that is not POST or GET request. To fix this, make `req.method` for `OPTIONS` equal 200.
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


## Authentication
- To enable user authentication, in schema.js, define a query that requires login details
- Firstly, schema must be able to handle user login. Example code implementation
```
    type RootQuery {
        login(email: String!, password: String!): AuthData!
        posts(page: Int): PostData!
        post(id: ID!): Post!
        user: User!
    }
```
- Then, define the user token from front end. See JWT in 01_api_node_js.
```
    type AuthData {
        token: String!
        userId: String!
    }
```
- Then in resolvers, create a method/function to handle this. The methods called in resolvers are similar to JWT setup in 01_api_node_js.
```
login: async function({ email, password }) { ... },
```
- In front end, create a login handler to send login request and save the JWT response.
```
loginHandler = (event, authData) => { ... };
```
- Before creating a post, check if user is authenticated then extract user data from JWT with graphql. First make changes to auth in middleware
```
const auth = require('./middleware/auth');
app.use(auth);
```
then in resolvers
```
createPost: async function({ postInput }, req) {
  
  ...

  if (!req.isAuth) {
    const error = new Error('Not authenticated!');
    error.code = 401;
    throw error;
  }

  ...

  const user = await User.findById(req.userId);
  if (!user) {
    const error = new Error('Invalid user.');
    error.code = 401;
    throw error;
  }

  ...  

},
```
- creating a post in front end
```
  let graphqlQuery = {
    query: `
    mutation CreateNewPost($title: String!, $content: String!, $imageUrl: String!) {
      createPost(postInput: {title: $title, content: $content, imageUrl: $imageUrl}) {
        _id
        title
        content
        imageUrl
        creator {
          name
        }
        createdAt
      }
    }
  `,
    variables: {
      title: postData.title,
      content: postData.content,
      imageUrl: imageUrl
    }
  };

  ...

  return fetch('http://localhost:8080/graphql', {
    method: 'POST',
    body: JSON.stringify(graphqlQuery),
    headers: {
      Authorization: 'Bearer ' + this.props.token,
      'Content-Type': 'application/json'
    }
  });
```


## Pagination
- to get pagination, add an argument as `page: Int` in `post` in schema. 
```
// /graphql/schema.js
    type RootQuery {
        ...
        posts(page: Int): PostData!
        ...
    }
```
- in resolver, get the `page` property, then use `page`. `page` is extracted with deconstructor.
```
posts: async function({ page }, req) { 

  ...

  if (!page) {
    page = 1;
  }

  ...

},
```
- in front end, fetch posts with page pagination
```
    const graphqlQuery = {
      query: `
        query FetchPosts($page: Int) {
          posts(page: $page) {
            posts {
              _id
              title
              content
              imageUrl
              creator {
                name
              }
              createdAt
            }
            totalPosts
          }
        }
      `,
      variables: {
        page: page
      }
    };
    fetch('http://localhost:8080/graphql', {
      method: 'POST',
      headers: {
        Authorization: 'Bearer ' + this.props.token,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(graphqlQuery)
    })
```


## Image Upload
- graphql only works with JSON data. Doesn't work with images. So to upload images, have to use back classic endpoint
```
// /app.js 
app.put('/post-image', (req, res, next) => {
  if (!req.isAuth) {
    throw new Error('Not authenticated!');
  }
  if (!req.file) {
    return res.status(200).json({ message: 'No file provided!' });
  }
  if (req.body.oldPath) {
    clearImage(req.body.oldPath);
  }
  return res
    .status(201)
    .json({ message: 'File stored.', filePath: req.file.path });
});
```
- then in front end, upload the image to cloud storage with classic API endpoint. Because in this example, we are storing the image in local storage, once stored, get the image file path and send it along with the graphql query.
```
finishEditHandler = postData => {
  
  ...

  formData.append('image', postData.image);
  if (this.state.editPost) {
    formData.append('oldPath', this.state.editPost.imagePath);
  }
  fetch('http://localhost:8080/post-image', {
    method: 'PUT',
    headers: {
      Authorization: 'Bearer ' + this.props.token
    },
    body: formData
  })
    .then(res => res.json())
    .then(fileResData => {
      const imageUrl = fileResData.filePath || 'undefined';
      let graphqlQuery = {
        query: `
        mutation CreateNewPost($title: String!, $content: String!, $imageUrl: String!) {
          createPost(postInput: {title: $title, content: $content, imageUrl: $imageUrl}) {
            _id
            title
            content
            imageUrl
            creator {
              name
            }
            createdAt
          }
        }
      `,
        variables: {
          title: postData.title,
          content: postData.content,
          imageUrl: imageUrl
        }
      };

  ...

}
```

## Getting Single Post
- add `id: ID!` to `post` in `RootQuery` schema
```
    type RootQuery {
        login(email: String!, password: String!): AuthData!
        posts(page: Int): PostData!
        post(id: ID!): Post!
        user: User!
    }
```
- add `post: async function({ id }, req) { ... },` to resolver. `id` is extracted using a descontructor. then add logic to get post from the database and return results.
- Then in front end, 
```
// /src/pages/Feed/SinglePost/SinglePost.js
    const postId = this.props.match.params.postId;
    const graphqlQuery = {
      query: `query FetchSinglePost($postId: ID!) {
          post(id: $postId) {
            title
            content
            imageUrl
            creator {
              name
            }
            createdAt
          }
        }
      `,
      variables: {
        postId: postId
      }
    };
    fetch('http://localhost:8080/graphql', {
      method: 'POST',
      headers: {
        Authorization: 'Bearer ' + this.props.token,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(graphqlQuery)
    })
      .then(res => {
        return res.json();
      })
      .then(resData => {
        if (resData.errors) {
          throw new Error('Fetching post failed!');
        }
        this.setState({
          title: resData.data.post.title,
          author: resData.data.post.creator.name,
          image: 'http://localhost:8080/' + resData.data.post.imageUrl,
          date: new Date(resData.data.post.createdAt).toLocaleDateString('en-US'),
          content: resData.data.post.content
        });
      })
      .catch(err => {
        console.log(err);
      });
```

## Updating/Editing Single Post
- add `id: ID!, postInput: PostInputData` to `updatePost` in `RootMutation` schema
```

    type RootMutation {
        ...
        updatePost(id: ID!, postInput: PostInputData): Post!
        ...
    }
```
- in resolvers, `updatePost: async function({ id, postInput }, req) { ... }`. `id` and `postInput` is extracted using a descontructor. then add logic to get post from the database and return results.
- then in front end
```
// front end code /src/pages/feed/feed.js
  ...
        if (this.state.editPost) {
          graphqlQuery = {
            query: `
              mutation UpdateExistingPost($postId: ID!, $title: String!, $content: String!, $imageUrl: String!) {
                updatePost(id: $postId, postInput: {title: $title, content: $content, imageUrl: $imageUrl}) {
                  _id
                  title
                  content
                  imageUrl
                  creator {
                    name
                  }
                  createdAt
                }
              }
            `,
            variables: {
              postId: this.state.editPost._id,
              title: postData.title,
              content: postData.content,
              imageUrl: imageUrl
            }
          };
        }

        return fetch('http://localhost:8080/graphql', {
          method: 'POST',
          body: JSON.stringify(graphqlQuery),
          headers: {
            Authorization: 'Bearer ' + this.props.token,
            'Content-Type': 'application/json'
          }
        });
  ...
```

## Deleting Single Post
- create a schema in `RootMutation`
- create `deletePost` in resolver with logic and return if success. In this example, have to remove images in local storage, remove the user dependency because a post is associated to a user. Wrap the functions in try catch in case of error.
- in front end, find the handler `deletePostHandler` then handler the delete function.


## Using Variables
- Using variables in graphQL. String interpolation with `${String}` is not a recommended way to add variables in graphql queries.
- This is done by declaring variables in the front end. 
- For example, add `FetchPosts` and the `$page: Int` variables to be used. Then `$page` can be called in the query below. `, variables: { page: page }` is added to enable this.
```
// front end code /src/pages/feed/feed.js
    const graphqlQuery = {
      query: `
        query FetchPosts($page: Int, $variable2: String, $variable3: ID) {
          posts(page: $page) {
            posts {
              _id
              title
              content
              imageUrl
              creator {
                name
              }
              createdAt
            }
            totalPosts
          }
        }
      `,
      variables: {
        page: page,
        variable2: some.variable.2,
        variable3: some.variable.3
      }
    };
```
- Take note that the types declared in the front end have to match the types in backend schema types.
- `FetchPosts` can be any name.
- `query` before `query FetchPosts` and `posts` that comes after it, should match backend's schema's `posts` in `RootQuery` and resolver's `posts`.