+++
author = "James D Hughes"
categories = ["ReactJS", "Database", "Offline Storage", "IndexedDB", "idb", "JavaScript"]
date = 2019-01-09T13:07:23Z
description = ""
draft = false
image = "/images/2019/01/DSC_0992-1.JPG"
slug = "indexeddb-react"
tags = ["ReactJS", "Database", "Offline Storage", "IndexedDB", "idb", "JavaScript"]
title = "IndexedDB + React for Offline Storage"

+++


Offline data is annoying.

I'm used to storing data in localstorage which is okay... sometimes.  When you have a small amount of data to store such as user settings or persisting filters between views then this is a decent enough solution. I was working on a project recently that had a wild requirement appear where we needed to store upwards of 10k items offline.  Needless to say, this put performance problems in our application.

This was primarily because of the time it took to serialize and deserialize (stringify and parse) the payload to be used.  We were using some observables to watch our collection and any time there was a change, we persisted the whole collection to localstorage - sounds not bad right? For typical usage (a thousand items or so) it was just fine. When we had upwards of 10k items in our collection - we started experiencing lag.  Approximately 1.5-3 seconds worth of lag each time we changed an item in the collection.  Needless to say, this was unacceptable.

We started researching different usages of IndexedDB. The team was pretty comfortable using localstorage and structured our data to use key/value stores in that way. We didn't _really_ understand the true power of IndexedDB as an awesomex data store in the browser.

