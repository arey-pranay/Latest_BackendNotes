# Latest_BackendNotes

https://www.udemy.com/course/backend-master-class-golang-postgresql-kubernete

### dbdiagram.io
![image](https://github.com/user-attachments/assets/61684fe7-380a-454a-b3be-2695a0523356)


# 🔥 MongoDB Backend Mastery with Node.js, Express & Next.js

This guide walks you through setting up a complete backend with MongoDB using both Node.js/Express and Next.js API routes. It includes CRUD, Authentication (JWT), and MongoDB Atlas integration.

---

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

This README contains detailed step-by-step explanation and line-by-line comments. You can now run a full-stack backend project with MongoDB and Node.js confidently. 🎉
