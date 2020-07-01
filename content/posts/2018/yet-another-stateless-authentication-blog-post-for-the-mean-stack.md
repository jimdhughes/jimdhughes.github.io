+++
author = "James D Hughes"
categories = ["nodejs", "REST", "JWT", "ExpressJS", "Middleware", "development", "JavaScript", "MongoDB", "async"]
date = 2018-05-05T03:32:35Z
description = ""
draft = false
slug = "yet-another-stateless-authentication-blog-post-for-the-mean-stack"
tags = ["nodejs", "REST", "JWT", "ExpressJS", "Middleware", "development", "JavaScript", "MongoDB", "async"]
title = "Yet Another Stateless Authentication Blog Post for the MEAN stack."

+++


#inb4: this ain't new.

I'm writing this for everyone that wants a *slightly* more organized approach to express middleware and authentication.  I'm writing this because once again I was inspired by how much I **adore** Node, Express, and all the delights that come from being able to implement my API's and leverage middlewares.  Also, I'm using async / await, which is pretty neat.

I'm going to use MongoDB as a datastore it just jives so well with Node.

So - let's bootstrap if you don't have express-generator installed, do it. `npm i -g express-generator`
```console
$ express express-jwt-tutorial
$ cd express-jwt-tutorial
$ npm i jsonwebtoken mongoose bcryptjs --save
```

I like to add my folders immediately after I make a new project so...
```console
$ mkdir models
$ mkdir middleware
$ mkdir config
```

First things first - our user model:

```javascript
//models/user.model.js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = mongoose.Schema({
    email:{type:String, required:true, unique:true},
    password:{type:String, required:true}
});

//encrpyt a password
userSchema.methods.generateHash = function(password) {
  return bcrypt.hashSync(password, 10, null);
}

//Check to see if password is a valid hash
userSchema.methods.validPassword = function(password) {
    return bcrypt.compareSync(password, this.password);
}

//Hash password if it's been changed
userSchema.pre('save', function(next){
  if(this.isModified('password')){
    console.log('password changed');
    this.password = this.generateHash(this.password);
  }
  next();
});

//export the model
module.exports = mongoose.model('User', userSchema);

```

I added in some methods to create and validate hashes for passwords. Never store those passwords in plain text!

Below is an example mongodb configuration file that I'll put in the `config` folder
```javascript
// config/db.js
module.exports = {
    development:'mongodb://localhost/someapp'
}
```

I like to keep my mongodb initializations in one file.  Set up the connections, load up the models, etc. This makes it easy to get the app up and running with a single line of code later. Below is an example of how I do this.

```javascript
// models/index.js
const mongoose = require('mongoose');
const env = process.env.NODE_ENV || 'development';
const db = require('../config/db')[env];

mongoose.connect(db);

require('./user.model');
```
Onto bootstrapping...

In `app.js` I usually do the following immediately:
1. Delete the predefined `index` and `user` routes
2. Delete the predefined route mappings denoted by `app.use('/', indexRouter)` and `app.use('/users/', usersRouter)`

Next: initialize mongo by adding a require line. I typically put this just below the app initialization:

```javascript
//app.js
...
var app = express();

//Load Mongoose Models
require('./models');

// view engine setup
...
```
# Run it! 
You can use npm start, or whatever mechanism you like to use to get your app up and running.  I like to use nodemon for livereload, but it's totally up to you.

# Define some Routes
Time to define some endpoints!
Again - I have a flavour of setup that I like to use - so that's what I'm going to show!
Under the `routes` folder, open up `index`, wipe it out, and add the following:

```javascript
 // routes/index.js
 module.exports = {
   users:require('./users')
 }
```

Open up `users` and replace it with the following:

```javascript
//routes/users.js
const router = require('express').Router();

router.route('/')
    .get(getUsers);

router.route('/:id')
    .get(getUser);

module.exports = router;

async function getUsers(req, res, next){
  next(new Error('Not Yet Implemented'));
}

async function getUser(req, res, next) {
  next(new Error('Not Yet Implemented'));
}
```

Now it's pretty easy to set up our endpoints. Return to app.js and add the following code.  You will need to import it after you load your mongoose models.

```javascript
//app.js

//Load Mongoose Models
require('./models');
var routes = require('./routes');
...
// Middleware
...
app.use('/api/v1/users', routes.users);
```

We've written all this code -- but haven't actually done anything yet.

