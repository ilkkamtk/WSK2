<!-- TODO: ES6 import instead of Node require(), e.g. https://github.com/ilkkamtk/SSSF21-gql-hello/blob/main/server.js -->
<!-- TODO: change helmet for contentSecurityPolicy: false only on localhost to have graphql playground to work -->
# Few steps into security

## HTTPS

* [HTTPS](https://en.wikipedia.org/wiki/HTTPS)
  * HTTP over [TLS/SSL](https://en.wikipedia.org/wiki/Transport_Layer_Security)
    * TLS is the new progression of SSL
  * _authentication_ of the visited website
  * protection of the _privacy_ and _integrity_ of the exchanged data
* Generally better to implement TLS in reverse-proxy such as Apache or Nginx

### express

* Force redirection from HTTP to HTTPS

```javascript
import http from 'http';

http.createServer((req, res) => {
  res.writeHead(301, { 'Location': `https://localhost:8000${req.url}` });
  res.end();
}).listen(3000);
```
Notes:
* about [HTTP status code](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html), the 3XX codes are redirects, 301 means Moved Permanently.
* when testing with your browser, because of self-signed certificate, it will show a warning page instead of your app. In Chrome, click [details and proceed unsafe](https://support.google.com/chrome/answer/99020?co=GENIE.Platform%3DDesktop&hl=en-GB). In Firefox, click [Advanced and Accept the Risk and Continue](https://support.mozilla.org/en-US/kb/what-does-your-connection-is-not-secure-mean).

### express (production server)

* A valid certificate in Azure can be used for all its subdomains
  * [Make sure you enable SSL](https://learn.microsoft.com/en-us/azure/app-service/configure-ssl-certificate?tabs=apex%2Cportal)
  * navigate to your app with https://
* Eventually, force the redirection from HTTP to HTTPS

```javascript
'use strict';

import express  from 'express';
const app = express();

app.enable('trust proxy');

// Add a handler to inspect the req.secure flag (see
// https://expressjs.com/en/api.html#req.secure). This allows us
// to know whether the request was via http or https.
// https://github.com/aerwin/https-redirect-demo/blob/master/server.js
app.use ((req, res, next) => {
  if (req.secure) {
    // request was via https, so do no special handling
    next();
  } else {
    // request was via http, so redirect to https
    res.redirect(`https://${req.headers.host}${req.url}`);
  }
});

app.listen(3000);
```

### consider separating development and production code

* create node modules for localhost (development) and remote (production) server
  * cut/paste corresponding code and use node export e.g. for localhost.js

 ```javascript
// cut-pasted code about localhost: import, tls certs, options,...
// note, if you do the dotenv import in app.js and import this
// after, you don't need to redo it here
const httpsPort = process.env.HTTPS_PORT || 8000;

export default (app, port) => {
  https.createServer(options, app).listen(httpsPort);
  http.createServer(httpsRedirect).listen(httpPort);
};
```

* use .env file to choose which code to load
  * Note: in Azure, the NODE_ENV is defined in 'Configuration' by adding 'New application setting', setting it in .env will have no effect.

```javascript
process.env.NODE_ENV = process.env.NODE_ENV || 'development';
if (process.env.NODE_ENV === 'production') {
  (async () => (await import('./utils/production')).default(app, port))();
} else {
  (async () => (await import('./utils/localhost')).default(app, port))();
}
app.get('/', (req, res) => {
  res.send('Hello Secure World!');
});
```

---

## Authentication

* Process of validating the user is who she claims to be
* Usually done online by user providing username and password
* For APIs tokens are usually used because:
   * Single user can have multiple tokens
   * If one token is exposed, it can be easily disabled
   * Tokens can have explicit access restrictions and thus prevent possible damages on exposure


### express

* Implementing security-related things yourself is generally not smart
* Still, for the sake of example, authentication can be implemented as Express.js middleware:

```javascript
app.use((req, res, next) => {
  if (req.query.token === 'SECRET_TOKEN') {
    next();
  }
  else {
    res.status(401).send('Please sign in.');
  }
});
```

### [Passport.js](http://passportjs.org/)

* De facto standard authentication solution for Express.js applications
* Flexible & does not mount routes
* Supports OAuth (Facebook, Twitter, etc.) & OpenID
* Has over 500 login strategies available
* (check week one task on usage)

## bcrypt

* never store user password in plain text
  * avoid returning password and salt in results from DB queries
* use a one way hash [key stretching](https://en.wikipedia.org/wiki/Key_stretching) function with random salt like [bcrypt](https://en.wikipedia.org/wiki/Bcrypt) (or scrypt, argon2, etc.)
  * why [not](https://codahale.com/how-to-safely-store-a-password/) md5, sha1, sha256,...?
  * try to be slow when computing hashes
* you can use NodeJS native [crypto](https://nodejs.org/api/crypto.html) or [bcrypt package](https://www.npmjs.com/package/bcrypt)
* Installation:

```shell
$ npm install bcrypt --save
```

* Basic usage, hash (e.g. in user controller):

```javascript
'use strict';

import bcrypt from 'bcrypt';

const saltRound = 12; //okayish in 2022

