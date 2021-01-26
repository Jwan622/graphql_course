# module 1
- grpah ql solves under and overfetching of resources. underfetching is when in REST, you have to make multiple calls to the server if you need multiple resources.
- graph ql benefit: you describe what's possible with a type system. the schema describe the API.
- tools: GraphiQL, running graphql queries in browser.
- graphql is used over HTTP often, most common.
- apollo server is an easy way to setup a graphql server in javascript. data can be from any sources. open source. apollo server serves over http.

-graphql schema describes what the api can do.
-graphql written  in graphql schema definition language.
- in a schema you always need aquery type, something like this:

```text
const typeDefs = `
  type Query {
    greeting: String
  }
`
```
 contains all possible queries client can make to server. in the above, we can only ask server to give us a greeting.
- gql can parse the schema.  it's from apollo server module. gql is a tag function, it's a function for graphql code to convert string intto an AST for graphql code.
- so teh above is the interface for our API. Now we need an implementation to specify how the server returns a greeting. resolvers do this, just an object.
- resolvers need to match structure of type definitions. Whenever graphql calls for a field, the function for field is called from resolver with matching field. The logic in the resolve can be anything. 
- resolver needs to mirror the type def exactly structure.

- Apollo Server gives you graphql playground at the port. To setup apollo server, it looks like this:

```text
const server = new ApolloServer({typeDefs, resolvers})
server.listen({port: 9000})
  .then(({url}) => console.log(`Server running at ${url}`))
```

you can make queries and graphql playground also gives docs.

![playground](photos/module_1/playground.png)

the response:

```text
{
  "data": {
    "greeting": "Hello Graphql world"
  }
}
```

notice the top level response key is `data`
- graphql playground is essentially a client. 
- graphql always is a post request.
- request is in json from client. and the query payload is a string, newlines esecaped in json string.
- response is a json object from apollo server.
- since graphql clients just makes requests over http, we can call graphql server from any language as long as it provides http requests.
- if a function is an async function, it returns a promise and so you need to resolve it using `.then`


- graphql can play well with an express server. we can use `apollo-server-express` package.
- the type defs specifies what the resolver returns. If the typeDef schema specifies that ta resolver returns an Int, the resolver cannot return "hello world!". The typedef tells us what the client can ask of the graphql server.

# module 2
- you can remove a field and the graphql server handles this. helps prevent client from overfetching. Client can specify fields and the graphql engine takes care of this for us.

```text
{
  jobs {
    id
    title
    description
  }
}
```

is equiv to

```text
query {
  jobs {
    id
    title
    description
  }
}
```

- you don't need to specify query at the beginning. this is a nested query, query is teh default root object.

- this is how to make associations:

```text
type Company {
    id: ID!
    name: String
    description: String
}

type Job {
    id: ID! # string, not human readable
    title: String
    company: Company # you can associate using a custom type
    description: String
}
```
- each resolver has arguments, the first argument is the parent object. we can use this to resolve associations. if you're finding the association, the parent will be the object that stores the foreign key.
- the server will resolve the object request and if it comes across an association, it will resolve that too.
  
if you want to query a single job, it needs arguments. write this in typedefs:

```text
type Query {
     ...
    job(id: ID!): Job
}
```

you would then query the server like this, using double quotes for args:

```text
{
  job(id: "rJKAbDd_z") {
    id
    title
  }
}
```

- second argument to resolver is args from the client.