Let's build in some funcitonality! We'll start with registration and authentication.  Some people will suggest you use something like passport.js - which I have used previously. However since I am going to go stateless, I find it completely unnecessary to do this when it really just takes a few lines of code!

Go ahead and create your authentication router.

```javascript
// routes/authentication.js
const router = require('express').Router();
const User = require('mongoose').model('User');

router.route('/login')
  .post(doLogin)

router.route('/register')
  .post(doRegister);

module.exports = router;

async function doLogin(req, res, next) {
 next(new Error('Not Yet Implemented'));
}

async function doRegister(req, res, next) {
  next(new Error('Not Yet Implemented'));
}
```

To handle a registration, life is pretty easy. We can just create a new user.  Our Mongoose Model definition will handle the encrypting of passwords and the like for us so we don't really need to worry too much about this. We could maybe add some validations, but they're not totally necessary.

```javascript
...
async function doRegister(req, res, next) {
  let user = req.body;

  if (!user.email) {
    return res.status(400).json({ message: 'Require Email For User' });
  }
  if (!user.password) {
    return res.status(400).json({ message: 'Require Password For User' });
  }
  try {
    let result = await User.create(user);
    res.status(201).json({message:'Successfully Registered! Please Login'});
  } catch(err) {
    //You can handle this better.  I'm not going to though
    next(err);
  }
}
```
I'm not going to give you a tutorial on how to issue HTTP requests. I use Postman on windows and CURL or Postman on Linux. It's up to you, but you can now POST to api/v1/auth/register and send a body with an email and password and we can now register! It's that easy!  After you post a user to api/v1/auth/register, you should get a return value with our little json message: 'Successfully Registered! Please Login'.  Which I would LOVE to do...  If I had a login method!

So to create a login controller:
```javascript
...
async function doLogin(req, res, next) {
  let credentials = req.body;
  if(!credentials.email) {
    return res.status(400).json({message:'Require Email'});
  }
  if(!credentials.password) {
    return res.status(400).json({message:' Require Password'});
  }
  try {
    let u = await User.findOne({email:credentials.email});
    if(!u) {
      return res.status(404).json({message:'User Not Found!'});
    }
    let validPassword = u.validPassword(credentials.password);
    if(!validPassword) {
      return res.status(401).json({message:'Invalid Username / Password combination'});
    }
    return res.status(200).json({message:'Welcome!'});
  } catch(err) {
    next(err);
  }
}
...
```

Go ahead and POST your email / password to the `api/v1/auth/login` endpoint and you will get our nice Welcome message.

But wait --
This doesn't really DO anything.  Sure your challenge is correct, and if you wanted to send an email and password on every request you COULD do it this way..  OR you could sign a json web token!
That's what we're going to do.
First - We need to set up a secret for our app.  This will need to be shared across all servers that could handle your authentication so it can get a bit tricky.  You could use a super secure key or a RSA cert or the like.  We're going to just use a big string.

So - let's make an app config file at `config/properties.js`
```javascript
// config/properties.js
module.exports = {
  secret:'thisisasecretthatisnotinsecureatall'
}
```
Go back to `routes/authentication.js` and import the jsonwebtoken library.
```javascript
// routes/authentication.js
...
const jwt = require('jsonwebtoken');
const config = require('../config/properties');
...
```

Now we can update our doLogin function to create a token and send it back to the client! We're going to add code JUST before we return the success message.

```javascript
async function doLogin(req, res, next) {
...
    let token  = jwt.sign({id:u._id}, config.secret);
    return res.status(200).json({message:'Welcome!', token: token});
...
}
```

