In this article, I propose a fast way to set up Firebase authentication with Google Single Sign-On (SSO) in your Angular project. I also include how to make authenticated requests to your backend of choice.

```
@startuml
Frontend --> Firebase: User Authentication (SSO)
Firebase --> Frontend: Provide IdToken

Frontend --> Backend: Send Request with IdToken
Backend --> Firebase: Verify Token Request
Firebase --> Backend: Token Validation Response
Backend --> Frontend: Processed Response

@enduml
```

## Firebase setup

Go to Firebase and create a new project.

### App setup

In the project overview, add a web application by clicking the Web icon or the `+Add app` icon
![[firebase_add_app.jpg]]
In the window that opens, give your app a name, for ex `myangularapp` then register the app. Save the firebase config somewhere. It should look like this:

```js
const firebaseConfig = {
  apiKey: "<firebase-api-key>",
  authDomain: "<my-project-id>.firebaseapp.com",
  projectId: "<my-project-id>",
  storageBucket: "<my-project-id>.appspot.com",
  messagingSenderId: "<my-messaging-sender-id",
  appId: "<my-app-id>",
};
```

Note that you can access the Firebase config of your app anytime by going to `General` tab of the project settings (cogs icon) and scrolling down.

### Authentication Setup

Go to `All products` and select `Authentication`.
In `Sign-in method`, click `Add new provider` and select Google. As Firebase is tightly integrated with Google, you actually don't need any extra setup.

And that's it, you are ready to use Google SSO to authenticate your Firebase users.

Note: You can also add other providers (such as `Email/Password`, see image below) but this article will focus on Google SSO.
![[firebase_authentication_providers.png]]

## Angular setup

#### Project creation

First, install angular globally with `npm install -g @angular/cli`. I am using version `17.0.1`

Then, create a new folder : `mydemo` and run `ng new frontend --no-standalone --routing ssr=false` in your folder. This will create the frontend folder with the necessary files.
After this step, repository structure should look like this:

```
demofirebase
└── frontend
```

We just need one extra package. In your `frontend` folder, run `npm i @angular/fire`

Next, generate environments for dev / prod: `ng g environments`. It will generate 2 files in `src/environments` folder: `environment.development.ts` and `environment.ts`.
When developing locally, `environment.ts` is replaced by `environment.development.ts` which means you can import from `environment.ts` and it will automatically use the variables in `environment.development.ts`. This can be seen in the file `angular.json`:

```json
"fileReplacements": [
	{
		"replace": "src/environments/environment.ts",
		"with": "src/environments/environment.development.ts"
	}
]
```

Paste your Firebase config in `environment.development.ts`.

```ts
// environment.development.ts
export const environment = {
  production: false,

  // supposing you have a backend where to send some requests
  backendUrl: "http://127.0.0.1:8000",

  // The firebase config you retrieved from the console.
  // Note that this is NOT sensitive information
  firebaseConfig: {
    apiKey: "<firebase-api-key>",
    authDomain: "<my-project-id>.firebaseapp.com",
    projectId: "<my-project-id>",
    storageBucket: "<my-project-id>.appspot.com",
    messagingSenderId: "<my-messaging-sender-id",
    appId: "<my-app-id>",
  },
};
```

Note: for production (`environment.ts` file), you will just need to replace the backendUrl with your deployed backend and the firebaseConfig with your production config. As a firebase project does not allow you to have multiple environments, it is better to create one project per environment.
For now, you can just use dummy values so that your IDE does not complain about missing fields.

```ts
// environment.ts
export const environment = {
  production: true,
  backendUrl: "backendUrl",
  firebaseConfig: {},
};
```

Finally in your `app.module.ts`, add the necessary imports and initialize the application:

