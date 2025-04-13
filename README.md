# Latest_BackendNotes

https://www.udemy.com/course/backend-master-class-golang-postgresql-kubernete

### dbdiagram.io
![image](https://github.com/user-attachments/assets/61684fe7-380a-454a-b3be-2695a0523356)


# 🔥 MongoDB Backend Mastery with Node.js, Express & Next.js

This guide walks you through setting up a complete backend with MongoDB using both Node.js/Express and Next.js API routes. It includes CRUD, Authentication (JWT), and MongoDB Atlas integration.

---

- Har category ke liye 2 chize chahiye- Ek route aur ek controller. 
  - Routes folder me auth ke related routes ki ek file bnayi aur usme get(/signup,authController.signup) aur get("/login",authController.login) bnaya. Ek particular structure ke aage bhi hum route include kr skte hai, for example, app.use('/api/auth', authRoutes); to iske andr, the router will listen for parent/signup, and the parent is /api/auth. so the overall URL our route will listen for to call its controller will be `/api/auth/signup` and `api/auth/login`
  - Controller folder me auth ke related controlling functions ki ek file bnayi aur usme se signupContr = (req,res)=>{..} aur loginContr = (req,res)=>{..} export kara
  - Since getting all users protected route hota hai, we need to check the request ka authorization, after getting req and before sending res, so we put a middleware between req and res
  - In the userRoutes file we export router.get('/getUserById', verifMiddleware, userController.getAllUsers);

  - this is how a typical middleware in a file in the middleware folder looks like:
  ```
  export.verifMiddleWare = function (req, res, next) {
  const token = req.header('Authorization')?.split(' ')[1]; // Get token
  if (!token) return res.status(401).json({ msg: 'No token, authorization denied' });

  try {
    const decoded = jwt.verify(token, JWT_SECRET); // Verify token
    req.user = decoded; // Attach to request
    next(); // Continue
  } catch (err) {
    res.status(400).json({ msg: 'Invalid token' });
  }
  };
  ```

  - if the middleware returns next() then the controller will execute for the route
  ```
  exports.getUserById = async (req, res) => {
  try {
    const user = await User.findById(req.params.id); // Find user
    if (!user) return res.status(404).json({ msg: 'User not found' });
    res.json(user);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
  };
  ```

  - To do changes in the db, we need a Model to fit the DB _schema_ and structure, to this is how we create a model and export the schema
  -  ```
     const mongoose = require('mongoose'); // Import mongoose
      Define schema
     const userSchema = new mongoose.Schema({
     name: { type: String, required: true }, // Name is required
     email: { type: String, required: true, unique: true }, // Unique email
     password: { type: String, required: true } // Hashed password
     }, { timestamps: true }); // Add createdAt and updatedAt
     module.exports = mongoose.model('User', userSchema); // Export model
     ```
## Now you can read the below written notes and the code for more understanding  

## 🟦 PART 1: MongoDB with Node.js + Express

### ✅ STEP 1: Project Setup

```bash
mkdir express-mongo-api && cd express-mongo-api
npm init -y
npm install express mongoose dotenv cors bcryptjs jsonwebtoken
```
> Initializes project and installs required packages.

---

### ✅ STEP 2: Folder Structure

```
express-mongo-api/
├── models/              # Mongoose models
│   └── User.js
├── routes/              # Route files
│   ├── userRoutes.js
│   └── authRoutes.js
├── controllers/         # Logic separated from routes
│   ├── userController.js
│   └── authController.js
├── middleware/          # Middleware functions
│   └── auth.js
├── .env                 # Environment variables
├── index.js             # Entry point
```

---

### ✅ STEP 3: MongoDB Config in `.env`

```env
MONGO_URI=mongodb://localhost:27017/expressdb
PORT=3000
JWT_SECRET=supersecret123
```

---

### ✅ STEP 4: `index.js` – Main Server File

```js
require('dotenv').config(); // Load .env variables
const express = require('express'); // Import Express
const mongoose = require('mongoose'); // Import Mongoose
const cors = require('cors'); // Handle cross-origin requests

const userRoutes = require('./routes/userRoutes'); // Import user routes
const authRoutes = require('./routes/authRoutes'); // Import auth routes

const app = express(); // Create Express app
app.use(cors()); // Enable CORS
app.use(express.json()); // Parse JSON body

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URI)
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error(err));

// Define API routes
app.use('/api/users', userRoutes);
app.use('/api/auth', authRoutes);

// Start server
app.listen(process.env.PORT, () =>
  console.log(`Server running at http://localhost:${process.env.PORT}`)
);
```

---

### ✅ STEP 5: `models/User.js` – Mongoose Model

```js
const mongoose = require('mongoose'); // Import mongoose

// Define schema
const userSchema = new mongoose.Schema({
  name: { type: String, required: true }, // Name is required
  email: { type: String, required: true, unique: true }, // Unique email
  password: { type: String, required: true } // Hashed password
}, { timestamps: true }); // Add createdAt and updatedAt

module.exports = mongoose.model('User', userSchema); // Export model
```

---

### ✅ STEP 6: `controllers/userController.js` – CRUD Logic

```js
const User = require('../models/User'); // Import User model

// GET all users
exports.getAllUsers = async (req, res) => {
  const users = await User.find();
  res.json(users);
};

