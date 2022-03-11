---
description: >-
  In development, it is often necessary to combine babel to transform the code
  to make it compatible with more browsers
---

# Babel

1. Install package：`npm i -D @babel/core @babel/preset-env babel-loader core-js`

* core-js
  * core-js is used to make older browsers support the new version of ES syntax

&#x20;   2.Modify the webpack.config.js configuration file

* ```javascript
      module: {
          rules: [
              {
                  test: /\.ts$/,
                  use: [
                      {
                          loader: "babel-loader",
                          options:{
                              presets: [
                                  [
                                      "@babel/preset-env",
                                      {
                                      // Compatible to version 58 of chrome
                                          "targets":{
                                              "chrome": "58",
                                              "ie": "11"
                                          },
                                          "corejs":"3",
                                      // The method of using corejs
                                      // usage: load on demand
                                          "useBuiltIns": "usage"
                                      }
                                  ]
                              ]
                          }
                      },
                      {
                          loader: "ts-loader",

                      }
                  ],
                  exclude: /node_modules/
              }
          ]
      }

  ```

