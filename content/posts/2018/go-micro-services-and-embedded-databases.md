+++
author = "James D Hughes"
categories = ["golang"]
date = 2018-09-19T06:30:15Z
description = ""
draft = false
slug = "go-micro-services-and-embedded-databases"
tags = ["golang"]
title = "Go - Micro-services and Embedded Databases [Part 1]"

+++


This is fun!

FYI - If you're looking for Part 2, it's [here](https://blog.jimdhughes.com/go-micro-services-and-embedded-databases-part-2/)[.]

---

I've been meaning to learn a few of the things I'm going to go over in this post.  Firstly, I want to make an actual useful application using Go. I want it to be run as a microservice with it's own database.  Instead of putting it in a docker container, I want to just use an embedded database that the executable can use.

We're going to build a user authentication and token validation app.  It'll use REST for communication (gRPC may come later) and will store users using BoltDB.  The server will allow registration, logging in, will issue tokens and will validate tokens.

The endpoints for our app will be:

POST /login - Takes an email and password and attempts to log a user in

POST /register - Takes an email and password an attempts to create a new user

POST /validateToken - checks a supplied token. If it's valid it sends back the information of the authenticated user.

We're going to use a couple golang libraries that seem pretty popular. For routing, we will use [gorilla/mux](https://github.com/gorilla/mux), for the database we will use [boltdb/bolt](https://github.com/boltdb/bolt) and for password hashing we will use [golang.org/x/crypto/bcrypt](https://godoc.org/golang.org/x/crypto/bcrypt).  Go ahead and 'go get' them and we'll dive right in.

I start every go project by making a main.go and printing a simple Hello, World! so let's start there again

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello, World!")
}
```

It's kind of hard to figure out where to start next.  I'm going to elect to start by making my model.  I'm going to make a new file called `model.go` and it'll look like so:

```go
package main

// User structure for accounts
type User struct {
	ID       string `json:"id"`
	Email    string `json:"email"`
	Password string `json:"password"`
}

```

Since we're going to need to encrypt and check password hashes, we may as well just attach it the the User struct. Let's do that next.

```go

// ...

// HashPassword sets the users password to the bcrypt hashed version of said password
func (u *User) HashPassword() error {
	password, err := bcrypt.GenerateFromPassword([]byte(u.Password), 14)
	if err != nil {
		return err
	}
	u.Password = string(password)
	return nil
}

// CheckPassword compares a plain text password against the Hashed Password of the user
func (u *User) CheckPassword(password string) error {
	err := bcrypt.CompareHashAndPassword([]byte(u.Password), []byte(password))
	if err != nil {
		return err
	}
	return nil
}

We now have some testable code! For fun let's write a test for the password hasing!

```go
package main

import (
	"testing"
)

func TestUserPassword(t *testing.T) {
	password := "testpassword"
	u := User{
		Email:    "testEmail@domain.com",
		Password: password,
	}
	u.HashPassword()
	if u.Password == password {
		// Password failed to hash
		t.Fail()
	}
}

func TestUserPasswordCompareCorrectPassword(t *testing.T) {
	password := "testpassword"
	u := User{
		Email:    "testEmail@domain.com",
		Password: password,
	}

	u.HashPassword()
	err := u.CheckPassword(password)
	if err != nil {
		t.Fatal(err)
	}
}

func TestUserPasswordCompareIncorrectPassword(t *testing.T) {
	password := "testpassword"
	u := User{
		Email:    "testEmail@domain.com",
		Password: password,
	}
	password = "nope!"
	u.HashPassword()
	err := u.CheckPassword(password)
	if err == nil {
		t.Fatal(err)
	}
}


```

Run these tests using `go test`. Everything should pass and now we can move on to making some actual functionality!

Next I want to make some placeholder routes to handle the requests we want to handle. To do this, let's make a new file called `router.go`

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/gorilla/mux"
)

