# exercise-todo-api-with-session-auth

## Learning Goals

After this lesson, you will be able to:

- Implement session-cookie authentication in a Node.js and Express application
- Create secure user registration and login functionality
- Protect API routes using authentication middleware
- Manage user sessions with MongoDB as the session store
- Structure REST APIs to handle authenticated requests without exposing user IDs in URLs

## Introduction

In this exercise, you'll enhance your To-do API by adding session-cookie authentication. You will build upon your existing implementation that already has MongoDB integration and a relationship between users and todos.

But this time around, you’ll implement session-cookie authentication to secure your routes!

## Getting Started

1. Start here by [forking this repository](https://github.com/codevergehq/exercise-todo-api-with-session-auth)
2. Clone your forked repository


```bash
git clone https://github.com/YOUR-USERNAME/exercise-todo-api-with-session-auth
```


Navigate into your newly cloned repository:

```bash
cd exercise-todo-api-with-session-auth
```

> What you've forked and cloned is essentially an empty project (except for the `.README` file). We'll now copy your previous project files into this repository.


Copy your previous project files over into the new directory:

```bash
# Assuming your previous project is in the same parent directory
cp -r ../todo-api-with-db/* .
```

>The command above assumes both projects are in the same parent directory. If your `todo-api-with-db` is located elsewhere, adjust the path accordingly.

Now because we already have a `.gitignore` file, the `node_modules` will not be copied over. 
You'll need to install the dependencies again using `npm install`.

Run the following command to install the existing dependencies already mentioned in the `package.json` file:

```bash
# Install existing dependencies
npm install
```

Let's also add the new dependencies we'll need for implementing session-based authentication.

```bash
# Install additional dependencies for session management
npm install express-session connect-mongo
```

## Tasks

### Task 1: Configure Session Management

Add these environment variables to your `.env` file:

```bash
SESSION_SECRET=your-session-secret-here
CRYPTO_SECRET=your-crypto-secret-here
```

Don’t forget to overwrite the default values, e.g. `your-session-secret-here` and set secure random strings for both. 

The important thing is to use long, random strings that would be difficult to guess.

Now update `app.js` to include session configuration with MongoDB store.



> 🫵🏻 Remember, you’ve just done this very recently in a code-along 🙂


### Task 2: Modify User Model and Routes

Currently, your user creation route in `routes/user.routes.js` looks something like this:

```jsx
router.post('/api/users', async (req, res, next) => {
  try {
    const { name, email } = req.body;
    const newUser = await User.create({ name, email });
    res.status(201).json(newUser);
  } catch (error) {
    next(error);
  }
});
```

To implement authentication, we need to:

First install `bcrypt` for password hashing

```bash
npm install bcrypt
```

> Remember — Never store plain-text passwords in your database! Always use bcrypt or another secure hashing algorithm.


Next, update your `User` model in `models/User.model.js` to include a `password` field.

The `password` field should be set as a required field. 
Please consider what other validations might be useful here too.

Modify your user creation route by changing the route from `/api/users` to `/api/auth/register` .

This better represents the authentication related nature of the endpoint and creates a clear distinction between authentication routes and resource routes (i.e. todos, users, etc.).

Implement the following in this route:

- Accept a password in the request body
- Hash the password before storing
- Create a session after successful registration
- Return appropriate success/error messages
- Handle any duplicate email errors

Test your registration endpoint using Bruno.

Ensure that you test the following scenarios:

- Successful registration flow
- Test with missing fields
- Test with a duplicate email address

Verify in MongoDB Compass that the user was created, verify that the password is properly hashed, and look for the session document in the sessions collection.


> Testing each step ensures that you catch issues early and have a solid foundation for the next features that you’ll build!


### Task 3: Implement Login Route

Now that you have registration working, let's implement user login.

First, create a new login route in `routes/user.routes.js`

The path should be `/api/auth/login`

This login route should handle email and password verification and create a session upon successful login.

Additionally, please ensure that the route handler logic:

- finds the user by email
- compares the passwords using bcrypt
- creates session for an authenticated user
- returns appropriate responses for both success and error states

Before proceeding further, please test your login endpoint using Bruno.

After testing with Bruno, you can inspect MongoDB Compass to check that a new session document was successfully created, and verify that the session contains the correct user ID.

### Task 4: Implement Authentication Middleware

Now that we have registration and login working, let's create the middleware that will protect our routes.

Create a new file `middleware/auth.middleware.js`

Inside this file, implement the authentication middleware that:

- Checks if `req.session.userId` exists (this indicates a valid session)
- If no session exists, returns a `401 Unauthorized response`
- If session exists, allows the request to continue to the route handler

The middleware should look something like this (but implement it yourself):

```jsx
// Example structure of what you need to implement
const isAuthenticated = (req, res, next) => {
    // Check if session exists and has userId
    // If not, return 401
    // If yes, continue to route
};

module.exports = isAuthenticated;
```

Test your middleware implementation before proceeding to the next task.

### Task 5: Restructure and Secure Todo Routes

Currently, your todo routes look like this:

```jsx
// Current route structure
GET    /api/users/:userId/todos          // Get user's todos
POST   /api/users/:userId/todos          // Create todo for user
PUT    /api/users/:userId/todos/:todoId  // Update todo
DELETE /api/users/:userId/todos/:todoId  // Delete todo
```

In this task, please change the structure of your todo routes to this:

```jsx
// New route structure
GET    /api/todos             // Get logged-in user's todos
POST   /api/todos             // Create todo for logged-in user
PUT    /api/todos/:todoId     // Update todo
DELETE /api/todos/:todoId     // Delete todo
```

**Why this change?!?**

Previously, we needed `:userId` in the URL to identify which user’s todos to access.

But now with session authentication, we can get the user ID from the session, i.e. `req.session.userId` (we’ll make this change shortly)!

This is more secure as user IDs are not exposed in URLs.

The routes are now cleaner and follow REST conventions better!

After updating your routes in `routes/todo.routes.js` to remove the `/users/:userId` segment , please add the `isAuthenticated` middleware to each of your todo routes to protect them.

Add your middleware function to the following routes:

```jsx
// Todo routes - All need authentication
GET    /api/todos             // Only logged-in users can see their todos
POST   /api/todos             // Only logged-in users can create todos
PUT    /api/todos/:todoId     // Only logged-in users can update their todos
DELETE /api/todos/:todoId     // Only logged-in users can delete their todos
```

The following routes should NOT be protected and these should remain as public routes:

```jsx
// Auth routes - Must be public!
POST   /api/auth/register     // Anyone can register
POST   /api/auth/login        // Anyone can login
POST   /api/auth/logout       // Anyone can logout
```

> All todo routes should be protected. Only authenticated users should be able to create, read, update, or delete todos. The authentication routes (register, login, logout) remain unprotected.

Finally, make sure to update each todo route handler to use `req.session.userId` instead of `req.params.userId`. 

Since we're now getting the user ID from the session, we don't need to pass it through the URL parameters anymore.

### Task 6: Test your protected routes using Bruno

Now that you’ve restructured your routes and implemented the `isAuthenticated` middleware on your todo routes, it’s time to test your protected routes using Bruno.

First try to access the `/api/todos` endpoint without logging in. It should fail!

```jsx
GET http://localhost:3000/api/todos
```

Next, try to login:

```jsx
POST http://localhost:3000/api/auth/login
```

Now try to access the `/api/todos` again while authenticated, and it should work:

```jsx
GET http://localhost:3000/api/todos
```

You should also test creating, updating and deleting todos while authenticated to ensure the session-based authentication is working as expected for all CRUD operations. Make sure to include the session cookie in all your Bruno requests after logging in.

Finally, test the logout endpoint to verify that it properly destroys the session and prevents further access to protected routes.

## Submission

When you’re done with the exercise, commit and push your changes to GitHub:

```bash
git add .
git commit -m "Implement session authentication"
git push origin main
```

Create a Pull Request on GitHub and submit it here:

## Frequently Asked Questions (FAQ)

<details><summary>What happens if I try to access a protected route without being logged in?</summary>
If you try to access a protected route without being logged in (i.e., without a valid session), the `isAuthenticated` middleware should return a 401 Unauthorized response. This prevents unauthorized access to protected resources.
</details>

<details><summary>How do I know if my session is working correctly?</summary>
    
You can verify your session is working by checking MongoDB Compass and looking for documents in the `sessions` collection. Also, if you can successfully access protected routes after logging in, your session is working properly!</details>

<details><summary>How can I protect a route?</summary>
    
  ```jsx
    // Example of a protected route
    router.get('/api/todos', isAuthenticated, async (req, res, next) => {
        try {
            // Use req.session.userId instead of req.params.userId
            const todos = await Todo.find({ userId: req.session.userId });
            res.json(todos);
        } catch (error) {
            next(error);
        }
    });
  ```
</details>
