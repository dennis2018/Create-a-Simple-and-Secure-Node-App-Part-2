# Create-a-Simple-and-Secure-Node-App-Part-2
In this tutorial series, you'll learn how to create a secure and simple Node.js application built with the Express web framework. You'll see how Passport.js with Auth0 is used to manage user authentication and protect routes. On the frontend, you'll use Pug templates for rendering views along with CSS for maintaining style sheets.
In the first part, you learned how to create a user interface and an API and how to streamline your Node.js development workflow by using nodemon and browser-sync.

In this second part, you'll secure the web app by adding user authentication to it using Passport.js and Auth0.

# Setting Up the Project
If you are starting from this part of the tutorial, you can get the app that was built in part 1 by following these quick steps.

- Clone the project repository:


```
git clone https://github.com/dennis2018/Create-a-Simple-and-Secure-Node-App

```

- If you haven't connected to GitHub using SSH, please clone the repo with the HTTPS URL:

```
git clone https://github.com/dennis2018/Create-a-Simple-and-Secure-Node-App

```
- Make the project from Part 1 your current directory:

```
cd toody-portal
```
- Install the project dependencies:

```
npm install
```

- Run the app server using Nodemon:

```
npm run dev
```

- In a separate terminal tab or window, serve the app frontend on a static server using Browsersync:

```
npm run ui
```
Browsersync proxies the server running on port 8000 with nodemon.

To see the app in action and start following the tutorial, visit http://localhost:8000 on your browser.

# What You Will Build
In this exercise, you'll continue building a login portal for a restaurant named toody that specializes in making delicious food for developers.
Using server-side rendering (SSR), the web app consists of two views: a public login screen and a protected account information screen.

# Prerequisites
- Basic understanding of Node.js and JavaScript.

- A terminal app for MacOS and Linux or PowerShell for Windows.

- Node.js v8+ and a Node.js package manager installed locally.

To install Node.js and NPM, use any of the official Node.js installers provided for your operating system.

# Adding User Authentication to an Express App
In this section, you are going to learn how to add user authentication to an Express app using Passport.js and Auth0. If you click on the login button present on the index page, you'll see the following error on the screen:

```
Cannot GET /login
```
At this moment, there is no controller to handle the GET /login endpoint in your API. The goal for that endpoint is to manage everything related to authenticating a user, that is, verifying the identity of a user.

Saying that implementing authentication is challenging is not done with an effort to promote gatekeeping or FUD (fear, uncertainty, and doubt). The process of authentication itself is fairly straightforward; however, what is complex is implementing correctly and securely each step of the process and following best practices as suggested by OWASP which includes:
- Implementing proper password strength controls such as password length, complexity, and topology.
- Implementing secure password recovery mechanisms.
- Storing passwords in a secure fashion which includes hashing passwords with a salt.
- Transmitting passwords only over TLS or other strong transport.
- Implementing correctly authentication and error messages that mitigate user ID and password enumeration.
- Preventing brute-force attacks.

These authentication best practices address different attack vectors and mitigate vulnerabilities in the authentication process that could compromise the identity of your users. Technically, your team can implement all of these authentication steps; ideally, you delegate the responsibility of strong authentication to an identity platform such as toody.

## Identity as a service
Auth0 provides authentication as a service. It gives you the building blocks you need to secure your applications without having to become a security expert. You can connect any application to Auth0 and define the identity providers you want to use (how you want your users to log in).

To connect your app with Auth0, you'll use the Node.js SDKs. Then, any time a user tries to authenticate, Auth0 will verify their identity and send the required information back to your app.

## Using middleware for authentication
As explained in the "Using middleware" section of the Express docs, an Express application is essentially a series of middleware function calls that execute during the request-response cycle. Each function can modify the request and response objects as needed and then either pass control to the next middleware function or end the request-response cycle.

When creating protected routes in Express, you need to know if the user is authenticated before executing the logic of route controllers. Thus, authentication in Express can be seen as a step in the request-response cycle which can be implemented as middleware.