func NewRouter() *mux.Router {
	r := mux.NewRouter()
	r.HandleFunc("/login", HandleLogin).Methods("POST")
	r.HandleFunc("/register", HandleRegistration).Methods("POST")
	r.HandleFunc("/validateToken", HandleValidateToken).Methods("POST")
	return r
}

func HandleLogin(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusInternalServerError)
	fmt.Fprintf(w, "Not Yet Implemented")
}

func HandleRegistration(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusInternalServerError)
	fmt.Fprintf(w, "Not Yet Implemented")
}

func HandleValidateToken(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusInternalServerError)
	fmt.Fprintf(w, "Not Yet Implemented")
}
```

Then we update our `main.go` file to initialize the router and serve up our routes that will all issue error responses!

```go

package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	fmt.Println("Hello, World!")
	InitializeHttpServer()
}

func InitializeHttpServer() {
	r := NewRouter()
	log.Panic(http.ListenAndServe(":8080", r))
}


`

I like to use postman to test my apis but you can curl it if you want.  Go ahead and send some POST requests to our endpoints and you'll get a simple 500 response with "Not Yet Implemented" as the response.

Next we can either implement a service to handle these requests or we can implement the database interface.  I'm going to elect to implement the database functionality because I'm a bit of a database nerd.  BoltDB is a key/value store which is really all that we need for implementing an authentication database in a microservice.  I'm not going to do anything fancy such as allowing token invalidation etc so this should be pretty simple! Let's make a new file called `db.go`

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"strconv"

	"github.com/boltdb/bolt"
)

// Interface for the DB Client
type IDBClient interface {
	Initialize(filepath string)
	Open()
	Close()
	CreateUser(u User) (bool, error)
	Login(email, password string) (string, error)
	CheckUserIsNew(email string) (bool, error)
}
// Struct to handle the DB Connection
type DBClient struct {
	client   *bolt.DB
	filepath string
}

// The DB Client for the application
var DB IDBClient

const (
	usersBucketName string = "Users"
)

func (db *DBClient) Open() {

}
func (db *DBClient) Initialize(filepath string) {

}
func (db *DBClient) Close() {

}
func (db *DBClient) CreateUser(u User) (bool, error) {
  return false, fmt.Errorf("Not Implemented")
}
func (db *DBClient) Login(email, password string) (string, error) {
  return "", fmt.Errorf("Not Implemented")
}
func (db *DBClient) CheckUserIsNew(email string) (bool, error) {
    return false, fmt.Errorf("Not Implemented")
}
```

I'm no go pro but this is how I understand that go apps are organized.  My OOP career has drilled into my head that you should always program against an interface so when they exist, I use them.

The DBClient is sort of like a class. We're going to use it to store the bolt client, the location of the filepath, and then we are going to declare methods for it by using the syntax that is seen below.  The `var DB IDBClient` line helps us to initialize the database handler and makes sure that our initialization of DBClient has all the functions as defined by the IDBClient interface.  Clear as mud? Let's do it then!

First we may as well handle the easy stuff: Initialize, Open, and Close.

```go
func (db *DBClient) Initialize(filepath string) {
	db.filepath = filepath
	db.Open()
	defer db.Close()
	err := db.client.Update(func(txn *bolt.Tx) error {
		// Initialize Users Bucket
		_, err := txn.CreateBucketIfNotExists([]byte(usersBucketName))
		if err != nil {
			return err
		}
		return nil
	})
	if err != nil {
		log.Panic(err)
	}
}

func (db *DBClient) Open() {
	if db.filepath == "" {
		log.Fatal("Filepath required for Database")
	}
	d, err := bolt.Open(db.filepath, 0600, nil)
	if err != nil {
		log.Panic(err)
	}
	db.client = d
}

func (db *DBClient) Close() {
	db.client.Close()
}
```

The Initialize function takes a filepath. This would be the same as taking in an SQL connection string in any other app.  This makes it way easier for us to set up our environment for testing and for production.  We can set up and tear down test databases while leaving our production an development databases alone.  We then set up the buckets that our app is going to need (which is just a `Users` bucket for now).

