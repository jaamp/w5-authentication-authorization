# Authentication

By the end of this lesson, you should be able to authorize your routes with JWTs and securely store passwords.

## Core Learning Objective

*	Implement a basic authentication setup using JSON web tokens

## Sub-Objectives

* Describe the authentication process
* Store passwords with bcrypt
* Create signin and login routes that return JWTs
* Describe the difference between authentication and authorizations
* Authorize certain routes and information

## Installation

1. Fork & clone

1. `cp nodemon.sample.json nodemon.json`

1. Create a new Cluster on [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) titled something like `general-purpose` or reuse one you already have.

1. Update the `MONGO_DB_CONNECTION` in your `nodemon.json` file with the full connection string. Make sure you include the password you set up for the database and change the name of the database from `test` to something lke `exclusive_party_dev`.

1. `npm install`

1. `npm run reset-db`

1. `npm run dev`

Once installation is working, take a look at the existing code to make sure you understand what is happening. Then, try making requests to the API.

### Instructions & Guiding Questions

- [ ] Take a look at the `db/seeds.js` file.

* **Question:** Describe what this code is doing and what its purpose is.

* **Your Answer: calls deleteMany() wich clears out the db then it creates two new db entries.  Purpose is to just have clean data to work with and get rid of any old test material

---

- [ ] Imagine that as a user, you enter your username and password into a site in order to signup.

* **Question:** What happens next?

* **Your Answer: the server verifys that you have an account and that the credentials that you provided are correct and then you are given access at the role level your are authorized

---

- [ ] Imagine that as a user, you are now logging back into that same website. 

* **Question:** How does the website verify that you are indeed the same user?

* **Your Answer: only based on the fact of you having the correct credentials, if someone else has your credentials it does not know it is you only that the person entering the credentials have the correct credential

---

- [ ] Imagine that as a logged-in user, you try to go to a route you are not supposed to (e.g. /admin).

* **Question:** How does the website know you are or are not allowed on a specific route?

* **Your Answer: most have role based authorization, based on the credentials you provide you are given a token, the server then checks the token to see if it has access to the resource you are trying to access

* **Question:** Describe the difference between authentication and authorization.

* **Your Answer: authentication is providing credentials to prove you are who you say you are.  authorization is based on your authentication what resources you are allowed to access

---

- [ ] Build a new model called `Guest`. The `Guest` model should have the following fields: `username`, `password`

---

- [ ] Create a new route at `POST /api/signup` at `/api/routes/auth.js`. This route should:
  1. Create a new `Guest`, pulling `username` and `password` from the request body
  1. Return a success message with the user's information (for now)

* **Question:** This code is currently _very_ insecure. Why?

* **Your Answer: the username and password is being sent as part of an unencrypted json file. 

* **Question:** What would happen if three different users tried to sign up with the same username? How can we prevent that?

* **Your Answer: three entries would be created, with the same username and three different passwords, each with a different id.  set a unique value requirement, that would not allow the same username to be repeated in subsequent data rows

* **Question:** Why are we making our route `POST /api/signup` as opposed to `POST /api/users`?

* **Your Answer: it goes to different models and different databases - it also allows for future development in a clearer way, haveing a seperate sign up from users allows us to do more with users.

---

