# OAuth

## Learning Competencies
- Introduction to `auth` standard
- Authenticate an `express` application using `oauth`

## Overview


## Introduction to Concepts
### Wiki Says
> OAuth is an open standard for access delegation, commonly used as a way for Internet users to grant websites or applications access to their information on other websites but without giving them the passwords. This mechanism is used by companies such as Google, Facebook, Microsoft and Twitter to permit the users to share information about their accounts with third party applications or websites. Follow [Wiki](https://en.wikipedia.org/wiki/OAuth) for more. 



### oAuth
So far we have seen how to authenticate using a local strategy (username/password), but what happens if we want to authenticate through another provider like Facebook, Twitter, LinkedIn, or GitHub? To do that we use a protocol called oAuth. There are currently two versions of oAuth, 1.0 and 2.0 and some providers use 1.0 while others use 2.0. They are not very different; here is how it works:

- A user is first redirected to the service provider to authorize access.
- After authorization has been granted, the user is redirected back to the application with a code that can be exchanged for an access token.
- The application requesting access, known as a client, is identified by an ID and secret.

To set this up, we have to create a developer account with a provider (Facebook, Twitter, etc). Once we create an account we can create applications and receive a client_id, client secret key, and specify our callback URL (the url which we would like to tell the provider to go back to when the user has finished authenticating on the provider's website). These values - especially the secret key - should not be published on GitHub **ever** and should be hidden as environment variables.
## For example
### `passport-facebook`
Let's create a sample application to authenticate through Facebook.

In order to successfully authenticate through Facebook, you will need to create a new application at https://developers.facebook.com/. You can create a new app, call it whatever you'd like, and choose whatever category you'd like. You can then head to your dashboard where you will see your App ID and App Secret (which will be hidden until you click show). You can head over to "Settings" and add `localhost` to your App Domains and `http://localhost:3000/auth/facebook/callback` to your Site URL.

Next, let's create our application. Here's some code to get you started:
```js
var express = require("express");
var app = express();
var methodOverride = require("method-override");
var morgan = require("morgan");
var bodyParser = require("body-parser");
var passport = require("passport");
var findOrCreate = require('mongoose-findorcreate');
var FacebookStrategy = require('passport-facebook').Strategy;
var mongoose = require("mongoose");
var session = require("cookie-session");
var flash = require("connect-flash");
mongoose.Promise = Promise

app.set("view engine", "pug");
app.use(express.static(__dirname + "/public"));
app.use(morgan("tiny"));
app.use(bodyParser.urlencoded({extended:true}));
app.use(methodOverride("_method"));

app.use(session({ secret: process.env.SECRET_KEY }));
app.use(passport.initialize());
app.use(passport.session());
app.use(flash());

// Redirect the user to Facebook for authentication.  When complete,
// Facebook will redirect the user back to the application at
//     /auth/facebook/callback
app.get('/auth/facebook', passport.authenticate('facebook'));

// Facebook will redirect the user to this URL after approval.  Finish the
// authentication process by attempting to obtain an access token.  If
// access was granted, the user will be logged in.  Otherwise,
// authentication has failed.
app.get('/auth/facebook/callback',
  passport.authenticate('facebook', { successRedirect: '/',
                                      failureRedirect: '/login' }));

app.get("/", function(req, res, next){
    // send the authenticated user
  res.send(req.user);
});

app.get("/logout", function(req, res, next){
    // send the authenticated user
  req.logout();
  res.redirect('/')
});

var userSchema = new mongoose.Schema({
    facebook_id: String
})

userSchema.plugin(findOrCreate); // give us a findOrCreate method!

var User = mongoose.model('User', userSchema);

// instead of a username and password we will be using information that Facebook has given us when we create an application with them
passport.use(new FacebookStrategy({
    // these should ALL be values coming from a .env file
    clientID: process.env.FACEBOOK_APP_ID,
    clientSecret: process.env.FACEBOOK_APP_SECRET,
    // when you deploy your application you can add an environment variable for CALLBACK_URL, right now let's stick with localhost:3000/auth/facebook/callback
    callbackURL: process.env.CALLBACK_URL || "http://localhost:3000/auth/facebook/callback"
  },
  // in the verify callback we will get an accessToken to make authenticated requests on the users behalf along with a refreshToken which is used in some authentication strategies to refresh an expired accessToken. We also are given an object called "profile" which has data on the authenticated user
  function(accessToken, refreshToken, profile, done) {
    User.findOrCreate({facebook_id: profile.id}, function(err, user) {
      if(err) done(err);
      done(null, user);
    });
  }
));

// same process as before:
passport.serializeUser(function(user, done) {
  done(null, user.id);
});

// same process as before:
passport.deserializeUser(function(id, done) {
  User.findById(id, function(err, user) {
    done(err, user);
  });
});

app.listen(3000, function(){
  console.log("Server is listening on port 3000");
});
```
You might be asking, should I store the access token in the database? You can read more about that [here](http://security.stackexchange.com/questions/72475/should-we-store-accesstoken-in-our-database-for-oauth2) and [here](http://security.stackexchange.com/questions/80727/best-place-to-store-authentication-tokens-client-side).


## Exploration
- Follow the documentation [link](http://passportjs.org/docs/facebook) to get more information about Passport.js
- Check out [this](http://tutorials.jenkov.com/oauth2/index.html) beautifully designed OAuth 2.0 tutorial.
- Read [this](https://tests4geeks.com/oauth2-javascript-tutorial/) blog by *Test4Geeks* on **OAuth JavaScript**.


Note: You can find a sample app with passport and facebook [here.](https://github.com/rithmschool/node_curriculum_examples/tree/master/passport_oauth)