To make the implementation of authentication easier, instead of writing all the code needed to structure the authentication middleware, you'll use Passport.js, a simple and unobtrusive authentication middleware for Node.js created by Jared Hanson, former Principal Architect at Auth0.

# Setting up Passport.js with Node and Express
As explained in the Passport.js "Overview" document, authentication takes a variety of forms: users may log in by providing a username and password or single sign-on using an OAuth provider such as Facebook or Twitter.

Passport.js offers different authentication mechanisms, known as strategies, to cater to the unique authentication requirements each application has. Strategies are packaged as individual modules and you can choose which strategies to employ, without creating unnecessary dependencies.

You are going to use the Auth0 authentication strategy with Passport.js so that you don't have to worry about creating and managing user credentials yourself. To start, install the following packages:

```
npm install passport passport-auth0 express-session dotenv --save
```
Here's a breakdown of each package being installed:

- passport: As discussed, Passport.js is Express-compatible authentication middleware for Node.js.

- passport-auth0: This is the Auth0 authentication strategy for Passport.js.

- express-session: This is a module to create and manage session middleware.

- dotenv: This is a zero-dependency module that loads environment variables from a .env file into process.env.

## Configuring express-session
Next, open index.js and import the express-session module and configure it as follows:

```
/ index.js

// Other imports
const expressSession = require("express-session");

// App instance, port declaration

const session = {
  secret: "LoxodontaElephasMammuthusPalaeoloxodonPrimelephas",
  cookie: {},
  resave: false,
  saveUninitialized: false
};

if (app.get("env") === "production") {
  session.cookie.secure = true; // Serve secure cookies, requires HTTPS
}

app.use(expressSession(session));
// Other Express app settings

// Route controllers

// App listening
```

expressSession takes a configuration object, session, that defines what options are enabled in a session. Here, you are configuring the following options:
- secret: This is the secret used to sign the session ID cookie. This can be either a string for a single secret or an array of multiple secrets. You are using the long string LoxodontaElephasMammuthusPalaeoloxodonPrimelephas, which is a mix of words from the scientific classification of the elephant.

- cookie: This is the settings object for the session ID cookie. The default value is { path: '/', httpOnly: true, secure: false, maxAge: null }. You are setting this to be an empty object.

- resave: This option forces the session to be saved back to the session store, even if the session was never modified during the request. For the Auth0 Passport.js strategy, you need this to be false.

- saveUninitialized: This forces a session that is new but not modified, uninitialized, to be saved to the store. Passport.js handles the modification of the session so you can set it to false. As Gabe de Luca points as a downside of savings uninitialized sessions, if set to true, when your web page gets crawled by bots, a session will be created for them in addition to users who only visit your front page but don't login, which uses up more sessions and memory.
Additionally, when your app is running as a production instance, you'll only send cookies over HTTPS:

```
if (app.get("env") === "production") {
  session.cookie.secure = true; // Serve secure cookies, requires HTTPS
}
```
With session.cookie.secure set to true, compliant clients won't send cookies back to the server if the browser doesn't have an HTTPS connection. Thus, an HTTPS-enabled website is required for secure cookies.

With the session middleware configured, you are ready to add and configure Passport.js in your application

## Configuring Passport with the application settings
Auth0 requires a couple of configuration values that let the authentication server identify your application securely and correctly. These values will be stored in a hidden file named .env that you can create under the project directory as follows:

```
# For macOS/Linux use:
touch .env
# For Windows PowerShell use:
ni .env
(gi .\.env).Attributes += 'Hidden'
```

Want to learn more about how to create hidden files and other common development tasks with PowerShell? Check out Windows PowerShell Commands for Web Developers.

You must add this .env hidden file to your .gitignore file to ensure it's not committed to version control.

You will populate this file with the Auth0 configuration values in the next section. In this section, you'll focus on wiring up Passport.js.

Open index.js and then import dotenv and load environment variables from .env on the top of the file as follows:

```
// index.js

require("dotenv").config();

// Rest of the file content
```

