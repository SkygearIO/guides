# Using Skygear JS SDK with other frameworks

Skygear JS SDK can be used with various frameworks smoothly to boost your development.

# Setting up the build process

First, install Skygear JS SDK.

```
npm install --save skygear
```

Next, import it into your code.

```
var skygear = require('skygear');
// Or ES6
import skygear from 'skygear';
```

For most of the cases, this should be enough to get Skygear JS SDK running. If you are using React Native, Angular or Ionic with an old version, please follow the guides below.

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

# Configurating

You can use the following code snippet to configure the SDK to connect to server.

```
skygear.config({
  'endPoint': 'https://<your-app-name>.skygeario.com/', // trailing slash is required
  'apiKey': '<your-api-key>',
}).then(() => {
  console.log('skygear container is now ready for making API calls.');
}, (error) => {
  console.error(error);
});
```

## Angular and Ionic

A project using Angular (including Ionic) should follow Angular's pattern of managing data. Skygear is usually used as a global singleton object, the Angular way to manage such data is through [Service](https://angular.io/docs/ts/latest/tutorial/toh-pt4.html).

First, you need to create a `SkygearService` that hold an refernce of the singleton skygear object. An empty service looks like this, save this as `skygear.service.ts`.

```
import { Injectable } from '@angular/core';

@Injectable()
export class SkygearService {
}
```

Then you have to add a method to the service for configurating and returing the skygear instance.

```
  isConfigurated = false;
  getSkygear() {
    if (this.isConfigurated) {
      // Return the instance directly if it is ready
      return Promise.resolve(skygear);
    }
    let promise = skygear.config({
      'endPoint': 'https://<your-app-name>.skygeario.com/', // trailing slash is required
      'apiKey': '<your-api-key>',
    });
    promise.then(()=> this.isConfigurated = true);
    return promise;
  }
```

Now you have a service that is working, the next step would be using the service in the component. Here I would use the root module `app.module.ts` and component `app.component.ts` as example, but the idea works for any other modules.

To use the service in the component, you have to pass it as a provider of the module.

```
import { SkygearService } from './skygear.service';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule
  ],
  providers: [SkygearService],  // This line is want you need
  bootstrap: [AppComponent]
})
export class AppModule {
}

```

Next, accepts the service in the component by modifying the constructor. This creates a private property `skygearService` in the component and assign the service to it.

```
constructor(private skygearService: SkygearService) {}
```

Next, it is recommended to do necessary configuration in the `ngOnInit` life cycle funcion.

```
  title = 'Loading...';
  skygear = null;
  ngOnInit(): void {
    this.skygearService.getSkygear()
    .then(skygear=> {
      this.skygear = skygear;
      this.title = "Configurated";
    })
    .catch(error=> {
      this.title = "Cannot configure skygear";
    });
  }
```

Finally, the state can be displayed via the template.

```
<h1>{{title}}</h1>
```

## Creating your first record with user interaction

First, you need a button, add the following code to your template file, for the default `AppModule`, it will be `app.component.html`.

```
<button (click)="addNewRecord()">Add new record</button>
```

Then, you have to add a handler that will be triggered when the button is pressed, it looks like this:

```
export class AppComponent {
  addNewRecord() {
    // Called when button is clicked
  }
}
```

You can then place code to create a record in the handler, and reflect the state to user interface.

```
this.skygearService.getSkygear()
// Skygear requires a user before creating a record
.then(()=> this.skygear.signupAnonymously())
.then(()=> {
  // Create the note
  var Note = this.skygear.Record.extend('Note');
  // And save the note
  return this.skygear.publicDB.save(new Note({
    'content': 'Hello World'
  }));
})
.then((record)=> {
  // Here, record is the saved note,
  // it can be shown to user interface by assigning to a property that is displayed by the template.
  this.title = "Saved record: " + record.id;
});
```

A full example project can be found [here for Angular](https://github.com/skygear-demo/skygear-angular) and [Here for Ionic](https://github.com/skygear-demo/skygear-ionic).