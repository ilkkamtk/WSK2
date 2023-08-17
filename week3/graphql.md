# [GraphQL](https://graphql.org/)
<!-- TODO: show also how to use postman for graphql queries and explain why it's good practice to hide the playground in production (so /graphql only with http POST) -->
## Study before the classes
1. [GraphQL](https://www.youtube.com/watch?v=bRnu7xvU1_Y)
2. [Apollo server tutorial](https://www.youtube.com/watch?v=zDdNDWEAu78)
3. [apollo-server-express](https://github.com/apollographql/apollo-server/tree/main/packages/apollo-server-express)

### Why GraphQL
Compare
* [Restful API](https://restcountries.com/v3.1/all)
* [GraphQL API](https://studio.apollographql.com/public/countries/explorer?explorerURLState=N4IgJg9gxgrgtgUwHYBcQC4QEcYIE4CeABAMIQyp4CWCAzkcADpJFFTmU31MutFIBDRMz5EEcCACsqIvu1RUkyFA1mjBw3qwC%2Bs3Um0htQA&variant=current) 

### Some GraphQL libraries
- [Apollo server](https://www.apollographql.com/docs/apollo-server/)
- [TypeGraphQL](https://typegraphql.com/)

## [SDL](https://www.prisma.io/blog/graphql-sdl-schema-definition-language-6755bcb9ce51)
"GraphQL has its own language to write GraphQL Schemas: The GraphQL Schema Definition Language (SDL). SDL is simple and intuitive to use while being extremely powerful and expressive."

### Queries and Mutations
- **GraphQL** is an open-source data query and manipulation language for APIs, and a runtime for fulfilling queries with existing data. [1]
- **GraphQL Queries** are used to fetch data from a GraphQL server. [1]
- A **query** is a request for data composed of one or more fields, and is structured in such a way that the server can return exactly what the query asks for. [1]
- The GraphQL query language is strongly typed, meaning that the server can validate a query before executing it. [1]
- The syntax for writing GraphQL queries is simple and easy to learn. [1]
- GraphQL queries can contain fields, arguments, aliases, fragments, and more. [2]
- A **field** is a unit of data you can request from the server, such as a person's name or a post's title. [2]
- An **argument** is an optional parameter that can be passed in a query, such as a person's age or a post's ID. [2]
- An **alias** is a way to rename the result of a field or fragment in a query. [2]
- A **fragment** is a reusable set of fields that can be included in a query. [2]
- GraphQL queries can be used to fetch data from multiple resources in a single request. [3]
- GraphQL queries can be optimized to minimize the amount of data transferred from the server to the client. [3]

[1]: https://graphql.org/learn/queries/
[2]: https://graphql.org/learn/queries/#fields-arguments-aliases-fragments
[3]: https://graphql.org/learn/queries/#fetching-multiple-resources

## Schemas and Types
- **GraphQL Schemas** define the structure of data in a GraphQL API. [1]
- The schema is a contract between the server and the client that defines the data that can be queried, as well as the type of data that can be returned. [1]
- A GraphQL schema is composed of a set of **types**, each of which defines a set of fields that can be queried. [1]
- The **root types** of a schema are the Query and Mutation types, which define the entry points for the API. [1]
- The **scalar types** of a schema define the primitive types of data that can be queried and returned, such as strings, integers, and booleans. [2]
- The **object types** of a schema define the data structures that can be queried and returned, such as users, posts, and comments. [2]
- The **input types** of a schema define the data structures that can be passed to the server as arguments in a query or mutation. [2]
- The **enum types** of a schema define the sets of valid values that can be used in a query or mutation. [2]
- The **interface types** of a schema define the common fields that can be shared across different object types. [2]
- The **union types** of a schema define the sets of object types that can be returned from a query. [2]

[1]: https://graphql.org/learn/schema/
[2]: https://graphql.org/learn/schema/#scalar-types-objects-input-types-enums-interfaces-unions

### Example project: Animals
In this exercise you'll create a simple GraphQL API with Express, Mongoose and Apollo Server based on the previous week's labs.

1. From previous lessons you shoud already have a MongoDB running on your localhost / cloud.
1. Use `sssf-labs-graphql1` as database name. (Always change the database name for assignments/labs beause you don't want to mess up the apps you've made earlier.)
2. (Use Studio 3T (or similar) to add test data. Add a category, a species and an animal just like in [previous lab](../week2/nosql-mongodb-mongoose.md#mongodb).)
1. Download the [starter files](https://github.com/ilkkamtk/SSSF_ts_graphql-starter).
2. Create first schema as `api/schemas/animal.graphql`
   ```graphql
   scalar DateTime

   type Animal {
     id: ID
     animal_name: String
     species: Int
   }
   
   type Query {
     animals: [Animal]
   }
   ```
1. Create first resolver as `api/resolvers/animalResolver.ts`
   ```typescript
   import {Animal} from '../../interfaces/Animal';
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
2. Run the app and open localhost:3000/graphql
1. Test with this query:
   ```graphql
   {
     animals {
       id
       animal_name
       species
     }
   }
   ```
1. The result should be: 
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

#### Add "Relations"
1. Modify the app so that you get species name instead of the id. For that you need to add a new schema 'species.graphql' where you define new type 'Species' (similar to Animal) and then modify the type of 'species' in animalType to be Species type instead of ID. It also needs a resolver to return one item form 'speciesData' array. Use `array.find` to match species.id to parent.id.
   * [Here is an example](https://www.apollographql.com/docs/apollo-server/data/resolvers/#example) (scroll a bit down and locate Expand example)
1. Do the same with categoryData so that you get category name instead of category id.
1. Example query:
   ```graphql
   {
     animals {
       id
       animal_name
       species {
         species_name
         category {
           category_name
         }
       }
     }
   }
   ```
1. Example result: 
   ```json
   {
     "data": {
       "animals": [
         {
           "id": "1",
           "animal_name": "Frank",
           "species": {
             "species_name": "Cat",
             "category": {
               "category_name": "Mammal"
             }
           }
         }
       ]
     }
   }
   ```
   
### Switch to database
1. Setup db connection with .env and mongoose like in previous labs
1. Add a mutation for adding categories to `api/schemas/category.graphql`:
   ```graphql
   type Category {
      id: ID
      category_name: String
   }
   
   extend type Mutation {
     addCategory(category_name: String): Category
   }
   ```
   ```javascript
   // resolvers/categoryResolver.ts
   import CategoryModel from '../models/categoryModel';
   import Category from '../../interfaces/Category';

   export default {
      Mutation: {
         addCategory: (_parent: undefined, args: Category) => {
            console.log(args);
            const newCategory = new CategoryModel(args);
            return newCategory.save();
         },
      },
   };
   ```
1. Test mutation:
   ```graphql
   mutation {
     addCategory(category_name:"Mammal"){
       id
       category_name
     }
   }
   ```
1. Use addCategory as an example and create mutations to add species and animals
1. Test: 
   ```graphql
   mutation {
     addSpecies(species_name:"Cat", category:"someCategoryId"){
       id
       species_name
     }
   }
   
   mutation {
     addAnimal(animal_name: "Frank", species: "someSpeciesId"){
       id
       animal_name
     }
   }
   ``` 
1. Change types Animal, Species, Category in resolver(s) to use database instead of the hard coded arrays.
1. Add mutation modifyAnimal to edit animal's name and species. Example query:
   ```graphql
   mutation {
     modifyAnimal(
       id: "someAnimalId",
       animal_name:"New Name",
       species: "someSpeciesId"
     ) {
       animal_name
       species {
         species_name
       }
     }
   }
   ```
1. Complete the app by adding queries and mutations
   - get all animals, species, categories
   - get one animal, species, category
   - add animal, species, category
   - modify animal, species, category
   - delete animal, species, category

## Uploading with GraphQL
[Best practices](https://www.apollographql.com/blog/backend/file-uploads/file-upload-best-practices/)
