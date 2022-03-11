# Compile Options

## Automatic compilation

* When compiling a file, after using the -w command, the TS compiler will **automatically monitor the changes** of the file, and recompile the file when the file changes
* Example：
  * ```
    tsc xxx.ts -w
    ```

### Automatically compile ts file

* If you use the tsc command directly, you can automatically compile all the ts files under the current project into js files
* But if you can **directly use the tsc** **command**, you must **first create a ts configuration file tsconfig.json** in the project root directory.
* **tsconfig.json is a JSON file**. **After adding the configuration file, you only need the tsc command** to complete the compilation of the entire project

### Config options：

* **include**
  * Define the directory where the files you want to be compiled are located
  * Default value：\["\*\*/\*"]
  * Example：
    * ```javascript
      // * represents file 
      // ** represents directly
      "include":["src/**/*", "tests/**/*"]
      ```
    * In the above example, all files in the src directory and tests directory will be compiled
* **exclude**
  * In the above example, all files in the src directory and tests directory will be compiled
  * Default value：\["node\_modules", "bower\_components", "jspm\_packages"]
  * Example：
    * ```javascript
      "exclude": ["./src/hello/**/*"]
      ```
    * In the above example, the files in the hello directory under src will not be compiled
* **extends**
  * Define the inherited configuration file
  * Example：
    * ```javascript
      "extends": "./configs/base"
      ```
    * In the above example, the current configuration file will automatically include all the configuration information in base.json in the config directory
*   **files**

    * Specify the list of files to be compiled, only used when there are few files to be compiled
    * Example：
      * ```javascript
        "files": [
            "core.ts",
            "sys.ts",
            "types.ts",
            "scanner.ts",
            "parser.ts",
            "utilities.ts",
            "binder.ts",
            "checker.ts",
            "tsc.ts"
          ]
        ```
      * The files in the list will be compiled by the TS compiler


* **compilerOptions**
* In the compilerOptions contains multiple sub-options to complete the configuration of the compilation
  * sub-options
    * **target**
      * version
      * optional value：
        * ES3（default）、ES5、ES6/ES2015、ES7/ES2016、ES2017、ES2018、ES2019、ES2020、ESNext
      * Exam：
        * ```javascript
          "compilerOptions": {
              "target": "ES6"
          }
          ```
    * lib---normally use default value
      * Specify the library (host environment) included when the code is running
      * optional value:
        * ES5、ES6/ES2015、ES7/ES2016、ES2017、ES2018、ES2019、ES2020、ESNext、DOM、WebWorker、ScriptHost ......
      * Exam：
        * ```javascript
          "compilerOptions": {
              "target": "ES6",
              "lib": ["ES6", "DOM"],
              "outDir": "dist",
              "outFile": "dist/aa.js"
          }
          ```
    * **module**
      * set the modular system used by the compiled code
      * optional value：
        * CommonJS、UMD、AMD、System、ES2020、ESNext、None
      * Exam：
        * ```typescript
          "compilerOptions": {
              "module": "CommonJS"
          }
          ```
    * **outDir**
      * The directory where the compiled file is located
      * By default, the compiled js file will be located in the same directory as the ts file. After setting outDir, you can change the location of the compiled file
      * Exam：
        * ```javascript
          "compilerOptions": {
              "outDir": "dist"
          }
          ```
        * After setting, the compiled js file will be generated to the dist directory
    * **outFile**
      * **Compile all files into a js file**
      * By default, all the code written in the global scope will be merged into one js file. **If the module specifies `None`, `System` or `AMD`, the modules will be merged into the file**.
      * Exam：
        * ```javascript
          "compilerOptions": {
              "outFile": "dist/app.js"
          }
          ```
    * rootDir
      * Specify the root directory of the code. By default, the directory structure of the compiled file will take the **longest public directory as the root directory.** You can manually **specify the root directory t**hrough rootDir
      * Exam：
        * ```javascript
          "compilerOptions": {
              "rootDir": "./src"
          }
          ```
    * **allowJs**
      * Whether to compile the js file
    * **checkJs**
      * Whether to check the js file
      * Exam：
        * ```javascript
          "compilerOptions": {
              "allowJs": true,
              "checkJs": true
          }
          ```
    * **removeComments**
      * Whether to delete the comment
      * Default：false
    * **noEmit**
      * Does not generated the compiled code
      * Default：false
    * **sourceMap**
      * Whether to generate the sourceMap
      * Default：false：false
* **Strict check**
  * **strict**
    * Enable **all strict checks**, the default value is true
  * **alwaysStrict**
    * Always compile code in **strict mode**
  * **noImplicitAny**
    * Disallow implicit any type
  * **noImplicitThis**
    * Prohibit ambiguous this
    * `function fn(){alert(this)}`
    * `box1?.addListner('click',function(){})`
  * **strictBindCallApply**
    * Strictly check the parameter list of bind, call and apply
  * **strictFunctionTypes**
    * Strictly check the type of the function
  * **strictNullChecks**
    * Strict null value check
  * **strictPropertyInitialization**
    * Strictly check whether the property is initialized
* **Extra check**
  * **noFallthroughCasesInSwitch**
    * Check that the switch statement contains the correct break
  * **noImplicitReturns**
    * Check that the function has no implicit return value
  * **noUnusedLocals**
    * Check for unused local variables
  * **noUnusedParamete**rs
    * Check unused parameters
* **High level**
  * **allowUnreachableCode**
    * Check for unreachable codes
    * optional value：
      * true，ignore
      * false，error
  * **noEmitOnError**
    * Do not generated the compiled file if there is an error
    * Default value: false