- [ ] We need a way to securely store a password in our database. Install [node.bcrypt.js](https://www.npmjs.com/package/bcrypt), require it in your new routes file, and use the `bcrypt.hash()` method to encrypt the password before storing it. Test your signup process to make sure the password is hashed.

  _NOTE 1:_ It is not uncommon for issues to arise while trying to install bcrypt. If you encounter issues during the lesson trying to get it to install, check the [Install via NPM](https://www.npmjs.com/package/bcrypt#install-via-npm) section of the documentation to see if you can find any help. If not, take notes until we move pass bcrypt and we can get it resolved during a break or after class.

  _NOTE 2:_ Use the promisified `bcrypt.hash()` method instead of the callback version.

* **Question:** Describe what is meant by the term `saltRounds`. If you need help, the [documentation](https://www.npmjs.com/package/bcrypt#a-note-on-rounds) has some explanation. This [StackOverflow answer](https://stackoverflow.com/a/46713082) also might help.

  _NOTE: We will not go into this too deeply for the sake of brevity, however this is a really interesting topic! I would encourage you to look into this more on your own, if you're interested._

* **Your Answer: the number of times a password is ran through a salting and hashing, the more rounds the longer the time and more calculations are being performed, but also the more secure the password is.

---

- [ ] Right now, users can create new accounts with the same username. Update your code so that before we create a guest, we check to see whether or not a guest already exists with that username. If it does, return an error.

  _NOTE: While it is possible to use the `unique: true` constraint on our model, it requires a bit of extra configuration to get working properly. See the [Advanced](#advanced) section below for more information on how to do this!_

---

- [ ] Now that we can signup a user, we want them to be able to login to our site. Build a `POST /login` route that expects a username and password in the request body. Use the `bcrypt.compare()` function to compare the incoming plain text password with the hashed password stored in MongoDB. If the username or password is incorrect, return a non-specific error message (e.g. "Login credientials incorrect.") with a status code of 401. If the username and password combination is correct, return a temporary success message (e.g. "You are now logged in") and a status code of 201.

* **Question:** Why is it important to give a non-specific error message as opposed to a message like "Password incorrect?"

* **Your Answer: it gives hackers answers.  if you get a general message, they do not know if the username or the password was incorrect, if you say password incorrect then, they know what the user name is, cutting down the work to hack a system.

---

- [ ] The above process can be a bit tricky. Take a moment to annotate your code with comments, explaining each step of your code.

---

- [ ] On subsequent requests to our API, how will we know a user is logged in? We need to provide them with something so that we later know they have indeed successfully logged in. There are a few different strategies for this, but we will be using [JWTs](https://jwt.io/introduction/) (pronounced "jots"). Take a moment to read the section titled "What is the JSON Web Token structure?"

* **Question:** In your own words, describe the three parts of a JWT.

* **Your Answer: Header, Payload, Signature:  The header contains the type of token (such as JWT) and the signing hash algorithm being used.  This is to let the receiving server know how to handle the token.  The payload is the claims.  Claims is the way which defines how applications get the identity information about a user.  There are several differnt Claims methods. Registered, Public and Private.  Finally there is the Signatue, which takes the endcoded header, the encoded payload and a secret to develop the hash, by checking this on the server side if the hash match you can expect the message has not been modified and the person making the identitiy claim is who they are.

---

- [ ] We will implement JWTs using the [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) package. Install this package and include it at the top of your `auth.js` file.

* **Question:** Which of our current routes will require us to use the `jsonwebtoken` library? (i.e. When will we be creating or decoding JWTs?)

* **Your Answer: the login route so that the token can be used to to verify requestor

* **Question:** JWTs allow for custom information (i.e. payload) to be returned back to the client. What kind of information do you think would be useful to send back to our client?

* **Your answer: User name can be used for other js tasks, the expiration of the token, permissions of user settings

* **Question:** The custom information (i.e. payload) inside of JWT can be [easily decoded](https://jwt.io/#debugger). What kind of information should we _not_ store inside of a JWT?

* **Your Answer: password, or other sensitive information that can be captured and used by hackers

---

- [ ] Add the following code to `/login` route and then respond with the token when a user successfully is able to login. _NOTE: In the example below, I assume you've required the package and assigned it to a `jsonwebtoken` variable._

  ```js
  const payload = { id: guest._id }
  const options = { expiresIn: '1 day' }
  const token = jsonwebtoken.sign(payload, 'MYSECRETPASSCODE', options)
  ```

* **Question:** The `.sign()` method takes three arguments. Describe each argument in your own words, using the above code as an example.

* **Your Answer: basically this creates the token that is sent out.  the payload is the JWT header information and payload , the second is the passcode the secret no one should know, the third are the options sucn as expiration, algorithm used, access timeframe who issued the token, id, ect...

---

- [ ] Right now our secret is not so secret. Add a new environment variable to your `nodemon.json` file that stores the secret code. Then, use it in your `auth.js` file. _NOTE: Make sure to restart your server!_

---

- [ ] Modify the `/signup` route to return a token in place of the user information.

---

- [ ] We are now responding with JWTs when a user is properly **authenticated**. We will use those JWTs to **authorize** whether or not someone is allowed to visit certain routes or gain certain information.

* **Question:** Describe the difference between **authentication** and **authorization**, given the above context.

* **Your Answer: authenticated is when the user passes the correct credentials within the token to verify who they are.  The result is authorization or permission to access a given resource.

---

- [ ] Add the following route to the top of your `auth.js` file. Then, make a request to this route in Postman.
  
  ```js
  router.get('/profile', async (req, res, next) => {
    try {
      const token = req.headers.authorization.split('Bearer ')[1]
      const payload = jsonwebtoken.verify(token, SECRET_KEY)
      const guest = await Guest.findOne({ _id: payload.id }).select('-__v -password')

      const status = 200
      res.json({ status, guest })  
    } catch (e) {
      console.error(e)
      const error = new Error('You are not authorized to access this route.')
      error.status = 401
      next(error)
    }
  })
  ```

* **Question:** What happens? Why?

* **Your Answer: The error message is returned and no access is granted, because the authorization token was not passed

---

- [ ] In order to successfully access this route, we will need to send over the token in the HTTP Authorization Header. The typical way to do this is by sending a [Bearer token](https://security.stackexchange.com/questions/108662/why-is-bearer-required-before-the-token-in-authorization-header-in-a-http-re). To do this in Postman, go to the "Authorization" tab, select "Bearer Token" as the Type, and then enter your token.

* **Question:** What happens? Why?

* **Your Answer:by adding the token, the route model is able to use the claims made in the token to authorize the user and grant access

---

- [ ] There is a lot going on in the above code. Take a moment to annotate each line so you are able to confirm your understanding of what is happening.

### Mini-Exercise

Our final step is to **authorize** users to view certain routes or information. With a partner, complete the following:

1. Using route-level middleware, protect the `GET /api/parties/exclusive` route. If a user provides a token, they are able to see all of the exclusive parties. If not, return a 401 Unauthorized message.

1. In the `GET /api/parties` route, return all parties if a user is authorized. Otherwise, return only those parties where `exclusive: false`.

1. In the `GET /api/parties/:id` route, return the party if the user is authorized. If the user is not authorized, only return the party if it is _not_ exclusive. Otherwise, return a 401 Unauthorized message.

### Advanced

#### Building a New Index

If you want to build a new [index](https://docs.mongodb.com/manual/indexes/), you will have to connect directly to MongoDB using your command line due to the fact that we are using a free tier from MongoDB Atlas. It's not too hard!

1. From the "Overview" tab on your MongoDB cluster, click the "Connect" button.

1. Click "Connect from Mongo Shell"

1. Follow the instructions for installing MongoDB locally on your system

1. Copy the command in the final step of the instructions and paste that into your command line

1. Enter your password as defined in your `nodemon.json` file

1. Once connected, try running the `show dbs` command

1. Connect to your database using the `use <database-name>` command

1. Run the following:
  ```
  db.guests.createIndex({ "username": 1}, { unique: true })
  ```

1. Type `exit` to leave the shell