The Open function does just that.  Opens the database and applies it to it's client parameter.   The close does just what you'd expect a close function to do as well.

We may as well write the functional code now.  We'll start with `CreateUser` and `CheckUserIsNew`

```go
func (db *DBClient) CreateUser(u User) (bool, error) {
	isNew, err := db.CheckUserIsNew(u.Email)
	if err != nil || isNew == false {
		return false, err
	}
	db.Open()
	defer db.Close()
	err = db.client.Update(func(txn *bolt.Tx) error {
		b := txn.Bucket([]byte(usersBucketName))
		id, err := b.NextSequence()
		if err != nil {
			return err
		}
		u.ID = strconv.Itoa(int(id))
		u.HashPassword()
		userBytes, err := json.Marshal(u)
		if err != nil {
			return err
		}
		err = b.Put([]byte(u.Email), userBytes)
		if err != nil {
			return err
		}
		return nil
	})
	if err != nil {
		return false, nil
	}
	return true, nil
}

func (db *DBClient) CheckUserIsNew(email string) (bool, error) {
	db.Open()
	defer db.Close()
	err := db.client.View(func(tx *bolt.Tx) error {
		b := tx.Bucket([]byte(usersBucketName))
		userBytes := b.Get([]byte(email))
		if userBytes != nil {
			return fmt.Errorf("User with email %s already exists", email)
		}
		return nil
	})
	if err != nil {
		return false, err
	}
	return true, nil
}
```

`CheckUserIsNew` is just a helper function to determine if our user exists in the database already or not.  This is helpful as we don't really have key constraints in the embedded database.  We need to ensure that we're not just re-registering users when a registration request comes in.  The `CreateUser` calls this as soon as possible and if a user exists, it returns an error. The `CreateUser` will then return this error to the calling function.

We can write some tests for this now! in `db_test.go` let's set up some cases

```go
package main

import (
	"os"
	"testing"
)

func TestMain(m *testing.M) {
	DB = &DBClient{}
	DB.Initialize("./bolt-test.db")
	retCode := m.Run()
	os.Remove("./bolt-test.db")
	os.Exit(retCode)
}

func TestCheckUserShouldNotReturnError(t *testing.T) {
	email := "test@domain.com"
	isNew, err := DB.CheckUserIsNew(email)
	if err != nil {
		t.Error("An Error was thrown that should not have been")
	}
	if isNew == false {
		t.Error("User shouldn't Exist but does")
	}
}

func TestCreateUser(t *testing.T) {
	u := User{
		Email:    "test@domain.com",
		Password: "password",
	}
	result, err := DB.CreateUser(u)
	if err != nil {
		t.Error(err)
	}
	if result == false {
		t.Error("User not created")
	}
}

func TestCreateUserWithSameEmailShouldFail(t *testing.T) {
	u := User{
		Email:    "test@domain.com",
		Password: "password",
	}
	result, err := DB.CreateUser(u)
	if err == nil {
		t.Error(err)
	}
	if result == true {
		t.Error("User not created")
	}
}

func TestCheckUserShouldReturnError(t *testing.T) {
	email := "test@domain.com"
	isNew, err := DB.CheckUserIsNew(email)
	if isNew == true {
		t.Error("User Should Exist but doesn't")
	}
	if err == nil {
		t.Error("Error should have been thrown")
	}
}

```

We first make sure that our `CheckUserExists` function returns false. We then test some `CreateUser` functionality and then finally make sure that our `CheckUserExists` returns an error as the account does already exist. Follow? Sweet! Let's move on to the `Login` function!

```go
// db.go
// ...
func (db *DBClient) Login(email, password string) (string, error) {
	db.Open()
	defer db.Close()
	err := db.client.View(func(tx *bolt.Tx) error {
		b := tx.Bucket([]byte(usersBucketName))
		userBytes := b.Get([]byte(email))
		u := User{}
		json.Unmarshal(userBytes, &u)
		err := u.CheckPassword(password)
		if err != nil {
			log.Printf("ERROR: %s", err)
			log.Printf("Invalid Login Attempt for User: %s", email)
			return fmt.Errorf("Invalid Email/Password Combnation")
		}
		return nil
	})
	if err != nil {
		return "nope", err
	}
	return "yep", nil
}
```

