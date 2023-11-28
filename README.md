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



 1- To create an account 

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


bcrypt.compare:

    This is a function provided by the bcrypt library to compare a plain text password (req.body.password) with a hashed password stored in the database (userDataFromDb.password).
    It takes three parameters: the plain text password, the hashed password from the database, and a callback function to handle the result
 

this code is handling the comparison of a user-provided password with the hashed password stored in the database using bcrypt. If the passwords match, it creates a new session cookie and sends a success response with user data. If the passwords don't match, it sends an error response.


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
