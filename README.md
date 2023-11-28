# Your API Name

## Overview

Briefly describe what your API does, its main features, and the problems it solves.

## Table of Contents


## Getting Started

### Prerequisites

This api is using Node.js and Express with HTTPS-only request with cookies to authentication


Note : This api is still in development and does require alot of configuration,since this is not a tutorial about how to install and use this api,i'll not be provind a step-by-step on how to use it,
this api is using Stripe as payment method and require you to have a key,make sure that you have one and put it on the .env file on your project

### Installation

## Clone this Repository and Install Dependencies

To get started with this project, follow these steps:

1. **Clone the repository:**
```git

git clone https://github.com/de-Padua/inspire-api

```
1. **Install dependecies:**

```npm

npm i 

```



### Usage

This api is using Node.js and Express with HTTPS-only request with cookies to authentication



## 1- To create an account 

Route Definition:
This code defines a route for handling HTTP POST requests to "/users". It uses the jsonParser middleware, configured to parse incoming JSON data from the request body.

Checking for Existing User:
Queries the database (USER_MODEL_DB) to check if a user with the specified email already exists.

Handling Existing User:
If a user with the provided email already exists in the database, it responds with a JSON object indicating that the email is already in use.

Creating a New User:
If the email is not in use, it proceeds to create a new user. It uses the Stripe API to create a new customer, hashes the password using bcrypt, and creates a new user object with the provided name, email, hashed password, and a Stripe customer ID.

Creating a New Session:
It creates a new session using a model named sessionValidation. The session ID is generated and associated with the user's email. The session is then saved in the database.

Setting Cookie and Responding:
The new user is saved in the database. The response sets an HTTP status code of 200, sets a cookie named "SESSION_ID" with the session ID, and sends a JSON response indicating that the user has been created successfully.


```Javascript
  app.post("/users", jsonParser, async (req, res) => {
  const userDataFromDb = await USER_MODEL_DB.findOne({
    email: req.body.email,
  });

  if (userDataFromDb) {
    res.json({
      data: {
        message: "Email already in use",
        error: null,
      },
    });
  } else {
    const stripeCustomer = await stripe.customers.create();
    const hashedPassword = await bcrypt.hash(req.body.password, 10);

    const newUser = new USER_MODEL_DB({
      name: req.body.name,
      email: req.body.email,
      password: hashedPassword,
      stripe_id: stripeCustomer.id,
    });

    const newSession = new sessionValidation({
      SESSION_ID: uuidv4(),
      USER_MAGNET: req.body.email,
    });

    await newSession.save();
    await newUser.save();

    res
      .status(200)
      .cookie("SESSION_ID", newSession.SESSION_ID, {
        secure: true,
        httpOnly: true,
        sameSite: "none",
      })
      .json({
        data: {
          message: "user created ",
          error: null,
          sucess: true,
        },
      });
  }
});
```



-----------------------------------------


## 2 - Login route 

Route Definition:
This code defines a route for handling HTTP POST requests to "/users/login". It uses the jsonParser middleware to parse incoming JSON data from the request body.

Checking for Existing Session:
It retrieves the session ID from the request cookies and checks if there is a corresponding session in the database.

Handling Existing Session without Credentials:
If the request body does not contain email and there is a valid session cookie, it retrieves user data from the database based on the session's user email. If the user exists, it sets a new session cookie and responds with the user data.

Handling Login Attempt:
If the request body contains email and password, it attempts to find the user in the database. If the user is not found, it responds with a message indicating that the user was not found.

Deleting Old Session:
If the user is found, it deletes any existing sessions for that user.

Creating a New Session:
It creates a new session for the user, associating it with a new session ID.

Password Comparison:
It uses bcrypt to compare the provided password with the hashed password from the database. If there's an error during comparison, it responds with an internal server error. If the passwords match, it sets a new session cookie, specifies the expiration date, and responds with a success message and user data.

Password Mismatch Handling:
If the passwords do not match, it responds with a message indicating that the passwords do not match.


```javascript
app.post("/users/login", jsonParser, async (req, res) => {
  const cookie = req.cookies["SESSION_ID"];

  const expirationDate = new Date(Date.now() + 1000 * 60 * 1000);

  const validateSession = await sessionValidation.findOne({
    SESSION_ID: cookie,
  });

  if (req.body.email === undefined && cookie !== undefined) {
    const userDataFromDb = await USER_MODEL_DB.findOne({
      email: validateSession.USER_MAGNET,
    });

    if (userDataFromDb !== null) {
      res
        .cookie("SESSION_ID", validateSession.SESSION_ID, {
          secure: true,
          httpOnly: true,
          sameSite: "none",
        })
        .json({
          sucess: true,
          data: userDataFromDb,
        });
    }
  } else if (req.body.email !== null && req.body.password !== null) {
    const userDataFromDb = await USER_MODEL_DB.findOne({
      email: req.body.email,
    });

    const deleteOldSession = await sessionValidation.findOneAndDelete({
      USER_MAGNET: req.body.email,
    });

    if (userDataFromDb === null) return res.json({ data: "user not found" });

    const newSession = new sessionValidation({
      SESSION_ID: uuidv4(),
      USER_MAGNET: req.body.email,
    });

    await newSession.save();

    bcrypt.compare(
      req.body.password,
      userDataFromDb.password,
      function (err, response) {
        if (err) {
          res.json({ data: "internal server error" });
        }
        if (res) {
          console.log(res);
          res
            .cookie("SESSION_ID", newSession.SESSION_ID, {
              expires: expirationDate,
              secure: true,
              httpOnly: true,
              sameSite: "none",
            })
            .json({
              sucess: response,
              data: userDataFromDb,
            });
        } else {
          // response is OutgoingMessage object that server response http request
          res.json({
            success: false,
            message: "passwords do not match",
          });
        }
      }
    );
  }
});
```

----------------------------

## User Cart Update Endpoint Explanation

- **Route Definition:**  
  This code defines a route for handling HTTP PATCH requests to "/update/usercart". It uses the `jsonParser` middleware to parse incoming JSON data from the request body.

- **Updating User Cart:**  
  The endpoint attempts to update the user's cart in the database. It uses `updateOne` from the `USER_MODEL_DB` to find the user by email and update the cart with the new data from the request body.

- **Handling Success:**  
  If the update is successful, it responds with a JSON object containing the updated user data and a success message. The HTTP status code is set to 200.

- **Handling Failure:**  
  If the update fails for any reason, it responds with a JSON object indicating that something went wrong. The HTTP status code is set to 400.

- **Error Handling:**  
  There is a try-catch block to catch any potential errors during the update process. If an error occurs, it is logged to the console.


```javascript
app.patch("/update/usercart", jsonParser, async (req, res) => {
  console.log("ITS HERE");

  try {
    const updatedUser = await USER_MODEL_DB.updateOne(
      { email: req.body.email },
      { cart: req.body.cart }
    );
    if (updatedUser) {
      res.status(200).json({ data: updatedUser, message: "success" });
    } else {
      res.status(400).json({ data: null, message: "something went wrong" });
    }
  } catch (err) {
    if (err) {
      console.log(err);
    }
  }
});
```




## API Endpoints

List and describe the available endpoints in your API, including their methods, parameters, and expected responses.

## Authentication

Explain the authentication mechanism used by your API, whether it's API keys, OAuth, JWT, etc.

## Error Handling

Document how errors are handled in your API and provide a list of possible error codes and their meanings.

## Examples

Include more detailed examples or use cases to help developers understand how to integrate your API into their projects.

## Contributing

If you're open to contributions, provide guidelines for how others can contribute to your project.
