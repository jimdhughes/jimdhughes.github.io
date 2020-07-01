+++
author = "James D Hughes"
categories = ["golang"]
date = 2018-09-24T09:51:32Z
description = ""
draft = false
slug = "go-micro-services-and-embedded-databases-part-2"
tags = ["golang"]
title = "Go - Micro-services and Embedded Databases [Part 2]"

+++


If you haven't already, check out [Part 1](https://blog.jimdhughes.com/go-micro-services-and-embedded-databases/)[!]

First things first - let's make these responses something machine readable instead of just some plain text! That's a nice and easy way to dive back into the code!

We are going to create 2 new structs in the `router.go` file to declare how the app is going to return Errors and Standard responses.

```go
package main

// ...

type ApiErrorResponse struct {
	Error string `json:"error"`
}

type ApiStandardResponse struct {
	Payload interface{} `json:"payload"`
}

```

This is menial, but it helps our responses have a bit more structure.  The first simply returns an error message as a string value. We will assume that our handlers will set the http response status accordingly.  The `ApiStandardResponse` is pretty well the same except we have an `interface{}` type on the payload.  This is so that we can return any type of data.   The response will be dependent on what is being returned by the handler.  Next let's update our `WriteError` function to use this new struct

```go
func WriteError(w http.ResponseWriter, statusCode int, err error) {
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    w.WriteHeader(statusCode)
	response := ApiErrorResponse{Error: err.Error()}
	json.NewEncoder(w).Encode(response)
}

```

Here we take the error, make a new response object and then encode it as a json format and send it back to the requester. Easy as pie!

Taking a leaf out of our `WriteError` book, let's implement a `WriteResponse` function to handle all the good responses! It's going to look a lot the same as the `WriteError` function.

```go
func WriteResponse(w http.ResponseWriter, statusCode int, payload interface{}) {
    w.Header().Set("Content-Type", "application/json; charset=UTF-8")
    w.WriteHeader(statusCode)
	response := ApiStandardResponse{Payload: payload}
	json.NewEncoder(w).Encode(response)
}

```

Again, pretty straight forward.  You should update all of your response lines that were previously just fmt.Fprintf(w, ...) with the WriteResponse function.  Hint - they are in the `HandleRegistration` and `HandleLogin` functions!

Now on to some real functionality.  Tokens!

We're going to return a token from our `Login` function. This is going to contain some user information such as their Email and ID. In a real app you could return some permission grants for your apps or user roles, but we're not going to be that sophisticated here.  Make a new file called `tokenHandler.go`

```go
package main

const (
	secretKey = "asupersecretekeythatshouldntbehardcodedhere"
)

type ITokenService interface {
	CreateToken(user User) (string, error)
	ValidateToken(token string) (interface{}, error)
}

type TokenService struct{}

var TS ITokenService

func (t *TokenService) CreateToken(user User) (string, error) {
	return "", nil
}

func (t *TokenService) ValidateToken(token string) (interface{}, error) {
	return User{}, nil
}

```

This should be all we need as a scaffold.  We create an interface as the contract for the service and then implement a struct that will essentially implement the ITokenService interface.  We have method stubs to handle the creation of a token from a user and the validation of a token.

In contrast to my last post, we're going to do our development TDD style this time around.

Firstly we need to initialize our TokenService in our TestMain function (found in db.go)

```go
func TestMain(m *testing.M) {
	DB = &DBClient{}
	TS = &TokenService{}
	DB.Initialize("./bolt-test.db")
	retCode := m.Run()
	os.Remove("./bolt-test.db")
	os.Exit(retCode)
}
```

Next comes the test file for the Token Service `tokenService_test.go`

```go
package main

import (
	"testing"
)

func TestCreateTokenFromNotValidUser(t *testing.T) {
	user := User{}
	token, err := TS.CreateToken(user)
	if err == nil {
		t.Errorf("Token should not be created for an empty user")
		return
	}
	if token != "" {
		t.Errorf("Error should have been thrown as token cannot be created")
		return
	}
}

func TestCreateTokenFromValidUser(t *testing.T) {
	user := User{Email: "tokentester@domain.com", Password: "password"}
	DB.CreateUser(user) // we test this elsewhere so assume it works. see db_test.go
    // ... I'm Stuck!

}

```