Next, below the statement that imports express-session, import passport and passport-auth0:

```
// index.js

// dotenv loading
// Other imports
const passport = require("passport");
const Auth0Strategy = require("passport-auth0");

// Rest of the file content
```

With Auth0Strategy in place, proceed to define this strategy. Between the definition of the session options object and app.use(expressSession(session));, add the following variable definition:

```
// index.js

// dotenv loading
// Imports

// App, port, and session definitions

const strategy = new Auth0Strategy(
  {
    domain: process.env.AUTH0_DOMAIN,
    clientID: process.env.AUTH0_CLIENT_ID,
    clientSecret: process.env.AUTH0_CLIENT_SECRET,
    callbackURL:
      process.env.AUTH0_CALLBACK_URL || "http://localhost:3000/callback"
  },
  function(accessToken, refreshToken, extraParams, profile, done) {
    /**
     * Access tokens are used to authorize users to an API 
     * (resource server)
     * accessToken is the token to call the Auth0 API 
     * or a secured third-party API
     * extraParams.id_token has the JSON Web Token
     * profile has all the information from the user
     */
    return done(null, profile);
  }
);

// Rest of the file content
```
Here, the Auth0Strategy method takes your Auth0 credentials and initializes the strategy. In the next section, you'll create an Auth0 application to get your Auth0 credentials. At this point, it's expected for you to see errors in the console as follows:

```
Error: You must provide the domain configuration value to use passport-auth0.
```

It's important first to understand what the Auth0Strategy is doing for you.

The callback function passed to the Auth0Strategy method is known as the verify callback, which has the purpose of finding the user that possesses a set of credentials. When Passport.js authenticates a request, it parses the credentials or any other authentication information contained in the request. It then invokes the verify callback with the authentication data as arguments, in this case, accessToken, refreshToken, extraParams, and profile.

Since Passport.js is a middleware function, the verify callback also receives done as an argument to pass control to the next middleware function. As Auth0 does all the credential validation for you, the verify callback invokes done to supply Passport with the user that authenticated through the profile object.

Once the strategy is defined, you need to have Passport.js use it. In Express applications, you are also required to initialize Passport and modify the persistent login session using Passport through the app.use() method from Express. To achieve these tasks, update index.js as follows:

```
// index.js

// dotenv loading
// Imports

// App, port, session, and strategy definitions

// Production check for cookie configuration

app.use(expressSession(session));
passport.use(strategy);
app.use(passport.initialize());
app.use(passport.session());

// Other app settings

// Rest of the file content
```

You must ensure that passport.initialize() and passport.session() are added after binding the express-session middleware, expressSession(session), to the application-level middleware.

## Storing and retrieving user data from the session
The last step in setting up Passport.js is to support login sessions by serializing and deserializing user instances to and from the session. Under the binding of passport.session(), add the following code:

```
// index.js

// dotenv loading
// Imports

// App, port, session, and strategy definitions\

// Production check for cookie configuration

app.use(expressSession(session));
passport.use(strategy);
app.use(passport.initialize());
app.use(passport.session());

passport.serializeUser((user, done) => {
  done(null, user);
});

passport.deserializeUser((user, done) => {
  done(null, user);
});

// Rest of the file content
```

It's important to keep the amount of data stored within the session small to ensure good performance and fast user lookup. Only the user object is serialized to the session. When subsequent requests are received by the server, this stored object is used to locate the user and reassign it to req.user.

Now that Passport.js is configured, the next step is to add authentication endpoints to your API to handle user login and logout along with providing an endpoint for the Auth0 authentication server to communicate with your app.

## Creating Express Authentication Endpoints
In this section you are going to create three endpoints that will handle the authentication flow of the application:

- GET /login

- GET /logout

- GET /callback

To manage these endpoints better, you are going to create them within an authentication module and export them through an Express router so that your Express application can use them.

To start, create an auth.js file under the project directory:

```
# For macOS/Linux use:
touch auth.js
# For Windows PowerShell use:
ni auth.js
```