Here we take an email and password, load up the user from the database, check if the password is the valid hash, and then return a string response.  We will replace this "nope" and "yep" with a token later but for now we will just use these.

And again let's make sure that we have some test cases for `Login`:

```go
// db_test.go
// ...
func TestValidLogin(t *testing.T) {
	_, err := DB.Login("test@domain.com", "password")
	if err != nil {
		t.Error(err)
	}
}

func TestInvalidLogin(t *testing.T) {
	_, err := DB.Login("test@domain.com", "wrongpassword")
	if err == nil {
		t.Error("The password should have been invalid")
	}
}
```

Check our code again using `go test` and we should pass everything.

---

# A Note on Tests

This isn't really TDD.  This is what we call lazy testing. Typically you'd write tests to test your functionality and THEN implement the code.  This allows you to come up with as many different scenarios as possible and then write resilient code.  That's outside the scope of this little learning activity but some tests are better than no tests.

---

Now that we have our core functionality in controllers we need to expose it externally.  We will do this by going back to our `router.go` file and adding some real functionality as opposed to just throwing random errors :)

Logically, we'll implement the registration route first:

```go

func HandleRegistration(w http.ResponseWriter, r *http.Request) {
	var user User
	decoder := json.NewDecoder(r.Body)
	err := decoder.Decode(&user)
	if err != nil {
		WriteError(w, http.StatusBadRequest, fmt.Errorf("Error Decoding User"))
	}
	success, err := DB.CreateUser(user)
	if err != nil {
		WriteError(w, http.StatusBadRequest, err)
        return
	}
	fmt.Fprintf(w, "Status: %t", success)
}

func WriteError(w http.ResponseWriter, statusCode int, err error) {
	w.WriteHeader(statusCode)
	fmt.Fprintf(w, err.Error())
}
```

You'll notice that I added a `WriteError` helper function since it's easier than repeating this code over and over again.  Passing the actual error through to the client is PROBABLY not a great idea in a production environment though.

Before we can successfully build and run this, we will need to initialize our database.  In `main.go` we'll add a helper function and call it before we initialize the Router.

```go

package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	fmt.Println("Hello, World!")
	InitializeDatabase()
	InitializeHttpServer()
}

func InitializeHttpServer() {
	r := NewRouter()
	log.Panic(http.ListenAndServe(":8080", r))
}

func InitializeDatabase() {
	DB = &DBClient{}
	DB.Initialize("bolt.db")
}
```

Now - Build it and run! `go build` and `./whatever-your-foldername-is` if you're on *nix or `./whatever-your-foldername-is.exe` if you're on windows!  I use Postman, you can use CURL if you like, but no matter what let's send a registration request to `http://localhost:8080/register` with a payload of

```json
{
  "email":"someemail@domain.com",
  "password":"password"
}
```

You should get a response of: `Status: true`. Success! We have registered a user!

Now we should allow this user to login. We'll implement the `HandleLogin` handler next.

```go
func HandleLogin(w http.ResponseWriter, r *http.Request) {
	var user User
	decoder := json.NewDecoder(r.Body)
	err := decoder.Decode(&user)
	if err != nil {
		WriteError(w, http.StatusBadRequest, fmt.Errorf("Error Decoding User"))
	}
	status, err := DB.Login(user.Email, user.Password)
	if err != nil {
		WriteError(w, http.StatusBadRequest, err)
        return
	}
	fmt.Fprintf(w, status)
}
```

Buld and run again! Send a POST to `http://localhost:8080/login` with the payload

```json
{
  "email":"someemail@domain.com",
  "password":"password"
}
```

And you should get the message: `yep`.

Anyways, this post has gotten REALLY long.  So check back for part 2 where we implement the token service, the `tokenValidation` endpoints, and make some more interesting responses that can be digested better by a calling service.

Keep Hacking!



