# inspire-api

## Overview


This is my backend for my e-commerce,if you wanna use this api i recommend you to configure it in the way that you need.



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

----------------

## User Retrieval Endpoint Explanation

- **Route Definition:**  
  This code defines a route for handling HTTP GET requests to "/users". It uses the `jsonParser` middleware to parse incoming JSON data from the request body.

- **Session Validation:**  
  It retrieves the session ID from the request cookies and checks if there is a corresponding session in the database using `sessionValidation.findOne`.

- **Handling Missing Session:**  
  If there is no valid session, it responds with a JSON object indicating failure with a HTTP status code of 400.

- **Handling Valid Session:**  
  If there is a valid session, it attempts to retrieve all users from the `USER_MODEL_DB`. If successful, it responds with a JSON object containing the user data and a success message.

- **Error Handling:**  
  There is a try-catch block to catch any potential errors during the user retrieval process. If an error occurs, it is logged to the console.

- **Handling No Session:**  
  If there is no session cookie in the request, it responds with a JSON object indicating no data with a HTTP status code of 200.

```javascript


app.get("/users", jsonParser, async (req, res) => {
  const cookie = req.cookies["SESSION_ID"];

  const validateSession = await sessionValidation.findOne({
    SESSION_ID: cookie,
  });

  if (validateSession === null) {
    res.status(400).json({
      sucess: false,
    });
  } else if (cookie !== undefined) {
    try {
      const users = await USER_MODEL_DB.find();
      res.json({ data: users, sucess: true });
    } catch (err) {
      if (err) {
        console.log(err);
      }
    }
  } else {
    res.status(200).json({ data: null });
  }
});


```

-----------------------------


## Checkout Endpoint Explanation

- **Route Definition:**  
  This code defines a route for handling HTTP POST requests to "/checkout". It uses the `jsonParser` middleware to parse incoming JSON data from the request body.

- **Request Data Retrieval:**  
  It retrieves product information (`PRODUCTS_FROM_CLIENT`) and user ID (`USERID`) from the request body. Also, it generates a unique purchase ID (`purchase_id`) using `uuidv4()`.

- **User Retrieval:**  
  It queries the `userModel` to find a user with the specified email (`USERID`).

- **User Existence Check:**  
  If no user is found, the process is terminated.

- **Logging Product Information:**  
  It logs the product information received from the client.

- **Quantity Check:**  
  It uses asynchronous mapping (`Promise.all`) to check if each item in the cart has enough quantity available in the store. The result is an array (`checkStore`) indicating whether each item has enough quantity.

- **Check for Insufficient Quantity:**  
  If any item in the cart has insufficient quantity, the process is terminated.

- **Formatting Product Data for Stripe:**  
  It formats the product data from the client to a structure suitable for Stripe Checkout.

- **Creating a Checkout Session:**  
  It uses the Stripe API to create a new checkout session. The session includes line items, payment mode, shipping address collection details, customer information, success, and cancel URLs.

- **Response:**  
  It responds with a JSON object containing the Stripe Checkout session URL.

```javascript
app.post("/checkout", jsonParser, async (req, res) => {
  const PRODUCTS_FROM_CLIENT = req.body.cart;
  const USERID = req.body.user;

  const purchase_id = uuidv4();

  const user = await userModel.findOne({ email: USERID });

  if (!user) return;

  console.log(PRODUCTS_FROM_CLIENT);

  //checkStore will return a array with ["true"] or ["false"]

  const checkStore = await Promise.all(
    PRODUCTS_FROM_CLIENT.map(async (item) => {
      let data = false;

      const checkIfItemHasEnoughQuantity = await products_model.findOne({
        title: item.name,
      });

      if (checkIfItemHasEnoughQuantity.quantity_available < item.quantity) {
        data = true;
      }

      return data;
    })
  );

 


  if (checkStore.includes(true)) return;


  const client_data_products_formated = PRODUCTS_FROM_CLIENT.map((i) => {
    return {
      price_data: {
        currency: "BRL",
        product_data: {
          name: i.name,
        },
        unit_amount: Math.round(i.price * 100),
      },
      quantity: i.quantity,
    };
  });

  const session = await stripe.checkout.sessions.create({
    line_items: client_data_products_formated,
    mode: "payment",

    shipping_address_collection: {
      allowed_countries: ["BR"],
    },
    custom_text: {
      shipping_address: {
        message:
          "Please note that we can't guarantee 2-day delivery for PO boxes at this time.",
      },
      submit: {
        message: "We'll email you instructions on how to get started.",
      },
    },
    customer: user.stripe_id,
    success_url: `${domain + purchase_id}?success=true`,
    cancel_url: `${domain + purchase_id}?canceled=true`,
  });

  res.json({ url: session.url });
});
```


---------------------

