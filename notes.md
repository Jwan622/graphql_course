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



# module 3
- you modify data with mutations
- resolvers are no different, but the schema type is different.. it's a mutation.

this is the mutation:
```text
type Mutation {
    createJob(title: String, description: String, companyId: ID): ID
}
```
the above only returns an ID but it should probably return more info for the client.
to return more info, you can do:

```text
type Mutation {
    createJob(title: String, description: String, companyId: ID): Job
}
```
this is the resolver:

```text
const Mutation = {
  createJob: (root, {companyId, title, description}) => {
    const id = db.jobs.create({companyId, title, description})
    
    return db.jobs.get(id);
  }
}
```

the above returns a Job object with multiple fields.

when quereying from a client, you can write this:

```text
mutation {
  createJob(
    companyId: "SJV0-wdOM111111111111",
    title: "some new job",
    description:"jeff wan desc"
  ) {
    id
    title
    company {
      id
      name
    }
  }
}
```

and you get back:

```text
{
  "data": {
    "createJob": {
      "id": "H1O2Kf-g_",
      "title": "some new job",
      "company": null
    }
  }
}
```

and thet resulting data:

```text
{
  "data": {
    "createJob": {
      "id": "Bynf5GWgu",
      "title": "some new job",
      "company": null
    }
  }
}
```

you can also alias so that the top key is changed:

```text
mutation {
  job: createJob(
    companyId: "SJV0-wdOM111111111111",
    title: "some new job",
    description:"jeff wan desc"
  ) {
    id
    title
    company {
      id
      name
    }
  }
}
```

responseE:

```text
{
  "data": {
    "job": {
      "id": "S19Qqf-l_",
      "title": "some new job",
      "company": null
    }
  }
}
```

- by nesting queries you can get th ejob, and hte company id and name. So instead of making 1 request we can make 2. This is the Job schema and as you can see, if anything returns a Job, you can ask for the relate company as well:

```text
type Job {
    id: ID! # string, not human readable
    title: String
    company: Company # you can associate using a custom type
    description: String
}
```
in the above query for the mutation... you can see it returns the job title and company id and name so we can just consolidate this to one request instead of two. That's great!
- quick note, the types are output types, they describe what can be returned from the server. You cannot type inputs or variables as types unfortuntaely.But you can define `input` types by simply using `input` So you can do this:

```text
type Mutation {
    createJob(input: createJobInput): Job
}
```
and the input type:

```text
type Mutation {
    createJob(input: createJobInput): Job
}

```

so our query can now look like this:

```text

```

the consequence: a single argument for each mutation instead of several.

a key thing about resolves is this if the schema says a query returns a Job, this tells graphql how to resolve a field on the Job, in this case company:

```text

const Job = {
  company: (job) => {
    return db.companies.get(job.companyId)
  }
}
```

and the Job schema does indeed involve a company:

```text
type Job {
    id: ID! # string, not human readable
    title: String
    company: Company # you can associate using a custom type
    description: String
}
```