If you POST a login again, you will get the welcome message AND a token! Make sure you keep it safe ;) (Truth be told, plain old json sending a token is probably not the best idea.  Using a cookie or something might be a little more secure, regardless. There's some potential security issues)

Now, what do you do next? We have this token, and logging in, but what is the POINT of it all?
Well, we need to secure some endpoints. Let's set it up so that a user can access their profile with a new router: `routes/me.js`

```javascript
// routes/me.js
const router = require('express').Router();

router.route('/')
  .get(getMe);
  
module.exports = router;

async function getMe(req, res, next) {
  next(new Error('Not Yet Implemented'));
}
```

This would be neat, but if you're thinking even remotely logically - you will know why this is pointless.  Nothing exists to determine who it is that's making the request! That's where the joyous middleware comes into play.

Let's make a new middleware file at: `middleware/checkToken.js`
```javascript
// middleware/checkToken.js
const jwt = require('jsonwebtoken');
const config = require('../config/properties');
const User = require('mongoose').model('User');

module.exports = async function(req, res, next) {
  let token = req.headers['x-auth-token'];
  if(!token) {
    return res.status(403).json({message:'No Token Provided'});
  }
  try{
    let payload = await jwt.verify(token, config.secret);
    let u = await User.findOne({id:payload._id}, {password:false});
    req.user = u;
    next();
  } catch(err) {
    res.status(403).json({message:'Invalid Token'});
  }
}
```

Then it's super easy to use! Add this middleware before each call in your `routes/me.js` or any route you want to be secured by at minimum an authentication!
```javascript
// routes/me.js
const router = require('express').Router();
const checkToken = require('../middleware/checkToken');

router.route('/')
  .get(checkToken, getMe);
  
module.exports = router;

async function getMe(req, res, next) {
  res.json(req.user);
}
```
Add `me.js` to your `routes/index.js` in the same manner as the other imports:
```javascript
// routes/index.js
module.exports = {
  users: require('./users'),
  authentication: require('./authentication'),
  me: require('./me')
}
```

Then add an endpoint in `app.js`
app.use('/api/v1/me', routes.me);

Add the `x-auth-token` header to your `api/v1/me` request with the token that was generated from your login and boom! You've got authorization!
Try again without sending a token and you will get our 'No Token Provided' message.
Pretty cool, yeah?

This is pretty simple example, but what I really wanted to discuss is next.  The middleware for even more functionality!

Adding in some permission-level authentication in this is super easy now that we have this architecture. First things first, let's adjust our user model to have a 'role' property that can be any of 'user', 'admin', or 'manager'. It'll look something like this:

```javascript
// models/user.model.js
...
const userSchema = mongoose.Schema({
    email:{type:String, required:true, unique:true},
    password:{type:String, required:true},
    role:{type:String, enum:['user', 'admin', 'manager'], required:true, default:'user'}
});
...
```

For entertainment purposes, we're going to add a new resource and simply call it asset. We will need a mongoose model for it to begin:

```javascript
// models/asset.model.js
const mongoose = require('mongoose');

const assetSchema = mongoose.Schema({
  title:{type:String, required:true},
  properties:{type:Object},
  assignedTo:{type:mongoose.Schema.Types.ObjectId, required:false},
  assignedAt:{type: Date, default: function() { return new Date()}}
});

module.exports = mongoose.model('Asset', assetSchema);
```
Require it in our `models/index.js` file as we did previously:
```javascript
// models/index.js
...
require('./asset.model');
...
```
Next we'll make a basic CRUD router for the asset:
```javascript
// routes/assets.js
const router = require('express').Router();

router.route('/')
  .get(getAssets)
  .post(createAsset);

router.route('/:id')
  .get(getAsset)
  .put(updateAsset)
  .delete(deleteAsset);

module.exports = router;

async function getAssets(req, res, next) {
  next(new Error('Not Yet Implemented'));
}

async function createAsset(req, res, next) {
  next(new Error('Not Yet Implemented'));
}

async function getAsset(req, res, next) {
  next(new Error('Not Yet Implemented'));
}

async function updateAsset(req, res, next) {
  next(new Error('Not Yet Implemented'));
}

async function deleteAsset(req, res, next) {
  next(new Error('Not Yet Implemented'));
}
```
Then add it to our index and app.js as we did previously. 
```javascript
// SEE ABOVE. It's the exact same as before
// routes/index.js
...
assets:require('./assets')
...

//app.js
app.use('/api/v1/assets', routes.assets);
```

The use case for this asset is as follows:
1. Assets can be assigned to any user
2. Administrators can assign assets to any individual
3. Users can only view the assets that are assigned to them
4. Manager can view all assets but can't edit or create them.

The first and foremost requirement of all of these scenarios is that a user must be authenticated to view.  So to do this, we add the `checkToken` middleware to all those routes.

```javascript
// routes/asset.js
const router = require('express').Router();
const checkToken = require('../middleware/checkToken');

router.route('/')
  .get(checkToken, getAssets)
  .post(checkToken, createAsset);

router.route('/:id')
  .get(checkToken, getAsset)
  .put(checkToken, updateAsset)
  .delete(checkToken, deleteAsset);
...
```

This is all fine and dandy, but what about those role requirements?
More middleware?
More middleware!

```javascript
// middleware/authorize.js
module.exports = {
  hasRole: function(role) {
    return function(req, res, next) {
      if(role === req.user.role) {
        next();
      }
      else{
        return res.status(403).json({message:'Unauthorized'});
      }
    }
  },
  hasAnyRole: function(roles) {
    return function(req, res, next) {
      let authorized = false;
      if(!(roles instanceof Array)) {
        roles = [roles];
      }
      roles.forEach(r => {
        if(r === req.user.role) {
          authorized = true;
        }
      });
      if(authorized){
        next();
      }
      else{
        return res.status(403).json({message:'Unauthorized'});
      }
    }
  }
}
```

This will allow us to implement all sorts of combinations of permission checking! I implemented the two that I need right now, but if you had an app with some more complicated permissions structure you could do something like `hasAllRoles` or `doesNotHaveRole` or `hasNoRoles` for some reason.

Next we can just chain these into our endpoints in our `asset` router.
```javascript
//routes/asset.js
const router = require('express').Router();
const Asset = require('mongoose').model('Asset');

const checkToken = require('../middleware/checkToken');
const authorize = require('../middleware/authorize');

router.route('/')
  .get(checkToken, authorize.hasAnyRole(['admin', 'manager']), getAssets)
  .post(checkToken, authorize.hasRole('admin'), createAsset);

router.route('/:id')
  .get(checkToken, authorize.hasAnyRole(['admin', 'manager']), getAsset)
  .put(checkToken, authorize.hasRole('admin'), updateAsset)
  .delete(checkToken, authorize.hasRole('admin'), deleteAsset);
```

Note that I put the checkToken middleware first.  This is because it allows us to check for a token first (and if there isn't one, end the request) and insert the user represented by the token into the request data.  It is then accessible by the next piece of middleware.

Once we implement the rest of our functions, the asset class will look something like this:
```javascript
const router = require('express').Router();
const Asset = require('mongoose').model('Asset');

const checkToken = require('../middleware/checkToken');
const authorize = require('../middleware/authorize');

router.route('/')
  .get(checkToken, authorize.hasAnyRole(['admin', 'manager']), getAssets)
  .post(checkToken, authorize.hasRole('admin'), createAsset);

router.route('/:id')
  .get(checkToken, authorize.hasAnyRole(['admin', 'manager']), getAsset)
  .put(checkToken, authorize.hasRole('admin'), updateAsset)
  .delete(checkToken, authorize.hasRole('admin'), deleteAsset);

module.exports = router;

async function getAssets(req, res, next) {
  try {
    let assets = await Asset.find();
    res.json(assets);
  } catch(err) {
    next(err);
  }
}

async function createAsset(req, res, next) {
  let asset = req.body;
  try {
    let a = await Asset.create(asset);
    res.status(201).json(a);
  } catch(err) {
    next(err);
  }
}

async function getAsset(req, res, next) {
  let id = req.params.id;
  try {
    let asset = await Asset.findById(id);
    res.json(asset);
  } catch(err) {
    next(err);
  }
}

async function updateAsset(req, res, next) {
  let id = req.params.id;
  let asset = req.body;
  try {
    let result = await Asset.update({_id:id}, asset, {new:true});
    res.json(result);
  } catch(err) {
    next(err);
  }
}

async function deleteAsset(req, res, next) {
  let id = req.params.id;
  try {
    let done = await Asset.deleteOne({_id:id});
    if(done) {
      return res.status(200).json({message:'Deleted'});
    }
  } catch(err) {
    next(err);
  }
}
```

I'm not going to prove it, but if you want to test the use cases, go ahead and make a couple of users and try to access each endpoint as each of them :D.

For most API's I've grown quite fond of implementing features for individuals with the `/me` endpoint. To complete our previous list of user requirements, I'm going to simply implement the feature for getting assets but by using the `/me` endpoint as the only requirement is not permissions based, but identity based.

```javascript
// routes/me.js
...
router.route('/assets')
  .get(checkToken, getMyAssets);
...
async function getMyAssets(req, res, next) {
  try {
    let assets = await Asset.find({ assignedTo: req.user._id });
    res.json(assets);
  } catch (err) {
    next(err);
  }
}
```

and Voila. We can get all the assets that's assigned to the currently authenticated user.

Isn't that fun?

~~I'm going to post the source code on my github - I'll update the post when I do!~~

The source is up at https://github.com/Daimonos/jdhc-express-stateless-template

Cheers!

