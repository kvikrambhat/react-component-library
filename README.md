# Build a React Component Library 

## 1. Setting up the component library project
In a directory of your choice, create a simple react app:
```sh
npx create-react-app hello-component
```
## 2 Create a HelloWorld Component
- Create a src/components folder and create a HelloWorld.js file inside it.(Optional: delete the public folder and all the other files inside the src folder)
**Note any parameter or call back to be sent to the component from the consumper app can be specific as params**
    ```javascript
    import React from 'react';
    
    function HelloWorld() {
        return <h1>Hello World</h1>
    }
    export default HelloWorld;
    ```
    Example of a componet with input params
    ```javascript
    export default function SignIn({ authContext, authUrl, redirectUrl }) {
    ```
- Create an index.js file inside the src folder and export the component we just created. index.js file will be our key file throughout this process, exporting all the components we will create inside the components folder.
    ```javascript
    export { default as HelloWorld } from './components/HelloWorld';
    ```

## 3.Configure Rollup.js & Babel to Build Our Library
 Now, to build our library, we need to install Rollup.js, Babel and other packages we require to bundle our library.
 - We will be installing the following packages:
    ```sh
    $ npm i -D @babel/cli @babel/core @babel/preset-env @babel/preset-react rollup @rollup/plugin-babel rollup-plugin-delete rollup-plugin-peer-deps-external npm-run-all
    ```
- Next, create a .babelrc file in the root folder. It should look like this:
    ```
    {
    "presets": ["@babel/env", "@babel/preset-react"]
    }
    ```
- Next, create a rollup.config.js file in the root folder. It should look like this:
    ```javascript
    import babel from '@rollup/plugin-babel';
    import external from 'rollup-plugin-peer-deps-external';
    import del from 'rollup-plugin-delete';
    import pkg from './package.json';
    
    export default {
        input: pkg.source,
        output: [
            { file: pkg.main, format: 'cjs' },
            { file: pkg.module, format: 'esm' }
        ],
        plugins: [
            external(),
            babel({
                exclude: 'node_modules/**',
                babelHelpers: 'bundled' 
            }),
            del({ targets: ['dist/*'] }),
        ],
        external: Object.keys(pkg.peerDependencies || {}),
    };
    ```
    This is what each configuration field stands for:
    
    - input: The entry point to the component(s) we want to bundle. We directly point this to our src/index.js which we have used to export our components.
    - output: This specifies the directory where you want to save the bundled library. We are importing the output paths from package.json, which we will specify as the ./dist folder.
    - plugins: This specifies all the plugins you wish to use and their respective configurations. For instance, external is asking rollup to exclude peerDependencies as part of the bundle as they will be imported separately by the app calling this package. We will also configure peerDependencies in the next step.
    
- Add source, main & module values in package.json for our rollup.config.js
    ```
      "name": "private-component-library",
      "version": "0.1.0",
      "private": true,
      "main": "dist/index.cjs.js",
      "module": "dist/index.esm.js",
      "source": "src/index.js",
      ...
    ```
- Add react, react-dom to peerDependencies and remove them from dependencies 
 **Note: Any package you install to be used by the component library needs to be moved to the peerDependencies from dependencies.This will ensure that the package is isntalled when the componet is used in a consumper applicaiton**
    ```
        "peerDependencies": {
            "react": "^18.2.0",
            "react-dom": "^18.2.0"
          }
    ```
- Update the scripts section in the package.json file
    ```
    "scripts": {
      "build": "rollup -c",
      "build-watch": "rollup -c -w", 
      "start-playground": "cd playground && npm run start",
      "i-all": "npm i && cd playground && npm i",
      "dev": "npm-run-all --parallel build-watch start-playground"
    },
    ```
 ## 4. Build our component library
 Run the following command to build the library
 ```sh
    $ npm update
    $ npm run build
```
You should have a ./dist folder with two bundled files in it, index.cs.js and index.esm.js

Next we’ll publish our component libary to a private Github NPM Registry. This will allow the library to be consumable for all of our apps.

## 5. Create a Personal Access Token

```
Github Account -> Settings -> Developer Settings -> Personal Access Tokens -> Generate New Token

Note: Private NPM Packages
Select scopes: write:packages, read:packages
```

- With the token copied, in the same directory as your package.json file, create an .npmrc file and add the following line, replacing TOKEN with your personal access token.With the token copied, in the same directory as your package.json file, create an .npmrc file and add the following line, replacing TOKEN with your personal access token.
    ```
    //npm.pkg.github.com/:_authToken=<TOKEN>
    Note: remove the "<" and ">" used in the placeholder above
    ```
- Create a Private repository in Github, just like you would normally and copy the URL, the same repo can be used to house the componet library code base
- To successfully publish, you need to make these changes to your package.json.
    * "name" : to the repo owner where the package will be published "@owner/repo-name" 
    * "files" : The folder with our generated library, i.e. the ./dist folder.
    * "publishConfig" : The registry where you want the package published
    * "repository" : URL of the private Github repository we just created.
    * if you have the following line in package.json, remove it. **"private": true**
    
    ```
    "name": "@kvikrambhat/react-component-library",
      "files": [
        "./dist"
      ],
      "publishConfig": {
        "registry": "https://npm.pkg.github.com"
      },
      "repository": "git://github.com/kvikrambhat/react-component-library",
    ```
- Publish the library using the following command
    ```
    $ npm publish
    ```
    the package will be available at the URL "https://github.com/<owner>/<repo-name>/packages"
    
## 6. Consuming private component library
- Create .npmrc at the root level in your consumer app and adding the following lines.
    * TOKEN : Can be the same as the previous one used to publish or a new one wiht only read access
    * OWNER : github id of the package owner
    ```
    //npm.pkg.github.com/:_authToken=<TOKEN>
    registry=https://npm.pkg.github.com/<OWNER>
    
    Note: remove the "<" and ">" used in the placeholder above
    ```
- Install the component package using the follwing command
    ```
    $ npm install @OWNER/repo-name
    ```
    The componet can then be imported and used as usual
    
    ```javascript
    import { HelloWorld } from '@kvikrambhat/react-component-library';
    
    function App() {
      return (
         <HelloWorld />
      );
    }
    
    export default App;
    ```