Yep. I'm stuck!

This is why TDD is good. It forces you to develop some better code.  The original flow that I had planned was that the Login function would just return a string value (the token) which isn't necessarily wrong, however for logical testing it makes life a bit harder than it has to be!

We have code to get a user by email twice in this application already. Once in the `Login` function and once in the `CheckUserExists` function.  So let's refactor this repeated code into a new function called `GetUserByEmail` in the `db.go` file.

```go
package main

type IDBClient interface {
	Initialize(filepath string)
	Open()
	Close()
	CreateUser(u User) (bool, error)
	Login(email, password string) (string, error)
	CheckUserIsNew(email string) (bool, error)
    // Add to the interface
	GetUserByEmail(email string) (User, error)
}

// ...

func (db *DBClient) GetUserByEmail(email string) (User, error) {
	db.Open()
	defer db.Close()
	user := User{}
	err := db.client.View(func(tx *bolt.Tx) error {
		b := tx.Bucket([]byte(usersBucketName))
		userBytes := b.Get([]byte(email))
		if userBytes == nil {
			return nil
		}
		err := json.Unmarshal(userBytes, &user)
		if err != nil {
			return err
		}
		return nil
	})
	if err != nil {
		return User{}, err
	}
	return user, nil
}

```

And now that we have our helper function we can rebuild our `CheckUserExists` and `Login` functions!

```go
func (db *DBClient) CheckUserIsNew(email string) (bool, error) {
	user, err := db.GetUserByEmail(email)
	if err != nil {
		log.Printf("%s", err)
		return false, err
	}
	if user.ID != "" {
		return false, fmt.Errorf("User Exists with email %s", email)
	}
	return true, nil
}

func (db *DBClient) Login(email, password string) (string, error) {
	user, err := db.GetUserByEmail(email)
	if err != nil {
		return "Nope", err
	}
	err = user.CheckPassword(password)
	if err != nil {
		return "nope", err
	}
	return "yep", nil
}

func (db *DBClient) CreateUser(u User) (bool, error) {
	isNew, err := db.CheckUserIsNew(u.Email)
	if err != nil || isNew == false {
		return false, err
	}
	db.Open()
	defer db.Close()
	err = db.client.Update(func(txn *bolt.Tx) error {
        // ...
}
```

With this new functionality, we have to make some changes to our original code.  We don't need to open the database in the `Login` function or the `CheckUserExists` function.  This also means though that when we get to our `CreateUser` we need to move the `db.Open()` and `defer db.Close()` calls to after the `CheckUserIsNew` function as we end up opening a database connection again.

We should test the new `GetUserByEmail` function so add a new test in `db_test.go`

```go

// ...

func TestGetUserByEmailInvalidEmail(t *testing.T) {
	user, err := DB.GetUserByEmail("invalidemail@somedomain.com")
	if err != nil {
		t.Errorf(err.Error())
	}
	if user.ID != "" {
		t.Errorf("No user should have been returned")
	}
}

func TestGetUserByEmailValidEmail(t *testing.T) {
	user, err := DB.GetUserByEmail("test@domain.com")
	if err != nil {
		t.Error(err.Error())
	}
	if user.Email == "" {
		t.Error("Expected to get user but got an empty user")
	}
	if user.Email != "test@domain.com" {
		t.Errorf("Returned a user other than expected")
	}
}
```

That's much better! Run our tests again to make sure we haven't broken any functionality using `go test`.  The only test that should be failing is the one we were previously stuck on! This means that we haven't broken the system! Woot!! We don't have to stay late tonight!

**Full Disclosure: My tests were failing when I first wrote the changes. I got to fix them before publishing. Tests are good.**

Now that we've made those changes, we can implement that test and continue on writing some more!