// GET user by ID
exports.getUserById = async (req, res) => {
  try {
    const user = await User.findById(req.params.id); // Find user
    if (!user) return res.status(404).json({ msg: 'User not found' });
    res.json(user);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
};

// UPDATE user
exports.updateUser = async (req, res) => {
  try {
    const updated = await User.findByIdAndUpdate(req.params.id, req.body, { new: true }); // Update and return new doc
    res.json(updated);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
};

// DELETE user
exports.deleteUser = async (req, res) => {
  try {
    await User.findByIdAndDelete(req.params.id); // Delete user
    res.sendStatus(204); // No content
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
};
```

---

### ✅ STEP 7: `routes/userRoutes.js` – Route Handlers

```js
const express = require('express'); // Import express
const router = express.Router(); // Create router
const auth = require('../middleware/auth'); // Import auth middleware
const userController = require('../controllers/userController'); // Import controller

router.get('/', auth, userController.getAllUsers); // GET all users (protected)
router.get('/:id', auth, userController.getUserById); // GET user by ID
router.put('/:id', auth, userController.updateUser); // UPDATE user
router.delete('/:id', auth, userController.deleteUser); // DELETE user

module.exports = router; // Export router
```

---

### ✅ STEP 8: Auth Logic – `controllers/authController.js`

```js
const bcrypt = require('bcryptjs'); // For password hashing
const jwt = require('jsonwebtoken'); // For generating token
const User = require('../models/User'); // Import User model

const JWT_SECRET = process.env.JWT_SECRET;

// Signup controller
exports.signup = async (req, res) => {
  const { name, email, password } = req.body;

  const existing = await User.findOne({ email });
  if (existing) return res.status(400).json({ msg: 'Email already registered' });

  const hashed = await bcrypt.hash(password, 10); // Hash password
  const user = await User.create({ name, email, password: hashed }); // Create user

  const token = jwt.sign({ id: user._id }, JWT_SECRET, { expiresIn: '1d' }); // Generate token
  res.json({ token, user: { id: user._id, name, email } });
};

// Login controller
exports.login = async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email });
  if (!user) return res.status(404).json({ msg: 'User not found' });

  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) return res.status(400).json({ msg: 'Invalid credentials' });

  const token = jwt.sign({ id: user._id }, JWT_SECRET, { expiresIn: '1d' }); // Generate token
  res.json({ token, user: { id: user._id, name: user.name, email: user.email } });
};
```

---

### ✅ STEP 9: `routes/authRoutes.js`

```js
const express = require('express'); // Import express
const router = express.Router(); // Create router
const { signup, login } = require('../controllers/authController'); // Import controllers

router.post('/signup', signup); // Signup route
router.post('/login', login); // Login route

module.exports = router; // Export router
```

---

### ✅ STEP 10: `middleware/auth.js`

```js
const jwt = require('jsonwebtoken'); // Import JWT
const JWT_SECRET = process.env.JWT_SECRET;

module.exports = function (req, res, next) {
  const token = req.header('Authorization')?.split(' ')[1]; // Get token
  if (!token) return res.status(401).json({ msg: 'No token, authorization denied' });

  try {
    const decoded = jwt.verify(token, JWT_SECRET); // Verify token
    req.user = decoded; // Attach to request
    next(); // Continue
  } catch (err) {
    res.status(400).json({ msg: 'Invalid token' });
  }
};
```

---

## 🧠 Useful Mongoose Functions

```js
User.find();                        // All users
User.findById(id);                 // By ID
User.findOne({ email });           // Find one by filter
User.find({ age: { $gt: 20 } });   // Filter
User.create({ name, email });      // Create
User.findByIdAndUpdate(id, obj);   // Update
User.findByIdAndDelete(id);        // Delete
```

---

## 🧪 Postman Testing

- POST `/api/auth/signup` with JSON body
- POST `/api/auth/login` and save `token`
- Set `Authorization: Bearer TOKEN` for protected routes

---

## ✅ Deployment: MongoDB Atlas

1. Go to [https://mongodb.com/cloud/atlas](https://mongodb.com/cloud/atlas)
2. Create free cluster
3. Add user, IP whitelist
4. Get connection URI and update `.env`

```env
MONGO_URI=mongodb+srv://<username>:<pass>@cluster0.mongodb.net/myDB
```

---

---

### Body vs Params vs Query**

- **Body**: 
  - The body is used to send data in a POST, PUT, or PATCH request.
  - It is typically used for complex data (e.g., user details in JSON format).
  - In Express, you access the body using `req.body`.

  **Example**:
  ```json
  {
    "name": "John Doe",
    "email": "john@example.com"
  }
  ```
  When making a request like a POST to create a new user, you send this data in the body of the request.

- **Params**: 
  - URL parameters are part of the URL and are typically used for identifying a specific resource.
  - They are part of the route and can be accessed using `req.params`.

  **Example**:
  ```js
  app.get('/user/:id', (req, res) => {
    const { id } = req.params;
    res.send(`User ID: ${id}`);
  });
  ```

  **URL**: `/user/123`  
  In this case, `123` is a parameter passed as part of the URL to identify a specific user by their ID.

- **Query**: 
  - Query parameters are passed in the URL and are used to filter data or add additional options to the request.
  - They are part of the URL after the `?` and can be accessed via `req.query`.

  **Example**:
  ```js
  app.get('/search', (req, res) => {
    const { name } = req.query;
    res.send(`Searching for user: ${name}`);
  });
  ```

  **URL**: `/search?name=John`  
  In this case, `name=John` is a query parameter used to filter or search for users.

### **Key Differences**
- **Body**: Used for complex data (sent with POST, PUT).
- **Params**: Part of the URL, typically used for specific resources (e.g., `/user/:id`).
- **Query**: Used for optional filters or parameters that are added to the URL (e.g., `/search?name=John`).


---

https://youtu.be/vfaRzV3P92o?si=xQILZ0bGrGYD7PhR
Chapter 8 - ![image](https://github.com/user-attachments/assets/18b6f4b5-5e6b-44e0-b183-5020707beeb3)
Chapter 14 - ![image](https://github.com/user-attachments/assets/6e4a5f46-ac93-4fc7-9046-30b6c38e2b59)