## Purchase Webhook Endpoint Explanation

- **Route Definition:**  
  This code defines a route for handling HTTP POST requests to "/purchasewebhook". It uses `bodyParser.raw` middleware to parse incoming raw JSON data from the request body.

- **Request Data Retrieval:**  
  It retrieves the raw payload (`payload`) from the request body and the signature (`sig`) from the request headers.

- **Event Validation:**  
  It attempts to construct a webhook event using the Stripe API (`stripe.webhooks.constructEvent`). If there's an error, it responds with a 400 status and an error message.

- **Event Type Check:**  
  It checks the type of the webhook event. If it's a "checkout.session.completed" event:
  - It retrieves the session with line items from Stripe.
  - Logs the line items data.
  - Extracts customer ID and creates a new invoice object.

- **Adding Invoice to User:**  
  - It finds the current user in the database using the customer ID.
  - Updates the user's database by adding the new invoice and clearing the cart.

- **Updating Products in Database:**  
  - It iterates over the line items and updates the corresponding products in the database by reducing available quantity and increasing the number of purchases.

- **Handling Payment Failure:**  
  If the event type is "payment_intent.payment_failed," it can be handled accordingly (currently commented out).

- **Response:**  
  Responds with a 200 status to acknowledge successful processing of the webhook event.

Feel free to use this Markdown code in your documentation!





```javascript
app.post(
  "/purchasewebhook",
  bodyParser.raw({ type: "application/json" }),
  async (req, res) => {
    const payload = req.body;

    const sig = req.headers["stripe-signature"];

    let event;

    try {
      event = stripe.webhooks.constructEvent(payload, sig, webhook_secret);
    } catch (err) {
      return response.status(400).send(`Webhook Error: ${err.message}`);
    }

    if (event.type === "checkout.session.completed") {
      const sessionWithLineItems = await stripe.checkout.sessions.retrieve(
        event.data.object.id,
        {
          expand: ["line_items"],
        }
      );
      const lineItems = sessionWithLineItems.line_items;

      console.log(lineItems.data);

      const customerId = event.data.object.customer;

      //add invoice to user
      const newInvoice = {
        id: event.id,
        date: formattedDate,
        total: event.data.object.amount_total,
        adress: { ...event.data.object.shipping_details },
        status: event.data.object.status,
        items: lineItems.data,
      };

      const currentUser = await userModel.findOne({ stripe_id: customerId });

      const addInvoiceToUserDB = await userModel.findOneAndUpdate(
        {
          email: currentUser.email,
        },
        { invoices: [...currentUser.invoices, newInvoice], cart: [] }
      );

      //update products documents

      const updateProducstdb = Promise.all(
        lineItems.data.forEach(async (element) => {
          const currentItem = await products_model.findOne({
            title: element.description,
          });
          const updateCurrentItem = await products_model.findOneAndUpdate(
            { title: currentItem.title },
            {
              quantity_available:
                currentItem.quantity_available - element.quantity,
              buys: currentItem.buys + element.quantity,
            }
          );
        })
      );
    }
    if (event.type === "payment_intent.payment_failed") {
      //return to user
    }

    res.status(200).end();
  }
);
```







## API Endpoints

1. **Logout**
   - Endpoint: `app.post("/logout", jsonParser, async (req, res) => {...})`

2. **Get Users**
   - Endpoint: `app.get("/users", jsonParser, async (req, res) => {...})`

3. **Update Users**
   - Endpoint: `app.patch("/users", jsonParser, async (req, res) => {...})`

4. **Update User Cart**
   - Endpoint: `app.patch("/update/usercart", jsonParser, async (req, res) => {...})`

5. **Delete User**
   - Endpoint: `app.delete("/users", jsonParser, async (req, res) => {...})`

6. **Create User**
   - Endpoint: `app.post("/users", jsonParser, async (req, res) => {...})`

7. **User Login**
   - Endpoint: `app.post("/users/login", jsonParser, async (req, res) => {...})`

8. **Get Products**
   - Endpoint: `app.get("/products", jsonParser, async (req, res) => {...})`

9. **Get Product by ID**
   - Endpoint: `app.get("/products/:id", jsonParser, async (req, res) => {...})`

10. **Test Route**
    - Endpoint: `app.get("/testRoute", (req, res) => {...})`

11. **Checkout**
    - Endpoint: `app.post("/checkout", jsonParser, async (req, res) => {...})`
    
12. **Purchase Webhook**
    - Endpoint: `app.post("/purchasewebhook", bodyParser.raw({ type: "application/json" }), async (req, res) => {...})`



    
## Authentication

This api uses a Cookies-Header-Http-Only approach,you can check out at this  [https://github.com/de-Padua/inspire-api/blob/main/routes/users.js]link 
## Error Handling