```go
// ...
func TestCreateTokenFromValidUser(t *testing.T) {
	user := User{Email: "tokentester@domain.com", Password: "password"}
	DB.CreateUser(user)                     // we test this elsewhere so assume it works. see db_test.go
	user, _ = DB.GetUserByEmail(user.Email) //we can ignore the error as we test this elsewhere
	token, err := TS.CreateToken(user)
	if err == nil {
		fmt.Errorf("Token could not be created for user: %s", err)
	}
	if token == "" {
		fmt.Errorf("Empty token was returned. Expected a token as a string")
	}
}

```

Let's run our tests!

They better all fail.  If they don't then my logic might be worse than I originally anticipated.

**Disclosure again: I got to run these before posting so I look smart :) Testing saves the day again!**

Now onto implementing our `CreateToken` function.  I'm using the package `"github.com/dgrijalva/jwt-go"` because it's the first that pops up when you type 'golang' and 'jwt' into google (and the Auth0 blog wrote a post on it as well, so you know it's good)

```go
// Creates a new Token for a User
func (t *TokenService) CreateToken(user User) (string, error) {
	safeUser := UserSafe{}
	safeUser.ID = user.ID
	safeUser.Email = user.Email
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
		"sub": user,
		"exp": time.Now().Add(time.Hour).Unix(),
	})

	// TODO: Remove hardcoded signed string
	tokenString, err := token.SignedString([]byte(secretKey))
	if err != nil {
		log.Println("Error signing token: ", err)
		return "", err
	}
	return tokenString, nil
}

```

In `model.go` I guess we should make the SafeUser struct.  This is just a helper struct to make sure we don't hand the passwords back to anyone that shouldn't have access to them.

```go
// ...
type UserSafe struct {
	ID    string `json:"id"`
	Email string `json:"email"`
}
```

Here we are parsing the interesting fields of our User into a SafeUser struct.  I've read some blogs saying that you can  pass around the password through your microservice architecture but I don't prescribe to that school of thought.  The less often your passwords are exposed on the wire the less chance you have of accidentally leaking it.  If you're making this service the trusted source of user authentication and authorization for your app, no other component needs to know what their password is.

Next we create a token by signing it using the HS256 method with the claims `sub` as the safeUser data and `exp` as the expiration date.  In a full implementation you may also issue a refresh token here but I'm not interested in handling that for this post.

We then take the token and stringify it then return it to the calling function!  run `go test` to make sure we have two less failing test now!  Wait! We are still not handling the scenario where we create a token for a user struct thats empty! That's illogical! Let's fix that really quickly.

```go
func (t *TokenService) CreateToken(user User) (string, error) {
	if (User{}) == user {
		log.Printf("%s", user.Email)
		return "", fmt.Errorf("Cannot create token for empty user")
	}
    // ...
}
```

In keeping with our TDD approach, let's whip up some scenarios to test our `ValidateToken` function.  We should at least try to validate an invalid token, and then validate a valid token.  Back to `tokenService_test.go`

```go
func TestValidateTokenFromInvalidToken(t *testing.T) {
	token := "thisisNOTatoken"
	payload, err := TS.ValidateToken(token)
	if err == nil {
		t.Errorf("Expected an error to be thrown as token is not valid")
	}
	if payload != nil {
		t.Errorf("Expected no user to be returned")
	}
}

func TestValidateTokenFromValidToken(t *testing.T) {
	user, err := DB.GetUserByEmail("tokentester@domain.com")
	if err != nil {
		t.Error(err)
	}
	if user.ID == "" {
		t.Errorf("Expected to get a user. Got none")
	}
	token, err := TS.CreateToken(user)
	if err != nil {
		t.Error(err)
	}
	if token == "" {
		t.Error("Got an empty token when expecting a real token")
	}
	payload, err := TS.ValidateToken(token)
	if err != nil {
		t.Error(err)
	}
	if payload == nil {
		t.Error("Expected to get a payload, got none")
	}
}
```

Now that we have our extensive tests planned out, we can implement the feature!

```go
func (t *TokenService) ValidateToken(tokenString string) (interface{}, error) {
	claims := jwt.MapClaims{}
	token, err := jwt.ParseWithClaims(tokenString, &claims, func(token *jwt.Token) (interface{}, error) {
		return []byte(secretKey), nil
	})
	if err != nil {
		return nil, err
	}
	if token.Valid {
		return claims["sub"], nil
	} else if ve, ok := err.(*jwt.ValidationError); ok {
		if ve.Errors&jwt.ValidationErrorMalformed != 0 {
			return false, fmt.Errorf("Malformed Token")
		} else if ve.Errors&(jwt.ValidationErrorExpired|jwt.ValidationErrorNotValidYet) != 0 {
			// Token is either expired or not active yet
			return false, fmt.Errorf("Token is Expired")
		} else {
			return false, fmt.Errorf("Couldn't handle the token")
		}
	} else {
		return false, fmt.Errorf("Error handling token: %v", err)
	}
}
```

There we go! We try to parse the token and if it succeeds, we return the subject of the token. If it's expired, or malformed, or the like then we return some errors! We could make this more bulletproof probably by returning a UserSafe object as opposed to an interface, but this post is getting long again and I'm getting lazy :)

