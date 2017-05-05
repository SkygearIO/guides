# Using Skygear JS SDK with other frameworks

Skygear JS SDK can be used with various frameworks smoothly to boost your development. First, install Skygear JS SDK.

```
npm install --save skygear
```

Next, import it into your code.

```
var skygear = require('skygear');
// Or ES6
import skygear from 'skygear';
```

## Using with React Native

Skygear has special integration with React Native to leverage the extended capability of the platform. Instead of just importing the normal skygear, you should do this

```
import skygear from 'skygear/react-native'
```


# Using older version

For versions before 0.24, some manual works are needed for integrating frameworks building with webpack, popular examples are Ionic and Angular.

## Webpack

Webpack does not work well with Skygear out of the box, if you are building with Webpack, please modify your Webpack config, and add `react-native` to `externals`.

```
  externals: [
    'react-native'
  ]
```

## Ionic

If you are using `ionic start` to create an Ionic project, you have to customize its internal Webpack config to get it working with Skygear.

1. update `package.json`, specify your own Webpack config.
```
  "config": {
    "ionic_webpack": "./webpack.config.json"
  }
```

2. Create the webpack config file
```
  {
    "externals": ["react-native"],
    "output": {
      "libraryTarget": "umd",
      "path": "{{BUILD}}",
      "publicPath": "build/",
      "filename": "main.js"
    }
  }
```

## Angular

If you are using `ng new` to bootstrap your app, you will find it uses Webpack internally to build the project. Unfortunately, there is no way to customize the build process and therefore not possible to make it work with Skygear. We recommend using [Angular-webpack](https://github.com/preboot/angular-webpack) for bootstrapping, and then follow the above guide for integrating with Webpack.
