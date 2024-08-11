## 🙌 Welcome

This cheat sheet is intended for newcomers to the Relay GraphQL compiler, or anyone who needs a quick reference or simplified version of the Relay docs!

## 📁 PDF Download

TODO: A concise, single-page PDF download format of this information is coming soon.

## 🧠 Prior Knowledge

This page assumes a previous understanding of GraphQL concepts. If you are new to GraphQL terminology (queries, mutations, fragments, etc.), or just need a refresher, it is recommended you start [here](https://graphql.org/learn/).

## 💡 Example

All concepts will be illustrated by example, and they assume a server running very simple GraphQL schema. This code can be found in the `server/` directory.


### I. Basics

---

##### `graphql`

A "string literal" for a graphql operation (query, mutation, subscription).

**Syntax**: ``const var = graphql`<operation>` ``;

**Example**: 

```
const NewsfeedQuery = graphql`
  query NewsfeedQuery {
    topStory {
      title
      summary
      poster {
        name
        profilePicture {
          url
        }
      }
      thumbnail {
        url
      }
    }
  }
`;
```

---

### II. Querying

##### `useLazyLoadQuery`

Executes a GraphQL query simultaneously while rendering a component. The simplest Relay API for fetching data through a query.

**Syntax**: 
`const data = useLazyLoadQuery(<graphql literal>, <variables>)`;

Intuitively, for a React component to fetch data from a server using a query, it needs to know 1) which query to execute (specified by the graphql literal) and 2) values of variables to pass.

**Example**: `const data = useLazyLoadQuery(NewsfeedQuery, {});`

---

##### `useQueryLoader` & `usePreloadedQuery`


`useQueryLoader` executes a GraphQL query. Child components access the results of this query using `usePreloadedQuery`. Avoids executing a separate network request (query) with each new component.

**Syntax (Parent Component)**: 
```
const [queryReference, fetch] = useQueryLoader(<graphql literal>);

...

fetch(<options>); // actually sends gql query - usually inside useEffect without dependencies

...

{/* Inside component JSX */}
{queryReference && (
  <ChildComponent queryReference={queryReference} />
})

```

**Syntax (Child Component)**

```
const ChildComponent = ({ queryReference, ... }) => {

  ...

  const queryData = usePreloadedQuery(<graphql literal>, <query reference>);

  ...

  return (
    <div>Component JSX derived from queryData!</div>
  );
}

...


```

The arguments to `useQueryLoader` are intuitive. To fetch query data, we need to know 1) which type of query to execute (indicated by gql literal) and 2) what options to execute with, which we can pass in at call time.

Likewise, the arguments to `usePreloadedQuery` are simpler than they seem. We have already executed a query, so again we need to know 1) which type of query we executed (again indicated by gql literal) and 2) a pointer to the actual results of the query (indicated by `queryReference`).

---

### III. Mutations

`useMutation`

Executes a GraphQL mutation (an operation on the server which changes data in some way).

**Syntax:** `const [commitMutation, loading] = useMutation(<graphql literal>);`

**Example**:

```
const StoryLikeButtonLikeMutation = graphql`
  mutation StoryLikeButtonLikeMutation(
    $id: ID!,
    $doesLike: Boolean!,
  ) {
    likeStory(
      id: $id,
      doesLike: $doesLike
    ) {
      story {
        id
        likeCount
        doesViewerLike
      }
    }
  }
`;

... 

const SomeComponent = () => {

  ...
  const [commitMutation, loading] = useMutation(StoryLikeButtonLikeMutation);
  ...

  return (
    <button>
      {loading ? "Loading Mutation Results" : "Press Me!"}
    </button>
  )
}
```