Run our tests and everything should look good!

Finally! We're going to implement the `HandleValidateToken` endpoint! It's been a long fought battle but here we go! We're going to be more strict with our token payload so first we declare a struct for it, and then implement the function.

```go
// ...
type TokenPayload struct {
    Token  string `json:"token"`
}
// ...
func HandleValidateToken(w http.ResponseWriter, r *http.Request) {
	var Token TokenPayload
	err := json.NewDecoder(r.Body).Decode(&Token)
	if err != nil {
		fmt.Fprintf(w, "Error Decoding Body: %v", err.Error())
		return
	}
	payload, err := TS.ValidateToken(Token.Token)
	if err != nil {
		fmt.Fprintf(w, "Error Decoding Token: %v", err.Error())
		return
	}
	WriteResponse(w, http.StatusOK, payload)
}
```

You've got some errors though.  We didn't initialize TS in our app. Go into main.go and add a `InitializeTokenService` function and call it from main:

```go
// ...
func main() {
	fmt.Println("Hello, World!")
	InitializeDatabase()
	InitializeTokenService()
	InitializeHttpServer()
}
// ...
func InitializeTokenService() {
	TS = &TokenService{}
}

```

Finally, to bring it all together, we need to update our `Login` function to return something meaningful (a token) so update the function like so:

```go
func (db *DBClient) Login(email, password string) (string, error) {
	db.Open()
	defer db.Close()
	var user User
	err := db.client.View(func(tx *bolt.Tx) error {
		b := tx.Bucket([]byte(usersBucketName))
		userBytes := b.Get([]byte(email))
		json.Unmarshal(userBytes, &user)
		err := user.CheckPassword(password)
		if err != nil {
			log.Printf("ERROR: %s", err)
			log.Printf("Invalid Login Attempt for User: %s", email)
			return fmt.Errorf("Invalid Email/Password Combnation")
		}
		return nil
	})
	if err != nil {
		return "", err
	}
	token, err := TS.CreateToken(user)
	if err != nil {
		return "", err
	}
	return token, nil
}
```

Great Success!  We've all learned something here (hopefully).  I've learned how to make something actually useful with Go. You've hopefully learned it as well.  We've put together a service that is built using go, with an embedded database, and has just one small job to perform.  It's got some tests so you can update code and feel more confident about it, and you could call it from other apps on your network.

There's a bunch of security things you should do before using this in a production environment such as:

* Limiting hosts and ports that can make requests to the server (CORS)
* Setting up SSL certificates to host
* Disabling non-SSL/TLS connections
* Automatic backups of your database file
* Refactoring your secrets into a config file or to CLI variables
* Implementing functionality to revoke token access for a specific user
* Add in some permissions to the user to have some granularity for controls within your suite of apps

Hopefully these posts have been as insightful for you to read as me to write! I'll have the code up on my github in the near future.

