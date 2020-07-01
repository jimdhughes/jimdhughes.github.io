+++
author = "James D Hughes"
categories = ["nodejs", "async/await", "asynchronous"]
date = 2018-05-31T01:35:29Z
description = ""
draft = false
slug = "async-await-in-node-js"
tags = ["nodejs", "async/await", "asynchronous"]
title = "Async / Await in Node.js"

+++


The Async / Await functionality introduced into Node.js in v7.10.0 is legitimately a godsend.  The Async / Await paradigm works with promises to do exactly what the paradigm implies.  It will asynchronously call the function and await it's response before continuing on with the rest of the function. Previously, you could do this with promise chaining. Passing results from one promise to the next. Where this issue fell apart is when you needed to use the result of Promise 1 in Promise 4. You would need to somehow pass the result of Promise 1 through Promises 2 and 3 in order to use them in promise 4.

Here's an example of a relatively easy to read chain of promises that I use in a production application to generate a user's timesheet.

```javascript
function getTimesheet(user, date, offset) {
  return new Promise((resolve, reject) => {
    getCompanyConfig(user)
      .then((result) => {
        return getTimesheetObject(result.config, result.user, date, offset);
      })
      .then((timesheetObject) => {
        return executeTimesheetRequest(timesheetObject);
      })
      .then((timesheetBuffer) => {
        resolve(timesheetBuffer);
      })
      .catch((e) => {
        console.error(e);
        reject(e);
      });
  });
}
```

Imagine this with callback hell. I thought promises were delightful until async / await was introduced.Essentially this function takes in a user, the date they're requesting a timesheet for, and the timezone_offset (calculated from the request object as we're located in multiple timezones). The function returns a promise of a binary buffer of the excel-based timesheet report required for legacy systems and for sign off by managers and clients.

Now to refactor this as a beautiful async / await function!

```javascript
async function getTimesheet(user, date, offset) {
  try {
      let config = await getCompanyConfig(user);
      let timesheetObject = await getTimesheetObject(config.config, config.user, date, offset);
      let buffer = await executeTimesheetRequest(timesheetObject);
      return buffer;
  } catch (e) {
    console.error(e);
    throw e;
  }
}
```
There we have it. The code is way more readable and flexible.  If i needed to pass that configuration object into by executeTimesheetRequest() function then I could just do it instead of having to find inventive ways of chaining the config through the getTimesheetObject() function.  I've even left this open enough to refactor the getcompanyConfig() function into two separate functions (as I originally intended) but had problems with returning the results of two promises to the getTimesheetObject() function.

In the first bit of code it is obvious that we need to do some extra work to get both the user and config object to chain into the next function.

Here is an example of my getCompanyConfig function that I'm going to also convert to async/await:

```javascript
function getCompanyConfig(userId) {
  return new Promise((resolve, reject) => {
    User.findById(userId, {
      include: [models.Company]
    })
      .then((u) => {
        if (!exportConfiguration.templates[u.Company.name]) {
          console.error('No Configuration Avaialable for Company: ' + u.Company.name);
          reject(new Error('No Configuration available for company: ' + u.Company.name));
        } else {
          resolve({
            user: u,
            config: exportConfiguration.config
          });
        }
      })
      .catch((e) => {
        console.error('Error Finding User');
        reject(e);
      })
  });
}
```

It's pretty verbose and a touch confusing.

```javascript
async function getCompanyConfig(userId) {
  try {
    let user = await User.findById(userId, {include:[models.Company]});
    if (!exportConfiguration.templates[user.Company.name]) {
      throw new Error('No Template Available for Company: '+u.Company.name);
    }
    return {
      user: user,
      config: exportConfiguration.config
    }
  } catch (e){
    throw e;
  } 
}
```

Smaller. Tighter. Cleaner. I can now also break this into two smaller functions that return more meaningful things.  1 - The getUser function and the getCompanyConfig function which will return ONLY the user information and the company information as required.

First we'll take the user definition out of the getCompanyConfig controller and put it in the getTimesheet function. Then we will adjust our getCompanyConfig function to only handle validating and returning the configuration object.
```javascript
async function getTimesheet(userId, date, offset) {
  try {
    let user = await User.findById(userId, { include: [models.Company] });
    let config = await getCompanyConfig(user.Company.name);
    let timesheetObject = await getTimesheetObject(config, user, date, offset);
    let buffer = await executeTimesheetRequest(timesheetObject);
    return buffer;
  } catch (e) {
    console.error('Error Retreiving Timesheet', e);
    throw e;
  }
}
```

``` javascript
async function getCompanyConfig(companyName) {
  try {
    if (!exportConfiguration.templates[companyName]) {
      throw new Error('No Template Available for Company: ' + u.Company.name);
    }
    return exportConfiguration.config;
  } catch (e) {
    throw e;
  }
}
```
There we have it.

The Async / Await functionality has made this small portion of code WAY more readable and way more maintainable.  I can now have functions handle specific actions and I can successfully handle the behaviour in a way that feels more synchronous but while not blocking the event loop.

Hopefully this has inspired you to give it a go!