Once the file is created, add the following imports and .env loading to its content:


```
// auth.js

require("dotenv").config();

const express = require("express");
const router = express.Router();
const passport = require("passport");
const util = require("util");
const url = require("url");
const querystring = require("querystring");
```
Here's an overview of the new modules being used:

- router: A router object is an isolated instance of middleware and routes. It is capable only of performing middleware and routing functions. Every Express application has a built-in app router.

- util: This module is designed to support the needs of Node.js internal APIs by providing useful utility functions to perform tasks like formatting and encoding strings.

- url: This module provides utilities for URL resolution and parsing.

- querystring: This module provides utilities for parsing and formatting URL query strings.

You'll soon see how these modules streamline your route controller logic.

The first endpoint you'll create is the GET /login one. Append the following route definition to your auth.js file:

```
// auth.js

// Imports, load .env

router.get(
  "/login",
  passport.authenticate("auth0", {
    scope: "openid email profile"
  }),
  (req, res) => {
    res.redirect("/");
  }
);
```
GET /login performs the user login. This endpoint demonstrates how Express route can take multiple callbacks after the definition of the route path (first argument). Here, you pass two callback functions:

- The passport.authenticate() method which gets the Passport.js strategy and an options object that defines the application scopes as arguments.

- A custom callback called after passport.authenticate() finishes to handle authentication success or failure and issue a response to the client.

After the authentication server identifies and validates the user, it calls the GET /callback endpoint from your API to pass all the required authentication data. Implement that endpoint by appending the following route definition to your auth.js file:

```
// auth.js

// Imports, load .env

// GET /login

router.get("/callback", (req, res, next) => {
  passport.authenticate("auth0", (err, user, info) => {
    if (err) {
      return next(err);
    }
    if (!user) {
      return res.redirect("/login");
    }
    req.logIn(user, (err) => {
      if (err) {
        return next(err);
      }
      const returnTo = req.session.returnTo;
      delete req.session.returnTo;
      res.redirect(returnTo || "/");
    });
  })(req, res, next);
});
```
GET /callback performs the final stage of authentication and redirects to the previously requested URL or / if that URL doesn't exist. Within the custom callback function, you check for the presence of an error, err and if the user object is defined. If there's an error, you pass control to the next middleware function along with the error object. If authentication has failed, user will be false and you redirect the user back to the /login page.

Since the passport.authenticate() method is called within the route controller, it has access to the request, req, and response, res, objects through a closure. If authentication is successful, you call req.LogIn() to establish a login session. req.logIn() is a function exposed by Passport.js on the req object. When the login operation completes, user is assigned to the req object as req.user.

To create a good user experience after the login session is established, you redirect users to the page they were using before the authentication request took place. The route of such page is the value of req.session.returnTo, which you assign to the returnTo variable before deleting it. If returnTo is defined, you redirect the user to that route; otherwise, you take them to a /user page that should, ideally, present the user with their information.

## You can redirect to any other routes.
As you can see, GET /callback acts as the bridge between your application and the Auth0 authentication server. Using Passport.js you are able to streamline the process of creating a login session and an active user in your app.

You are almost done. The last authentication endpoint to implement is GET /logout and it's the most complex one. Append the following route definition to the content of auth.js:
```
// auth.js

// Imports, load .env

// GET /login

// GET /callback

router.get("/logout", (req, res) => {
  req.logOut();

  let returnTo = req.protocol + "://" + req.hostname;
  const port = req.connection.localPort;

  if (port !== undefined && port !== 80 && port !== 443) {
    returnTo =
      process.env.NODE_ENV === "production"
        ? `${returnTo}/`
        : `${returnTo}:${port}/`;
  }

  const logoutURL = new URL(
    util.format("https://%s/logout", process.env.AUTH0_DOMAIN)
  );
  const searchString = querystring.stringify({
    client_id: process.env.AUTH0_CLIENT_ID,
    returnTo: returnTo
  });
  logoutURL.search = searchString;

  res.redirect(logoutURL);
});
```
This route performs a session logout and redirects the user to the homepage.

