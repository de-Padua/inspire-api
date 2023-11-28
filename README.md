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

When a user creates an account, a unique cookie code is generated for them in a secure environment. This cookie code serves as an identifier for the user. Upon successful authentication of the user's account on the server, a session ID code is generated. This session ID code has a time limit, typically around 5 hours. After this period, the session ID expires, and the user needs to re-authenticate by providing their login credentials again.

Here's a step-by-step breakdown:

    Account Creation:
        Users create an account, and a unique cookie code is generated for their session in a secure environment.

    Authentication:
        When a user logs in, the server authenticates their account.
        If the authentication is successful, a session ID code is generated for that session.

    Session Expiry:
        The session ID has a time limit, usually set to around 5 hours.
        After this time period, the session ID expires.

    Re-authentication:
        Once the session ID expires, the user needs to re-authenticate.
        Re-authentication involves providing login credentials again to obtain a new session ID.

This approach enhances security by regularly refreshing the session ID and ensuring that users periodically re-verify their identity.


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
