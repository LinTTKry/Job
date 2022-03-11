# webpack

1. Intial: `npm init -y`  ---->package.json
2. download package: `npm i -D webpack webpack-cli webpack-dev-server typescript ts-loader clean-webpack-plugin`&#x20;
   *
     * webpack-cli
   * webpack command
     * webpack-dev-server
   * webpack develop server
     * typescript
   * ts complier
     * ts-loader
   * ts loader，used to complie ts file in webpack
     * html-webpack-plugin
   * automatically generate html file
     * clean-webpack-plugin
   * Every build will clear the directory first
3. Create webpack configuration file **`webpack.config.js`** in the root directory

```javascript
       const path = require("path");
       const HtmlWebpackPlugin = require("html-webpack-plugin");
       const { CleanWebpackPlugin } = require("clean-webpack-plugin");

       module.exports = {
           optimization:{
               minimize: false // Turn off code compression, optional
           },

           entry: "./src/index.ts",

           devtool: "inline-source-map",

           devServer: {
               contentBase: './dist'
           },

           output: {
               path: path.resolve(__dirname, "dist"),
               filename: "bundle.js",
               environment: {
                   arrowFunction: false 
                   // Turn off the arrow function of webpack to be compatible IE, optional
               }
           },
           // set the template module
           resolve: {
               extensions: [".ts", ".js"]
           },

           module: {
               rules: [
                   {
                       test: /\.ts$/,
                       use: {
                          loader: "ts-loader"     
                       },
                       exclude: /node_modules/
                   }
               ]
           },

           plugins: [
               new CleanWebpackPlugin(),
               new HtmlWebpackPlugin({
                  template:"./src/index.html"
               }),
           ]

       }
```

4\. Create**`tsconfig.json`** in the root directory, the configuration can be according to your needs

````javascript
       {
           "compilerOptions": {
               "target": "ES2015",
               "module": "ES2015",
               "strict": true
           }
       }
       ```
````

5\. Modify **`package.json`** to add the following configuration

```javascript
       {
         "scripts": {
           "test": "echo \"Error: no test specified\" && exit 1",
           "build": "webpack",
           "start": "webpack serve --open chrome.exe"
         }
       }
```

6\. Create a ts file under src, and execute `npm run build` in the parallel command line to compile the code, or execute `npm start` to start the development server