Passport.js also exposes a logOut() function on the request object, req, that can be called from any route controller needing to terminate a login session. Calling req.logOut() removes the req.user property and clears the existing login session. After that, you build the URL that users will be redirected to once the logout is complete as follows:

- Start the URL string with the protocol and hostname:

```
let returnTo = req.protocol + "://" + req.hostname;
```
- Determine if there's a local port being used. If the port is neither undefined, the default port number for web servers using HTTP (80), nor the default port for web servers using HTTPS (443), you append this port to the returnTo URL string if the current Node environment is development:

```
const port = req.connection.localPort;

if (port !== undefined && port !== 80 && port !== 443) {
  returnTo =
    process.env.NODE_ENV === "production"
      ? `${returnTo}/`
      : `${returnTo}:${port}/`;
}
```

- Use the Node.js URL() constructor along with the util.format() to return a formatted string by replacing format specifiers (characters preceded by the percentage sign, %) with a corresponding argument. In this case the string specifier, %s, gets replaced with the value of process.env.AUTH0_DOMAIN. The Auth0 variables will be set up in the next section.

```
const logoutURL = new URL(
  util.format("https://%s/logout", process.env.AUTH0_DOMAIN)
);
```
- Using querystring.stringify, create a URL query string from an object that contains the Auth0 client ID and the returnTo URL:

```
const searchString = querystring.stringify({
  client_id: process.env.AUTH0_CLIENT_ID,
  returnTo: returnTo
});
```

- Using url.search, get and set the serialized query portions of logoutURL:

```
logoutURL.search = searchString;
```
- Finally, redirect the user to logoutURL:

```
res.redirect(logoutURL);
```
The GET /logout endpoint isn't too long but it's quite complex. It leverages different Node APIs to dynamically build a logout path depending on the active Node environment, the port being used, and configuration variables.

All the authentication API endpoints are now complete. You now need to export the router and use it with your Express app. First, append this line to the end of auth.js:

```
// auth.js

// Imports, load .env

// GET /login

// GET /callback

// GET /logout

module.exports = router;
```
Then, open index.js and import the authentication router below the Auth0Strategy definition and mount it on your Express app on the root path, /, right above the route definitions:


```
// index.js

// Load .env, other imports
const Auth0Strategy = require("passport-auth0");

// Import auth router
const authRouter = require("./auth");

// App, port, session, and strategy definitions and config

// App and passport settings
app.use(express.static(path.join(__dirname, "public")));

// Mount auth router
app.use("/", authRouter);

// App routes

// App listening
```
Now, when you click on the login button on the index page, the GET /login endpoint gets called. It was mentioned before that the /login endpoint presents the user with a login page, but where's that page? When using Auth0 as your identity platform, you don't need to create a login page, Auth0 provides one for you with a proper form and secure authentication error messages and prompts.

The last thing that you need to do is to mount a middleware function at the application level to define the value of the isAuthenticated variable used in the index Pug template.

## Creating custom middleware with Express
In index.js add the following code right above the mounting of the authentication router:

```
// index.js

// Load .env, imports
const Auth0Strategy = require("passport-auth0");

const authRouter = require("./auth");

// App, port, session, and strategy definitions and config

// App and passport settings
app.use(express.static(path.join(__dirname, "public")));

app.use((req, res, next) => {
  res.locals.isAuthenticated = req.isAuthenticated();
  next();
});

app.use("/", authRouter);

// App routes

// App listening
```
res.locals is a property used to expose request-level information, such as the authenticated user, user settings, etc. This information is then available to the views rendered during that request-response cycle, such as the templates that use the isAuthenticated variable.

Now, the button present on the index page can change its content based on whether the user is logged in with Auth0 or not:

```
// views/index.pug
...
    div.NavButtons
      if isAuthenticated
        a(href="/user")
          div.NavButton Just dive in!
      else
        a(href="/login")
          div.NavButton Log in
          
 ```
 
 To finally see all this in action, you first need to setup Auth0 by creating an Auth0 tenant and an Auth0 application.
 
 # Setting Up Auth0 with Express and Node.js
