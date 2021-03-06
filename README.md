# IonicFBAuthentication

Ionic PWA Application

## Run: Ionic Blank app in a local dev server

- Go to our newly created project
- Run ionic serve within the app directory to see our app (this will serve our app in localhost on port **8100**)
  Once installed, we will test how a blank Ionic app scores in our PWA checklist. To perform this audits we are going to use Google suggested tool, **Lighthouse**.
  So we are going to install and use the Lighthouse command line utility. Open your terminal and Install **lighthouse**

<sudo npm install -g lighthouse>

This command will install lighthouse which allow us to perform the audits for an Ionic app. It’s really simple to use, just run...
lighthouse URL-TO-TEST --view
( Note that we run Lighthouse with the --view parameter to immediately open the HTML report in your browser )
Perform the audits with Lighthouse by running:
lighthouse http://localhost:8100/home --view

<hr/>

For that, open your terminal, navigate to the root of your project and type:

<ng add @angular/pwa>

Here is what that command does:

It generates icons in different sizes, and those will be used for the add to home screen icon of your app, it's the default angular badge, so change them for your brand.
It creates a manifest.webmanifest file and links it in your main index.html file, and adds it to the build process in angular.json. (We'll explain that file below).
It installs the @angular/service-worker package.
It imports the ServiceWorkerModule in your app.module.ts file and enables it when you run production builds. (NOTE: It also imports the environment.ts file, so if you already had it there for something you'll get a double import, make sure you remove it.)
Adds the ngsw-config.json file which is the configuration for our service worker.

If you try to test this right now with ng serve your app will work as if you didn't do any of this, this is because service workers are only enabled on production builds.

So if you want to test your PWA locally, you might want to install the node http-server package:

npm install -g http-server
Then, you'd want to run a production build, and let the http-server serve you that build:

ng build --prod
http-server -p 8080 -c-1 path_to_build_folder/
Where path_to_build_folder/ is the production build your app just created, it's usually www or dist/something/.

That one will launch a server listening on port 8080, so going to localhost:8080 will take you to your app.

## Icons

The command generates icons for all needed sizes, but it's the angular default badge, you'll find them under src/assets/icons.

They'll all have their names like icon-72x72.png where the numbers are the size in pixels, and all of them will be linked in your manifest already.

Here you should take a moment to replace them with your own. There are several tools online that create them all based on a high-resolution version of it, also, there are packages that can handle it.

## Web Manifest File

By default, the CLI created a file called manifest.webmanifest in the src folder.

If you open it, you'll see something similar to this:

{
"name": "app",
"short_name": "app",
"theme_color": "#1976d2",
"background_color": "#fafafa",
"display": "standalone",
"scope": "/",
"start_url": "/",
"icons": [
{
"src": "assets/icons/icon-ABxAB.png",
"sizes": "ABxAB",
"type": "image/png"
}
]
}

Let's go through those properties and see what they do:

1.  name: This is the name of your application.
2.  short_name: It's the short name of your app, this is what you see under the app's icon on the home screen.
3.  theme_color: From what I've seen this is handled differently in different devices/OS/browsers, for example, on Android's task switcher, the theme color surrounds the site.
4.  background_color: This is a placeholder background color while your styles load, it's also used as a bg color for the Splash Screen.
5.  display: Controls how are users see the app, it has several options:
    fullscreen takes all of the available space.
    standalone is similar to fullscreen. It won't have navigation or URL bar, but it will show the status bar.
    minimal-ui Similar to standalone but the browser might choose to have UI for navigation.
    browser is a regular browser window.
6.  scope: Defines the navigation scope of the web app context. It restricts to which pages can the manifest be applied. For a full PWA we generally go with / but let's say a subset of your site is the PWA, you can do something like scope: '/app/' and the manifest will be applied only to the app subdirectory.
7.  start_url: This is our start URL, what will be loaded once we click on the home screen button.
8.  icons: These are the links to the icons we generated.

NOTE:
By the way, this file is auto-linked in the index.html file and added into the angular.json build process to make sure our build process includes it.

## Service Worker

Our application already did three things regarding service workers:

It installed the @angular/service-worker package.
Imported the service worker on our module file.
Generated a configuration file for our SW, since it will be autogenerated on build time based on that configuration.
The configuration file is located at the root of our project, and it's called ngsw-config.json, open it, you'll find something like this:

{
"$schema": "./node_modules/@angular/service-worker/config/schema.json",
"index": "/index.html",
"assetGroups": [
{
"name": "app",
"installMode": "prefetch",
"resources": {
"files": ["/favicon.ico", "/index.html", "/*.css", "/*.js"]
}
},
{
"name": "assets",
"installMode": "lazy",
"updateMode": "prefetch",
"resources": {
"files": [
"/assets/**",
"/*.(eot|svg|cur|jpg|png|webp|gif|otf|ttf|woff|woff2|ani)"
]
}
}
]
}

## Good Practices

Right now, you have your PWA ready, and it will work with all the defaults.

The first one is the update. You need to make sure your users are always on the most updated version of the app. That's not complicated since it's a web app; on every refresh, your users will have the most updated version.

What you can do is create a listener that listens for updates in the background and lets your app know when it needs to refresh.

For that, we'll use angular SW package. Go into your app.component.ts file and first, import the package and inject it into the constructor:

import { Component } from '@angular/core';
import { SwUpdate } from '@angular/service-worker';

@Component({
selector: 'app-root',
templateUrl: 'app.component.html',
styleUrls: ['app.component.scss']
})
export class AppComponent {
constructor(private swUpdate: SwUpdate) {}
}

Now, let's create an initialization function that checks if there's an update available and subscribes to it:

import { Component } from '@angular/core';
import { SwUpdate } from '@angular/service-worker';

@Component({
selector: 'app-root',
templateUrl: 'app.component.html',
styleUrls: ['app.component.scss']
})
export class AppComponent {
constructor(private swUpdate: SwUpdate) {
this.initializeApp();
}

initializeApp(): void {
if (this.swUpdate.available) {
this.swUpdate.available.subscribe(() => {
if (confirm('A new version is available. Load it?'))
window.location.reload();
});
}
}
}

We're checking if there's an update available, and subscribing to it when the update is available, we'll trigger an alert to reload the page.

## Installation

Right now, your app will prompt the "Add to home screen" banner. This is automatic, and it's handled by the browser.

If you want to leave it at that, fine, it's your choice, and it works, but consider that not everyone likes to be prompted to install to the home screen as soon as they load your page.

You can actually intercept the browser and stop it from displaying that prompt, and then present it yourself at a later time, maybe after the user has spent some time navigating through your site 😃

The browser triggers an event called beforeinstallprompt, you can add a listener to it like this:

window.addEventListener('beforeinstallprompt', e => {});

Now we need to prevent the default behavior, in other words, we need to stop it from showing the automatic prompt.

window.addEventListener('beforeinstallprompt', e => {
e.preventDefault();
});