```ts
//app.module.ts
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { AppRoutingModule } from "./app-routing.module";
import { AppComponent } from "./app.component";
import { AngularFireModule } from "@angular/fire/compat";
import { AngularFireAuthModule } from "@angular/fire/compat/auth";
import { environment } from "../environments/environment";

@NgModule({
  declarations: [AppComponent],
  imports: [
    AngularFireAuthModule,
    AngularFireModule.initializeApp(environment.firebaseConfig),
    BrowserModule,
    AppRoutingModule,
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

#### Create necessary component and routes

First, create a component for google sso: `ng g c signin` and a component for a landing page: `ng g c landing`

Then, modify `app-routing.module.ts`:

```ts
import { NgModule } from "@angular/core";
import { RouterModule, Routes } from "@angular/router";
import { SigninComponent } from "./signin/signin.component";
import { LandingComponent } from "./landing/landing.component";

const routes: Routes = [
  { path: "", component: LandingComponent },
  { path: "signin", component: SigninComponent },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

Finally, modify `app.component`

```ts
// app.component.ts
<nav>
  <ul>
    <li>
      <a routerLink="/">Landing page</a>
    </li>
    <li>
      <a routerLink="/signin">Signin page</a>
    </li>
  </ul>
</nav>
<router-outlet></router-outlet>
```

Now if you launch `ng serve` and go to `localhost:4200`, you should see this very beautiful page and you can navigate between landing page and signin page.
![[angular-landing-initial-setup.jpg]]

#### Adding google SSO

To add google SSO, you just need to add a click event listener on a button. I actually prefer to use a directive so that it is reusable and this is the implementation I show below.
Create a directive with`ng g d google-sso` to create the directive and automatically add it to `app.module.ts`

```ts
//google-sso.directive.ts
import { Directive, HostListener } from "@angular/core";
import { AngularFireAuth } from "@angular/fire/compat/auth";
import { GoogleAuthProvider } from "@firebase/auth";

@Directive({
  selector: "[googleSso]",
})
export class GoogleSsoDirective {
  constructor(private angularFireAuth: AngularFireAuth) {}

  @HostListener("click")
  async onClick() {
    const creds = await this.angularFireAuth.signInWithPopup(
      new GoogleAuthProvider(),
    );
    // do what you want with the credentials, for ex adding them to firestore...
  }
}
```

Add the directive to `app.module.ts`

Now you can use the directive in your signin page:

```html
<!-- signin.component.html -->
<button googleSso>Sign in with google</button>
```

And that's it.
To check that it works, you can try to connect and then go to Firebase. In `Authentication`, go to the `Users` tab and you should appear there

Before moving on, let's modify the sign in component to allow for logging out.

```ts
// signin.component.ts
import { Component } from "@angular/core";
import { AngularFireAuth } from "@angular/fire/compat/auth";

@Component({
  selector: "app-signin",
  templateUrl: "./signin.component.html",
  styleUrl: "./signin.component.scss",
})
export class SigninComponent {
  constructor(public angularFireAuth: AngularFireAuth) {}

  logOut() {
    this.angularFireAuth.signOut();
  }
}
```

```html
<!-- signin.component.html -->
@if (angularFireAuth.authState | async) {
<button (click)="logOut()">Log out</button>
} @else {
<button googleSso>Sign in with google</button>
}
```

#### Adding a route available only for logged in users

First, let's create another component: `ng g c require-auth`

Then le'ts create a guard: `ng g g auth` then select `CanActivate`

```ts
// auth.guard.ts
import { CanActivateFn } from "@angular/router";
import { inject } from "@angular/core";
import { AngularFireAuth } from "@angular/fire/compat/auth";

export const authGuard: CanActivateFn = async (route, state) => {
  const angularFireAuth = inject(AngularFireAuth);
  const user = await angularFireAuth.currentUser;
  // coerce to boolean
  const isLoggedIn = !!user;
  return isLoggedIn;
};
```

We then modify the routes in `app-routing.module.ts`

```ts
// app-routing.module.ts - new lines
...
import { RequireAuthComponent } from './require-auth/require-auth.component';
import { authGuard } from './auth.guard';

const routes: Routes = [
  ...
  {
    path: 'require-auth',
    component: RequireAuthComponent,
    canActivate: [authGuard],
  },
];
...
```

Add it in `app.component.html`:

```html
<li>
  <a routerLink="/require-auth">Auth protected</a>
</li>
```

Now you have a route that you can only access when you are logged in

#### Interceptor to log in to backend routes

If you have a backend and want to authenticate your requests, you need to add the token provided by Firebase to the Authorization header.
`ng g interceptor bearer-token`

Note:

```ts
// bearer-token.interceptor.ts
import {
  HttpEvent,
  HttpHandlerFn,
  HttpInterceptorFn,
  HttpRequest,
} from "@angular/common/http";
import { inject } from "@angular/core";
import { AngularFireAuth } from "@angular/fire/compat/auth";
import { from, lastValueFrom, Observable } from "rxjs";
import { environment } from "../environments/environment";

// needs to add this function because getting the token is async
const addBearerToken = async (
  req: HttpRequest<any>,
  next: HttpHandlerFn,
): Promise<HttpEvent<any>> => {
  const angularFireAuth = inject(AngularFireAuth);
  const firebaseUser = await angularFireAuth.currentUser;
  const token = await firebaseUser?.getIdToken();
  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` },
    });
  }
  return lastValueFrom(next(req));
};