// inside your user create/update async function
const myPwd = 'Secret123😉';
const hash = await bcrypt.hash(myPwd, saltRound);
// Store hash in the database
```

* Basic usage, check hash (e.g. in passport utils):

```javascript
//strict, require, etc.

passport.use(new Strategy(
  async (username, password, done) => {
    try {
      const user = await userModel.getUserLogin(username);
      console.log('Local strategy', user); // result is binary row
      if (user === undefined) {
        // consider artificial slowness to simulate slow password check (and anyway slowdown brute force attack)
        // setTimeout(() => { /*done*/ },  Math.floor(Math.random() * 1000));
        return done(null, false, {message: 'Wrong cretendials.'});
      }
      if (!await bcrypt.compare(password, user.password)) {
        return done(null, false, {message: 'Wrong cretendials.'});
      }
      delete user.password; // make sure that the password do not travel around...
      return done(null, {...user}, {message: 'Logged In Successfully'}); // use spread syntax to create shallow copy to get rid of binary row type
    } catch (err) {
      return done(err);
    }
  }));
```

---

## Authorization

* After user is identified, it needs to be determined whether user has access to certain resource
* Often much more complicated than authentication
* Some packages
  * [accesscontrol](https://www.npmjs.com/package/accesscontrol) (not maintained since 2018)
  * [role-acl](https://www.npmjs.com/package/role-acl)

---

## Helmet

* [Helmet](https://www.npmjs.com/package/helmet) secures Express.js application by modifying multiple headers.
* "It's not a silver bullet, but it can help!"
* Installation:

```shell
$ npm install helmet --save
```

* Basic usage:

```javascript
import helmet from 'helmet';
app.use(helmet());
```

* `helmet()` applies all default measures. To use only some, enable only those:

```javascript
app.use(helmet.noCache())
app.use(helmet.frameguard())
```

* or disable some:

```javascript
app.use(helmet({
  frameguard: false
}));
```

---


## CORS

* AJAX calls to other domains are forbidden by default by the [same-origin security policy](https://en.wikipedia.org/wiki/Same-origin_policy)
* Implemented by basically all browsers
* Can be relaxed by enabling `Access-Control-*` headers
* [Flow](https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/Flowchart_showing_Simple_and_Preflight_XHR.svg/2000px-Flowchart_showing_Simple_and_Preflight_XHR.svg.png)

### CORS with Express.js

* To enable Cross-Origin Resource Sharing (CORS), [cors package](https://www.npmjs.com/package/cors) can be used
* It supports enabling CORS for all routes, for single routes, and for pre-flight requests
* Installation:

```shell
$ npm install --save cors
```

* Enable for all requests:

```javascript
import cors from 'cors';
app.use(cors());
```

---

### Exercise

Implement CORS so that your application only allows CORS for POST on path `/cors-enabled`.

---

## Dependency security

* "Security of your app is only as strong as the “weakest link” in your dependencies."
* To check your dependencies for known vulnerabilities, run [security audit](https://docs.npmjs.com/auditing-package-dependencies-for-security-vulnerabilities)
* The [npm audit](https://docs.npmjs.com/cli/audit) command is available in npm@6. To upgrade, run ``npm install npm@latest -g``

```shell
$ npm audit
```

* If security vulnerabilities are found and updates are available, you can either:
   * Run the ``npm audit fix`` subcommand to automatically install compatible updates to vulnerable dependencies.
   * Run the recommended commands individually to install updates to vulnerable dependencies. (Some updates may be breaking backward compatibility)

Some remote git repository provider (like [gihub](https://help.github.com/en/articles/about-security-alerts-for-vulnerable-dependencies), [gitlab](https://docs.gitlab.com/ee/user/application_security/dependency_scanning/), etc.) can check for dependencies vulnerability (e.g. on every commit, at regular intervals,...)

---

### Exercise
Check your project for known vulnerabilities<br>
If you have old projects (e.g. some node labs/assignments from previous year), try with them 😉

---

## Update Local Packages

It's a good practice to periodically update the packages your application depends on. Then, if the original developers have improved their code, your code will be improved as well (Hopefully).

In the same directory as the package.json file of the application that you want to update:

```shell
$ npm update
$ npm outdated
# There should not be any results....
```

---

### Major update

* By default, ``npm update`` only updates minor versions (e.g. ``mongoose ^5.0.11  →  ^5.0.13`` or ``body-parser ^1.17.1  →  ^1.18.2``).
* Major version (e.g. ``dotenv ^4.0.0  →  ^5.0.1`` or ``passport ^0.3.2  →  ^0.4.0``) can bring nice new features; but DO NOT GUARANTEE BACKWARD COMPATIBILITY! Double check change logs, readme, etc. to know what changes you might need in your code.
  * Option 1: manually update package.json (or package-lock.json)
  * Option 2: [npm-check-updates](https://www.npmjs.com/package/npm-check-updates)

```shell
$ npm install -g npm-check-updates
$ ncu
```

Upgrade a project's package file:

**Make sure your package files (`package.json` and `package-lock.json`) are in version control and all changes have been committed. This will overwrite your package files.**

```shell
$ ncu -u
```

ncu can check for global packages:

```shell
$ ncu -g
```

---

### Exercise

Check if your packages are up-to-date (at least minor version)<br>
If you can, try with an old project
