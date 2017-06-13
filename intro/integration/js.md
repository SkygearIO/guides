---
title: Integrate Skygear with different JS frameworks
---

[[toc]]

Skygear JS SDK can be used with various JS frameworks smoothly to boost your development.

In this guide, we will guide you through the set up of Skygear with

- Angular
- Ionic
- React Native

If you are looking for a quick way to get started, simply check out our demo projects:
- [Angular-Skygear demo project](https://github.com/skygear-demo/skygear-angular)
- [Ionic-Skygear demo project](https://github.com/skygear-demo/skygear-ionic).


## Angular and Ionic

### Setting up the build process

First, install the Skygear JS SDK.

```
npm install --save skygear
```

### Configuring Skygear

Our next step is to configure Skygear in your web client so that it communicates with the server.

To do so, make sure you have already created an account at [Skygear.io](https://skygear.io). We will need the "app end point" and the "API key" of your Skygear app for configuration.

A project using Angular (including Ionic) should follow Angular's pattern of managing data. Skygear is usually used as a global singleton object, the Angular way to manage such data is through [Service](https://angular.io/docs/ts/latest/tutorial/toh-pt4.html).

First, you need to create a `SkygearService` that hold an reference of the singleton Skygear object. An empty service looks like this. Now, save this as `src/app/skygear.service.ts`.

``` javascript
import { Injectable } from '@angular/core';
import skygear from 'skygear';

@Injectable()
export class SkygearService {
}
```

Then you have to add a method to the service for configuring and returning the Skygear instance.

``` javascript
import { Injectable } from '@angular/core';
import skygear from 'skygear';

@Injectable()
export class SkygearService {
  // Configure and init Skygear JS Client
  isConfigured = false;
  getSkygear() {
    if (this.isConfigured) {
      // Return the instance directly if it is ready
      return Promise.resolve(skygear);
    }
    let promise = skygear.config({
      'endPoint': 'https://<your-app-name>.skygeario.com/', // trailing slash is required
      'apiKey': '<your-api-key>',
    });
    promise.then(()=> this.isConfigured = true);
    return promise;
  }
}
```

Now you have a service that is working, the next step would be using the service in the component. Here I would use the root module `app.module.ts` and component `app.component.ts` as example, but the idea works for any other modules.

To use the service in the component, you have to pass it as a provider of the module, edit `src/app/app.module.ts`.

``` javascript
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
  providers: [SkygearService],  // Include Skygear Service
  bootstrap: [AppComponent]
})
export class AppModule {
}

```

Next, accepts the service in the component by modifying the constructor. This
creates a private property `skygearService` in the component and assign the
service to it.

And you're also recommended to do any preparation in the `ngOnInit` life cycle
function.

So we will add the following in the `AppComponent` class
(`src/app/app.component.ts`).

``` javascript
export class AppComponent implements OnInit {
  title = 'Loading...';
  skygear = null;

  constructor(private skygearService: SkygearService) {}

  ngOnInit(): void {
    this.skygearService.getSkygear()
    .then(skygear=> {
      this.skygear = skygear;
      this.title = "Configured";
    })
    // Skygear requires a user before creating a record
    .then(()=> this.skygear.auth.signupAnonymously())
    .then(user=> {
      this.title = "Signed up anonymous user: " + user.id;
    })
    .catch(error=> {
      this.title = "Cannot configure skygear";
    });
  }
}
```

For demonstration purpose, we can display the `title` variable in the template
of `src/app/app.component.html`

``` html
<h1>{{title}}</h1>
```

### Creating your first Skygear record with user interaction

Let's create our first Skygear record to see if we have successfully set up the app.

First, you need a button, add the following code to your template file, for the default `AppModule`, it will be `app.component.html`.

```
<button (click)="addNewRecord()">Add new record</button>
```

Then, you have to add a handler that will be triggered when the button is pressed, it looks like this:

```
export class AppComponent implements OnInit{
  // Other codes

  addNewRecord() {
    // Called when button is clicked
    this.skygearService.getSkygear()
    // Skygear requires a user before creating a record
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
  }
}
```

That is it. Start building your awesome app now. :smile:

## React Native

### Setting up the build process

First, install the Skygear JS SDK.

```
npm install --save skygear
```

Next, import it in your code.

As Skygear has special integration with React Native to leverage the extended capability of the platform, instead of just importing the normal Skygear, you should do this

```
import skygear from 'skygear/react-native'
```

### Configuring Skygear

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