export const bearerTokenInterceptor: HttpInterceptorFn = (req, next) => {
  // only add the bearer token to requests to the backend
  // Note that you can customize it to only add the bearer token to certain requests
  if (req.url.startsWith(environment.backendUrl)) {
    return from(addBearerToken(req, next));
  } else {
    return next(req);
  }
};
```

Then in `app.module.ts`, we must add the provider

```ts
// app.module.ts

...
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { bearerTokenInterceptor } from './bearer-token.interceptor';

...

  providers: [provideHttpClient(withInterceptors([bearerTokenInterceptor]))],
...
```

Finally to test it, let's create a service to call the backend: `ng g s api`

```ts
import { HttpClient } from "@angular/common/http";
import { Injectable } from "@angular/core";
import { environment } from "../environments/environment";
import { Observable } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class ApiService {
  // backend url that returns the firebase user id of the current user
  userIdUrl = `${environment.backendUrl}/userid`;

  constructor(private http: HttpClient) {}

  getUserId(): Observable<any> {
    return this.http.get(this.userIdUrl);
  }
}
```

And then in our landing component:

```html
<button (click)="getUserId()">Get user id</button>
```

```ts
import { Component } from "@angular/core";
import { ApiService } from "../api.service";

@Component({
  selector: "app-landing",
  templateUrl: "./landing.component.html",
  styleUrl: "./landing.component.scss",
})
export class LandingComponent {
  constructor(private apiService: ApiService) {}
  getUserId() {
    this.apiService.getUserId().subscribe({
      next: (res) => console.log(res),
      error: (err) => console.log(err),
    });
  }
}
```

You now have a button that you can click to send a request to your backend. If you go to the Network tab in the dev tools, you will see that the token is added to the Authorization header if you are authenticated.


#### Bonus: Where does firebase store signin information?

When we used our interceptor, we used the following code: 
```typescript
const angularFireAuth = inject(AngularFireAuth);
const firebaseUser = await angularFireAuth.currentUser;
const token = await firebaseUser?.getIdToken();
```

But how does it work really? Where is the information stored?

In fact, when you authenticated with Google SSO, authentication information was saved on your browser. 
Open the browser dev tools, go to the storage tab (Application in Chrome) and then go to Indexed db
![[firebase_local_storage.png]]

Object details: 
```json
{
	"fbase_key": "firebase:authUser:<API_KEY>",
	"value": {
	"apiKey": "<API_KEY>",
	"appName": "<APP_NAME>",
	"createdAt": "<CREATED_AT>",
	"displayName": "<DISPLAY_NAME>",
	"email": "<EMAIL>",
	...
	"stsTokenManager":{
		"accessToken": "<b64_ACCESS_TOKEN>",
		"refreshToken": "<b64_REFRESH_TOKEN>",
		"expirationTime": <EXPIRATION_TIME>
		}
	}
}
```

Getting the firebaseUser consists in retrieving it from the local storage. Then the getIdToken call will either directly return the access token, or, if it is expired, refresh it and get a new one by calling firebase

