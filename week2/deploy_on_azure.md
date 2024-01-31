# Express and Mongoose on Azure

## Prerequisite

- Azure account for student:
  - visit [azure portal](https://portal.azure.com/)
  - choose 'Sign-in option' -&gt; 'Sign in to an organization' -&gt; Domain name: 'metropolia.fi'
  - use your metropolia username in the form of 'username@metropolia.fi' (NOT your metropolia email)

(In no order)

- Create a github/gitlab/bitbuckets/gnu savanah/framagit/... project
  - with .gitignore for node
  - (readme, licences,...)<br><br>

- Install Azure extensions to VSCode
  - Azure Account
  - Azure App Service
  - (Azure Databases)
  - (Azure Resources)


- Video about deploying TypeScript app: 
[![Watch the video](https://i.ytimg.com/vi/WRmHsc0vSbs/hqdefault.jpg?sqp=-oaymwExCNACELwBSFryq4qpAyMIARUAAIhCGAHwAQH4Af4JgALQBYoCDAgAEAEYfyA8KB4wDw==&rs=AOn4CLCRb2XmrnlT04I3MqTql6sSrh5oJA)](https://www.youtube.com/playlist?list=PLKenVLUxjmH_1obN-sz7KvOcBHbRuTdiO)

The same in text:
- [Clone this example app.](https://github.com/ilkkamtk/azure-example)

## Locally
1. `git clone` your repo on your local computer
2. Create repo on GitHub and connect it to your local repo.
3. Do your normal `npm install`
4. Create a `.env` file at the root of your repo (this is [why](https://12factor.net/config))
   - use `.env.sample` as example:
     - `DATABASE_URL=mongodb://tester:abc123@lurl-to-server:port/test-db`
5. Run dev server locally `npm run dev`
6. Run tests `npm run test`

## Azure
1. Create MongoDB on Azure Cosmos DB
   - Choose serverless option
2. Create Web App in App Services
   - Node 18
   - Select B1 plan to enable CD
   - Connect App to your GitHib repo
3. Add environment variables DATABASE_URL and NODE_ENV in configuration

## Locally
1. Pull changes from remote repo to get .github folder
2. Edit the yml file in .github folder
   - Add the following env setting:
   ```yaml
   - name: npm install, build, and test
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          npm install
          npm run build --if-present
          npm run test --if-present
   - name: Zip artifact for deployment
        run: zip release.zip ./* -r

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v2
        with:
          name: node-app
          path: release.zip
   ```
   - And in the `deploy` section:
   ```yaml
   steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v2
        with:
          name: node-app

      - name: unzip artifact for deployment
        run: unzip release.zip
   ```

## Remote
1. `git commit -a -m` your local changes
2. `git push` your local changes to your remote repo
3. Go to 'Actions' of your repo to see the deployment details in GitHub

---

- JS version: follow the steps from the [doc](https://docs.microsoft.com/en-us/azure/app-service/tutorial-nodejs-mongodb-app#1---create-the-azure-app-service)

