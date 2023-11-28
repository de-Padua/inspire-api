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

 Users need to create an account from a secure env and each user create a new Cookie code when it's acount is created,whe the user account is authenticaded in the server it generetes a session_id code,this sension id code 
 is time-limeted,about 5 hours it will reset and the user will need to be authenticaded again by providing his login data


```Javascript
  const cookie = req.cookies["SESSION_ID"];

  const expirationDate = new Date(Date.now() + 1000 * 60 * 1000);

  const validateSession = await sessionValidation.findOne({
    SESSION_ID: cookie,
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