Auth0 is a global leader in Identity-as-a-Service (IDaaS). It provides thousands of customers with a Universal Identity Platform for their web, mobile, IoT, and internal applications. Its extensible platform seamlessly authenticates and secures more than 2.5 billion logins per month, making it loved by developers and trusted by global enterprises.

The best part of the Auth0 platform is how streamlined it is to get started by following these five steps.

## Step 1: Signing up and creating an Auth0 application
- If you are new to Auth0, Sign up for a free Auth0 account here. A free account offers you:
- 7,000 free active users & unlimited logins.
- Universal Login for Web, iOS & Android.
- Up to 2 social identity providers like Facebook, Github, and Twitter.
- Unlimited Serverless Rules.
During the sign-up process, you'll create something called a Tenant, which represents the product or service to which you are adding authentication. More on this in a moment.

Once you are signed in, you are welcomed into the Auth0 Dashboard. In the left sidebar menu, click on "Applications". What are Auth0 applications?

Let's say that you have a photo-sharing app called Auth0gram. You then would create an Auth0 tenant called auth0gram. From a customer perspective, Auth0gram is a customer's product or service.

Say that Auth0gram is available on three platforms: web as a single-page application, Android as a native mobile app, and iOS also as a native mobile app. If each platform needs authentication, then you would need to create three Auth0 applications that would connect with each respective platform to provide it with the wiring and procedures needed to authenticate users through that platform. Auth0gram users belong to the Auth0 tenant and are shared across Auth0 applications.

If you have another product called "Auth0chat" that needs authentication, you'd need to create another tenant, auth0chat, and create new Auth0 applications for it depending on the platforms it uses.

With this knowledge, click on the "Create Application" button present in the "Applications" view. A modal titled "Create Application" will open up. You have the option to provide a Name for the application and to choose its type.

You can name this application "WHATABYTE", choose Regular Web Applications as the type, and click on "Create".

You'll be taken to the "Quickstarts" tab where Auth0 provides different guides to get you up and running fast in setting up Auth0 within a project. If you are curious, check out the Node.js quickstart; otherwise, keep reading on.

## Step 2: Creating a communication bridge between Express and Auth0
To reduce the overhead of adding and managing authentication, Auth0 offers the Universal Login page, which is the most secure way to easily authenticate users for web applications.

How does Universal Login work?

When your application calls the GET /login endpoint, users are taken to an Auth0 login page. Once they log in, they are redirected back to your application. With security in mind, for this to happen, you need to configure your Auth0 application with URLs that Auth0 can use to redirect users once they are authenticated.

To configure your Auth0 application, click on the Settings tab. Once there, populate the following fields like so:
- Allowed Callback URLs: http://localhost:3000/callback, http://localhost:8000/callback
- Allowed Logout URLs: http://localhost:3000/, http://localhost:8000/
If you take a look at package.json, you'll see that your app is configured to run through two NPM scripts:
- dev runs your Node.js application using nodemon on port 8000.
- ui creates a static server to serve the frontend of your application on port 3000.
To ensure that Auth0 is able to connect with any of these two environments, you need to enter URLs that use both port numbers.

Save these settings by scrolling down and clicking on "Save Changes".

## Step 3: Adding Auth0 configuration variables to Node.js
Open the .env hidden file that you created earlier and populate it with the following content:

```
AUTH0_CLIENT_ID=
AUTH0_DOMAIN=
AUTH0_CLIENT_SECRET=
```
Head back to your Auth0 application "Settings" tab and populate each property of the .env hidden file with its corresponding Auth0 application value:
- AUTH0_DOMAIN is your Domain value
When you created a new account with Auth0, you were asked to pick a name for your tenant. This name is used as a subdomain of auth0.com and becomes your unique Auth0 application domain. It's the base URL you will be using to access the Auth0 API and the URL where users are redirected in order to authenticate.

