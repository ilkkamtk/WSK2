# Apollo Server Setup

## Apollo Server

- [Apollo Server](https://www.apollographql.com/docs/apollo-server/) is an open-source GraphQL server that is compatible
  with any GraphQL client.
- It's built with Node.js and can be used with any database or data source including third-party REST APIs.
- These instructions apply to version 4.10.0.

## Step 1 - Project Boilerplate

1. Clone or download as ZIP the starter project from [GitHub](https://github.com/ilkkamtk/SSSF_ts_graphql-starter).
2. Rename the project folder to `wsk2-graphql-server` (or something similar).
3. Enter the project folder and run `npm i` to install the dependencies.
4. (Create a new repository in GitHub and push the project to it.)
5. Run `npm run dev` to start the server. Open `localhost:3000` in your browser to see if the server is running.
    - Run also `npm run test` to see if the tests are passing.
6. Open the project in VSCode.

## Step 2 - Install [Apollo Server with Express Middleware](https://www.apollographql.com/docs/apollo-server/api/express-middleware) and some [GraphQL tools](https://the-guild.dev/graphql/tools/docs/schema-merging#file-loading)

1. Install Apollo Server and its dependencies:
    - `npm i @apollo/server graphql`
    - `npm i @graphql-tools/load-files @graphql-tools/merge`

2. Create first schema as `api/schemas/animal.graphql`
   ```graphql
   
   type Animal {
     id: ID
     animal_name: String
     species: Int
   }
   
   type Query {
     animals: [Animal]
   }
   ```
3. Create first resolver as `api/resolvers/animalResolver.ts`
   ```typescript
   import {Animal} from '../../types/DBTypes';
   const animalData = [
      {
         id: '1',
         animal_name: 'Frank',
         species: '1',
      },
   ];
   
   const speciesData = [
      {
         id: '1',
         species_name: 'Cat',
         category: '1',
      },
   ];
   
   const categoryData = [
      {
         id: '1',
         category_name: 'Mammal',
      },
   ];
   
   export default {
      Query: {
         animals: (_parent: undefined, args: Animal) => {
          return animalData;
         },
      },
   };
   ```

4. Uncomment every line in `src/api/schemas/index.ts`.
5. Add the following to `src/app.ts`:
    ```typescript
    ...
    import {ApolloServer} from '@apollo/server';
    import {expressMiddleware} from '@apollo/server/express4';
    import typeDefs from './api/schemas/index';
    import resolvers from './api/resolvers/index';
    ...
    // remove app.use(cors());
    // after app.get('/', (req....
    const server = new ApolloServer({
      typeDefs,
      resolvers,
    });

    await server.start();

    app.use('/graphql', cors(), express.json(), expressMiddleware(server));
    ...
    ```
6. Test the server with Postman. Create a new request; type: POST, url: `localhost:3000/graphql`. 
   - Test with this query:
   ```graphql
   {
     animals {
       id
       animal_name
       species
     }
   }
   ```
   - The result should be:
   ```json
   {
     "data": {
       "animals": [
         {
           "id": "1",
           "animal_name": "Frank",
           "species": "1"
         }
       ]
     }
   }
   ```



## Step 3 - Add Sandbox for easier testing

1. Add the following import to `src/app.ts`:
    ```typescript
    import { 
       ApolloServerPluginLandingPageLocalDefault,
       ApolloServerPluginLandingPageProductionDefault,
    } from '@apollo/server/plugin/landingPage/default';
    ```

2. Modify the `server` variable in `src/app.ts`:
    ```typescript
    const server = new ApolloServer({
      typeDefs,
      resolvers,
      plugins: [
        ApolloServerPluginLandingPageLocalDefault(),
      ],
    });
    ```

3. Open `http://localhost:3000/graphql` in your browser. You should see the GraphQL Sandbox.
   - Try the following query in the sandbox:
      ```graphql
      {
           animals {
             id
             animal_name
             species
           }
         }
      ```
4. On production, you can use the following configuration:
    ```typescript
    const server = new ApolloServer({
      typeDefs,
      resolvers,
      plugins: [
        ApolloServerPluginLandingPageProductionDefault(),
      ],
    });
    ```
5. You can also use ternary operator to use different plugins based on the environment:
    ```typescript
    const server = new ApolloServer({
      typeDefs,
      resolvers,
      plugins: [
        process.env.NODE_ENV === 'production'
          ? ApolloServerPluginLandingPageProductionDefault()
          : ApolloServerPluginLandingPageLocalDefault(),
      ],
    });
    ```

6. Commit and push your changes to GitHub.
7. Continue with [the next steps](example.md).
