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