Custom domains can also be used to allow Auth0 to do the authentication heavy lifting for you without compromising on branding experience.

- AUTH0_CLIENT_ID is your Client ID
This is a critical value as it protects your resources by only granting tokens to requestors if they're authorized. Think of it as your application's password which must be kept confidential at all times. If anyone gains access to your Client Secret they can impersonate your application and access protected resources.

Together, these variables let your application identify itself as an authorized party to interact with the Auth0 authentication server.

## Step 4: Log in
While your Express app was already depending on variables present on the .env file, these values were not present until recently. This may have caused nodemon and Browsersync to error out and fail if you were running the application. In that case, close any browser window or tab hosting the application, restart nodemon (npm run dev), and then restart Browsersync (npm run ui).
A new browser window or tab will open up. Click the login button to test that the app is communicating correctly with Auth0 and that you can get authenticated.

If everything was set up correctly, you are redirected to the Universal Login page.
As explained earlier, this login page is provided by Auth0 with batteries included. It powers not only the login but also the signup of new users into your application. If you have any existing user already, go ahead and log in; otherwise, sign up as a new user.

Alternatively, you may also sign up and log in with Google as it is turned on by default as a social provider.

If you are signing into an application using Auth0 for the first time, you'll see a dialog asking you to authorize your application to access user profile data. Go ahead and click the green arrow button to authorize access.

An advantage of the Universal Login page is that it is part of the Auth0 domain. It lets you delegate the process of user authentication, including registration, to Auth0 which makes it both convenient and secure.

Unless you signed up with Google, if you created a new user through the sign-up process, you will receive an email asking you to verify your email address. There are tons of settings that you can tweak to customize the signup and login experience of your users, such as requiring a username for registration. Feel free to check out the different options presented to you by Auth0 within the Dashboard and the Auth0 documentation.

Once you are signed up or logged in, you are taken back to the home page of your Express app.

Notice that the label of the button in the home page changed from Login to Just Dive in!, which means that you are authenticated.

## Step 5: Accessing guarded routes
In your application, you will protect the GET /user endpoint using Passport.js middleware to prevent navigation to it if the user is not authenticated. Since you have logged in, when you click on the "Just Dive In!" button that points to /user, you are successfully taken to that view. In case that you were logged out, you should be taken to the Auth0 login screen when trying to access the /user route.

To start, create a middleware function named secure to protect routes and ensure they are only accessible if the user is logged in. Add the following code above the route definitions of index.js:

```
// index.js

// Other code...

const secured = (req, res, next) => {
  if (req.user) {
    return next();
  }
  req.session.returnTo = req.originalUrl;
  res.redirect("/login");
};

app.get("/", (req, res) => {
  res.render("index", { title: "Home" });
});

// Rest of the code...
```
Then, update the GET /user endpoint so that it uses that middleware function as follows:

```
// index.js

// Other code...

app.get("/user", secured, (req, res, next) => {
  const { _raw, _json, ...userProfile } = req.user;
  res.render("user", {
    title: "Profile",
    userProfile: userProfile
  });
});

// Rest of the code...
```
You use Javascript object destructuring to unpack values from the req.user object into distinct variables. Using the optional object argument of the res.render() method, you pass the userProfile variable to the user view template along with the page title.

Visit http://localhost:3000/user on the browser, if you are logged in, you will now see the email address you used to log into the application in the greeting banner and the userProfile object shown formatted as JSON within the content container.

If you click on the "Log Out" button, you'd be taken to the index page. Visit http://localhost:3000/user again and you'll be taken to the login page provided by Auth0. Log in back again and you'll be back into the user page.

# Authentication Integration Completed
That's it! In this part of the tutorial, you learned how Passport.js works, how to configure it, and how to integrate it with Node.js and Auth0 to add authentication to web applications. You also learned about security and identity best practices and how an identity platform such as Auth0 lets you delegate the giant responsibility of keeping logins secure to a team of experts.

# Thank you for your time.
# if you like the tutorial fork it or give it a star.
