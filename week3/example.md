# Example project: Animals
In this exercise you'll create a simple GraphQL API with Express, Mongoose and Apollo Server based on the previous week's labs.

1. Continue from [setup](./setup.md)


2. Run the app and open localhost:3000/graphql


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
1. From previous lessons you shoud already have a MongoDB running on your localhost / cloud.
2. Use `sssf-labs-graphql1` as database name. (Always change the database name for assignments/labs beause you don't
   want to mess up the apps you've made earlier.)
3. Setup db connection with .env and mongoose like in previous labs
4. Add a mutation for adding categories to `api/schemas/category.graphql`:
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
5. Test mutation:
   ```graphql
   mutation {
     addCategory(category_name:"Mammal"){
       id
       category_name
     }
   }
   ```
6. Use addCategory as an example and create mutations to add species and animals
7. Test:
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
8. Change types Animal, Species, Category in resolver(s) to use database instead of the hard coded arrays.
9. Add mutation modifyAnimal to edit animal's name and species. Example query:
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
10. Complete the app by adding queries and mutations
     - get all animals, species, categories
     - get one animal, species, category
     - add animal, species, category
     - modify animal, species, category
     - delete animal, species, category

## Uploading with GraphQL
[Best practices](https://www.apollographql.com/blog/backend/file-uploads/file-upload-best-practices/)
