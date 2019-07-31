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