So - I'm going to walk you through my PoC to show the awesome performance gains we would realize by using IndexedDB in our app to it's fullest.  As usual, the code is available on my [Github](https://github.com/Daimonos/react-indexeddb-demo)

We'll start it off using create-react-app. (I ejected the app before putting it in my GitHub because reasons) I'm not going to show you how to do that. There's loads of getting started tutorials for that.

`create-react-app my-app`

First thing I'm going to do, the meat and potatoes of this post, is to install the `idb` package which wraps the IndexedDB API with promises - making life so much easier.

`npm install --save idb`

Then I'm going to make a service in `src/services/DBService.js`. In this, I'm going to make sure to set some constants for setting up my database and versioning it, and for demo purposes we're going to set up some predictable placeholder data for the purposes of showcasing the awesome IndexedDB functionality.

```javascript
//src/services/DBService.js
import * as idb from 'idb';

const DATABASE_NAME = 'SERVICE_ORDERS';
const DATABASE_VERSION = 2;
const db = idb.default;

export const BUCKETS = ['Bucket01', 'Bucket02']
export const BUSINESS_UNITS = ['BU1', 'BU2', 'BU3', 'BU4', 'BU5']
export const STATUS = ['In Progress', 'Completed', 'Pending']
```

Next we declare the promise for creating the database. You can read the docs on how to work this [here](https://developers.google.com/web/ilt/pwa/working-with-indexeddb) and [here](https://developers.google.com/web/ilt/pwa/lab-indexeddb) [] but I'm not going to go into the nuts and bolts of indexed db here.

```javascript
...

/**
 * Initialize the IndexedDB.
 * see https://developers.google.com/web/ilt/pwa/lab-indexeddb
 * for information as to why we use switch w/o breaks for migrations.
 * add do the database version and add a switch case each time you need to
 * change migrations
 */
const dbPromise = db.open(DATABASE_NAME, DATABASE_VERSION, function (upgradeDb) {
  /* tslint:disable */
  switch (upgradeDb.oldVersion) {
    case 0:
    // a placeholder case so that the switch block will
    // execute when the database is first created
    // (oldVersion is 0)
    case 1:
      upgradeDb.createObjectStore('ServiceOrders', { keyPath: 'id' });
      const tx = upgradeDb.transaction.objectStore('ServiceOrders', 'readwrite')
      tx.createIndex('Bucket', 'bucket')
      tx.createIndex('businessUnit', 'businessUnit')
      tx.createIndex('status', 'status')
      tx.createIndex('BucketBusinessUnitStatus', ['bucket', 'businessUnit', 'status'])
      for (let i = 0; i < 100000; i++) {
        tx.put({
          id: i,
          bucket: BUCKETS[Math.floor(Math.random() * BUCKETS.length)],
          businessUnit: BUSINESS_UNITS[Math.floor(Math.random() * BUSINESS_UNITS.length)],
          status: STATUS[Math.floor(Math.random() * STATUS.length)]
        });
      }
  }
});
```

Long story short -this piece of code initializes our database if it doesn't exist (and at it's current version) and we create an object store, throw on some indexes, and then fake in 100k records (I went a bit overboard just to show how slick it is). Finally we can make a class and expose some simple CRUD functions with awful error handling.

```javascript
import * as idb from 'idb';

const DATABASE_NAME = 'SERVICE_ORDERS';
const DATABASE_VERSION = 2;
const db = idb.default;

export const BUCKETS = ['Bucket01', 'Bucket02']
export const BUSINESS_UNITS = ['BU1', 'BU2', 'BU3', 'BU4', 'BU5']
export const STATUS = ['In Progress', 'Completed', 'Pending']
/**
 * Initialize the IndexedDB.
 * see https://developers.google.com/web/ilt/pwa/lab-indexeddb
 * for information as to why we use switch w/o breaks for migrations.
 * add do the database version and add a switch case each time you need to
 * change migrations
 */
const dbPromise = db.open(DATABASE_NAME, DATABASE_VERSION, function (upgradeDb) {
  /* tslint:disable */
  switch (upgradeDb.oldVersion) {
    case 0:
    // a placeholder case so that the switch block will
    // execute when the database is first created
    // (oldVersion is 0)
    case 1:
      upgradeDb.createObjectStore('ServiceOrders', { keyPath: 'id' });
      const tx = upgradeDb.transaction.objectStore('ServiceOrders', 'readwrite')
      tx.createIndex('Bucket', 'bucket')
      tx.createIndex('businessUnit', 'businessUnit')
      tx.createIndex('status', 'status')
      tx.createIndex('BucketBusinessUnitStatus', ['bucket', 'businessUnit', 'status'])
      for (let i = 0; i < 100000; i++) {
        tx.put({
          id: i,
          bucket: BUCKETS[Math.floor(Math.random() * BUCKETS.length)],
          businessUnit: BUSINESS_UNITS[Math.floor(Math.random() * BUSINESS_UNITS.length)],
          status: STATUS[Math.floor(Math.random() * STATUS.length)]
        });
      }
  }
});

class DBService {

  get(tablespace, key) {
    return dbPromise.then(db => {
      return db.transaction(tablespace).objectStore(tablespace).get(key);
    }).catch(error => {
      // Do something?
    });
  }

  getAll(tablespace, indexName, index = []) {
    return dbPromise.then(db => {
      return db.transaction(tablespace).objectStore(tablespace).index(indexName).getAll(index);
    }).catch(error => {
      // Do something?
    });
  }

  put(tablespace, object, key = null) {
    return dbPromise.then(db => {
      if (key) {
        return db.transaction(tablespace, 'readwrite').objectStore(tablespace).put(object, key);
      }
      return db.transaction(tablespace, 'readwrite').objectStore(tablespace).put(object);
    }).catch(error => {
      // Do something?
    });
  }

  delete(tablespace, key) {
    return dbPromise.then(db => {
      return db.transaction(tablespace, 'readwrite').objectStore(tablespace).delete(key);
    }).catch(error => {
      // Do something?
    });
  }

  deleteAll(tablespace) {
    return dbPromise.then(db => {
      return db.transaction(tablespace, 'readwrite').objectStore(tablespace).clear();
    }).catch(error => {
      // Do something?
    });
  }
}

export const Service = new DBService()
```

Most of this is pretty straight forward. get, getAll, put, delete, deleteAll. There's not much else to say here!

I suppose we should load up our service and run it then! We'll set up a generic component for browsing and set up `react-router-dom` since it's super useful and `@material-ui` because it's pretty.

`npm install --save react-router-dom @material-ui/core @material-ui/icons`

Replace your `src/index.js` with the following:

```javascript
//src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom'
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';

ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>, document.getElementById('root'));

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: http://bit.ly/CRA-PWA
serviceWorker.unregister();
```

Then create a new component for displaying our data in a `ServiceOrderList` component

```javascript
// src/components/service-orders/service-order-list.js
import * as React from 'react';
import { Service } from '../../services/DBService';
import { Link } from 'react-router-dom';
import { CircularProgress, Table, TableHead, TableCell, TableRow, TableBody, AppBar, Toolbar, Typography } from '@material-ui/core';
import './service-orders-list.css';

export default class ServiceOrdersListComponent extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      bucket: this.props.match.params.bucket,
      businessUnit: this.props.match.params.businessUnit,
      status: this.props.match.params.status,
      isLoading: true
    }
    this._getOrders()
  }

  _getOrders() {
    let idx = [this.state.bucket, this.state.businessUnit, this.state.status]
    Service.getAll('ServiceOrders', 'BucketBusinessUnitStatus', idx).then(orders => {
      this.setState({ orders: orders, isLoading: false })
    });
  }

  render() {
    let { bucket, businessUnit, status, isLoading } = this.state
    let { orders } = this.state
    return <div>
      <AppBar position="static">
        <Toolbar>
          <Typography variant="h6">
            {bucket} - {businessUnit} - {status}
          </Typography>
        </Toolbar>
      </AppBar>

      {orders && <p>Viewing {orders.length} orders</p>}
      {isLoading && <CircularProgress />}
      {
        !isLoading && <Table>
          <TableHead>
            <TableRow>
              <TableCell className="sticky-header">ID</TableCell>
              <TableCell className="sticky-header">Bucket</TableCell>
              <TableCell className="sticky-header">Business Unit</TableCell>
              <TableCell className="sticky-header">Status</TableCell>
            </TableRow>
          </TableHead>
          <TableBody style={{ overflowY: 'scroll' }}>
            {orders && orders.map(o => <TableRow key={o.id}>
              <TableCell>
                <Link to={'/serviceOrders/' + o.id}>{o.id}</Link></TableCell>
              <TableCell>{o.bucket}</TableCell>
              <TableCell>{o.businessUnit}</TableCell>
              <TableCell>{o.status}</TableCell>
            </TableRow>)}
          </TableBody>
        </Table>
      }
    </div >;
  }
}
```

I had a .css file in there too:

```css
// src/components/service-orders/service-orders-list.css
.sticky-header{
  position:sticky;
  position:-webkit-sticky;
  background-color:#fff;
  top:0;
  text-align:left;
}
```

A `HomeComponent` to help us navigate:

```javascript
//src/components/home/home-component.js
import * as React from 'react';
import { Link } from 'react-router-dom';
import { BUCKETS, BUSINESS_UNITS, STATUS } from '../../services/DBService';

export default class HomeComponent extends React.Component {
  render() {
    let refs = []
    BUCKETS.forEach(bu => {
      BUSINESS_UNITS.forEach(b => {
        STATUS.forEach(s => {
          let path = "/serviceOrders/bucket/" + bu + "/businessUnit/" + b + "/status/" + s
          refs.push(<Link to={path}>{path}</Link>)
        })
      })
    })
    return <div>
      <h3>Some Routes</h3>
      {refs.map((r, i) => <li key={i}>{r}</li>)}
    </div>
  }
}
```

A `ServiceOrderDetail` component for more fun:

```javascript
//src/components/service-orders/service-order-detail.js
import * as React from 'react';
import { Service } from '../../services/DBService';
import { AppBar, Toolbar, Typography, IconButton } from '@material-ui/core';
import KeyboardArrowLeftIcon from '@material-ui/icons/KeyboardArrowLeft';

export default class ServiceOrderDetailComponent extends React.Component {
  constructor(props) {
    super(props)
    this.state = { id: parseInt(this.props.match.params.id) }
    this._getOrder()
  }

  _getOrder() {
    Service.get('ServiceOrders', this.state.id).then(o => {
      this.setState({ order: o })
    })
  }

  _goBack = () => {
    this.props.history.goBack()
  }

  render() {
    const { order } = this.state;
    return <div>
      <AppBar position="static">
        <Toolbar>
          <IconButton onClick={this._goBack}>
            <KeyboardArrowLeftIcon />
          </IconButton>
          <Typography variant="h6">
            Order Details
          </Typography>
        </Toolbar>
      </AppBar>
      <pre>{JSON.stringify(order, null, 2)}</pre>
    </div>
  }
}
```

A `NotFound` component for handling routing `NotFound` issues

```javascript
//src/components/errors/not-found.js
import * as React from 'react';

export default class NotFoundComponent extends React.Component {
    render() {
        return <h1>Not Found!</h1>
    }
}
```

Finally in src/App.js

```javascript
import React, { Component } from 'react';
import './App.css';
import { Route, Switch } from 'react-router-dom';
import ServiceOrdersListComponent from './components/service-orders/service-orders-list';
import HomeComponent from './components/home/home-component';
import ServiceOrderDetailComponent from './components/service-orders/service-order-detail';
import NotFoundComponent from './components/errors/not-found';
class App extends Component {
  render() {
    return (
      <div>
        <Switch>
          <Route exact path="/" component={HomeComponent} />
          <Route path="/serviceOrders/bucket/:bucket/businessUnit/:businessUnit/status/:status" component={ServiceOrdersListComponent} />
          <Route exact path="/serviceOrders/:id" component={ServiceOrderDetailComponent} />
          <Route component={NotFoundComponent} />
        </Switch>
      </div >

    );
  }
}

export default App;
```

Go ahead and start up the app now. The first load of your route will take a little bit as the database gets seeded, but you will see how quickly it loads (and renders) 3k records (on average per bucket)

The list is neat, but if you look in your devtools under application and IndexedDB you can see our table of data and all the different indexes we created to help us in our application!

{{< figure src="/content/images/2019/01/Screen-Shot-2019-01-08-at-10.19.49-PM.png" caption="IndexedDB Table with Indexes" >}}

By default our keyPath (and thusly main table) is indexed by id. We built some single key indexes and one compound key index which makes our data access a breeze! We can use this compound index to filter our data down by each of our keys, but in that specific order. For instance, we search for a specific Bucket first:

{{< figure src="/content/images/2019/01/Screen-Shot-2019-01-08-at-10.22.17-PM.png" caption="Browse by Bucket02 first" >}}

Then we browse by a specific business unit as well:

{{< figure src="/content/images/2019/01/Screen-Shot-2019-01-08-at-10.22.36-PM.png" caption="Browsing byBucket and Business Unit" >}}

Pretty neat, hey? Having this functionality provides us with some significant gains over a naive key-value store. These include:

1. We are now able to reduce our API's need to understand how our data will be organized on the client. We can just toss down all the orders and then query it like a regular old database client side.
2. Our client code is simplified as we won't need to do any map-reducing on big collections - the browser's API handles it for us.
3. When we want to update a record, we can serialize and deserialize a single record instead of the entire collection. Atomic updates are good updates.

This refactor is still being considered for the project as it's a fair bit of work to introduce a sweeping change such as this. However hopefully you read this before you got started on your projects and you have a new idea to try out before making a decision for your offline storage :)

Some other optimizations that would be good to do include paginating the data so that you're not rendering all 3k items on every load.  I figure that's outside the scope of this indexed db discussion though.

Enjoi :)

