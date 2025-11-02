# Books Management System - Complete Setup Guide

## Features

-  Create, read, and delete books
-  MongoDB integration with Mongoose ODM
-  RESTful API endpoints
-  Static file serving
-  Express.js web framework



##  Setup Steps and Approach

### 1. Initial Environment Setup

### EC2 Instance Setup
Create an EC2 instance for which the whole architecture will be built upon
 I have made use of an Ubuntu EC2 machine
![aws-machine.png](https://github.com/GrailRoyal/MEAN-stack/blob/images/aws-machine.png)


# Update system packages

```bash
sudo apt-get update
```

# Install Node.js and npm (if not already installed)
```
sudo apt-get install -y nodejs npm
```
![EC2 set up image](https://github.com/GrailRoyal/MEAN-stack/blob/images/node.png)

# Install MongoDB
To install MongoDB, there are a few steps we need to follow because of the approach this project requires, we will be installing MongoDB directly on our machine
![EC2 set up image](https://github.com/GrailRoyal/MEAN-stack/blob/images/mongodb%20install.png)

#### Steps
This command Installs both GnuPG and curl on your system
```
sudo apt install -y gnupg curl
```
This command downloads MongoDB’s official GPG signing key and saves it to your system’s keyring directory in binary format, so that future MongoDB packages from their repository can be verified as authentic when you install or update them.
```
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg
```

Adds MongoDB’s official APT repository to Ubuntu’s package sources.
Updates the system package list so that Ubuntu is aware of MongoDB.
Installs MongoDB 7.0 and all related components.
```
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

```
The next command updates the package repository for Ubuntu so that the machine can update the new MongoDB repository added.
```
sudo apt-get update
```
This is to Install MongoDB onto our Ubuntu Machine
sudo apt-get install -y mongodb-org


# Start MongoDB service

Checking the Status of our MongoDB to know the state,  as well as starting the MongoDB so that the status can move from inactive to Active and Running
```
sudo systemctl start mongod
sudo systemctl enable mongod
```
[testing mongodb.png](https://github.com/GrailRoyal/MEAN-stack/blob/images/testing%20mongodb.png)



### 2. Project Dependencies Installation

```bash
# Install Express and body-parser
sudo npm install body-parser

# Install additional dependencies
npm install express mongoose
```

### 3. Project Structure Creation

```bash
# Create project directory structure
mkdir -p Books/apps Books/models Books/public
cd Books

# Create all necessary files
touch server.js, apps/routes.js models/book.js
```

##  File Contents

### `server.js`
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3300;

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/test', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.listen(PORT, () => {
  console.log(`Server up: http://localhost:${PORT}`);
});
```

### `apps/routes.js`
```javascript
const Book = require('../models/book');

module.exports = function(app) {
  app.get('/book', async (req, res) => {
    try {
      const books = await Book.find();
      res.json(books);
    } catch (err) {
      res.status(500).json({ message: 'Error fetching books', error: err.message });
    }
  });

  app.post('/book', async (req, res) => {
    try {
      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });
      const savedBook = await book.save();
      res.status(201).json({
        message: 'Successfully added book',
        book: savedBook
      });
    } catch (err) {
      res.status(400).json({ message: 'Error adding book', error: err.message });
    }
  });

  app.delete('/book/:isbn', async (req, res) => {
    try {
      const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
      if (!result) {
        return res.status(404).json({ message: 'Book not found' });
      }
      res.json({
        message: 'Successfully deleted the book',
        book: result
      });
    } catch (err) {
      res.status(500).json({ message: 'Error deleting book', error: err.message });
    }
  });
};
```

### `models/book.js`
```javascript
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true,
        trim: true
    },
    isbn: {
        type: String,
        required: true,
        unique: true
    },
    author: {
        type: String,
        required: true,
        trim: true
    },
    pages: {
        type: Number,
        required: true,
        min: 1
    }
}, {
    timestamps: true
});

module.exports = mongoose.model('Book', bookSchema);
```


##  Running the Application

```bash
# Navigate to project directory
cd ~/Books

# Start MongoDB service
sudo systemctl start mongod

# Start the application
node server.js
```
![app lunch](https://github.com/GrailRoyal/MEAN-stack/blob/images/app%20lunch.png)


##  Verification

### 1. Check server status
Visit `http://your-server-ip:3300`

![book-app](https://github.com/GrailRoyal/MEAN-stack/blob/images/book-app.png)


### 2. Test API endpoints
```bash
# Get all books
curl http://localhost:3300/book

# Create a book
curl -X POST http://localhost:3300/book \
  -H "Content-Type: application/json" \
  -d '{"name":"Sample Book","isbn":"1234567890","author":"John Doe","pages":300}'

# Delete a book
curl -X DELETE http://localhost:3300/book/1234567890
```

##   Troubleshooting


###  Issue 1: Module Not Found Errors

**Error**: 
```
Error: Cannot find module './apps/routes'
Require stack: /home/ubuntu/Books/server.js
```

**Solution**:
```bash
# Create missing directory structure
mkdir -p apps models
touch apps/routes.js models/book.js
```

**Error**: 
```
Error: Cannot find module './models/book'
Require stack: /home/ubuntu/Books/apps/routes.js
```

**Solution**:
```bash
# Create the book model file with proper content
cat > models/book.js << 'EOF'
const mongoose = require('mongoose');
const bookSchema = new mongoose.Schema({
    name: { type: String, required: true },
    isbn: { type: String, required: true, unique: true },
    author: { type: String, required: true },
    pages: { type: Number, required: true }
});
module.exports = mongoose.model('Book', bookSchema);
EOF
```

###  Issue 2: PathError - Catch-All Route Conflict

**Error**: 
```
PathError [TypeError]: Missing parameter name at index 1: *
visit https://git.new/pathToRegexpError for info
```

**Root Cause**: The catch-all route `app.get('*')` was defined in both `routes.js` and `server.js`, causing route conflicts and syntax issues.

**Debugging Steps**:
1. **First encountered** in `routes.js` at line 47
2. **Moved catch-all route** to `server.js`
3. **Still encountered error** in `server.js` at line 23
4. **Discovered** the route was defined in both files

**Final Solution**:
```javascript
// REMOVED from routes.js
// app.get('*', (req, res) => {
//   res.sendFile(path.join(__dirname, '../public', 'index.html'));
// });

// OPTIONAL: Add to server.js only (commented out for now)
// app.get('*', (req, res) => {
//   res.sendFile(path.join(__dirname, 'public', 'index.html'));
// });
```

###  Issue 3: Directory Execution Problems

**Error**: 
```
Error: Cannot find module '/home/ubuntu/Books/public/server.js'
```

**Cause**: Running server from wrong directory

**Solution**:
```bash
# Navigate to correct directory
cd ~/Books
node server.js
```

###  Issue 4: AWS EC2 Instance Limits

**Error**: 
```
Request limit exceeded. Account 546769761885 has been throttled on ec2:RunInstances
```

**Solution**:
- Used existing EC2 instance instead of creating new ones
- Monitored AWS service quotas
- Implemented efficient resource usage

##  Summary

1. **Setup Environment**: Install Node.js, MongoDB, and dependencies
2. **Create Project Structure**: Set up directories and files as shown above
3. **Configure MongoDB**: Ensure service is running and connected
4. **Avoid Route Conflicts**: Keep catch-all routes in server.js only
5. **Run Application**: Start from correct directory with `node server.js`

---

This project demonstrates a complete full-stack JavaScript application with backend API, database integration, and proper error handling throughout the development process. The troubleshooting journey highlights common issues developers face when setting up Node.js/Express/MongoDB applications.
