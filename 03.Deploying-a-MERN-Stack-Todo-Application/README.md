# Deploying a MERN Stack Application on AWS Cloud

A `MERN` stack consists of a collection of technologies used for the development of web applications. It comprises of MongoDB, Express, React and Node which are all JavaScript technologies used for creating full-stack applications and dynamic websites.

**MongoDB**: is a document-oriented NoSQL database technology used in storing data in the form of documents.

**React**: a javascript library used for building interactive user interface based on components.

**Express**: its a web application framework of Node js which is used for building server-side part of an appication and also creating Restful APIs.

**Node**: Node.js is an open-source, cross-platform runtime environment for building fast and scalable server-side and networking applications.

We will be building a simple todo list application and deploying on AWS cloud EC2 machine.

## Creating an Ubuntu EC2 Instance
Login to AWS Cloud Service console and create an Ubuntu EC2 instance.

Login into the instance via ssh:
```
ssh -i <private_keyfile.pem> username@ip-address
```
## Configuring Backend

Update all default ubuntu dependencies to ensure compatibility during package installation.

Run:
```
sudo apt update
sudo apt upgrade -y
``` 

Next up will be to install nodejs, first we get the location of nodejs form the ubuntu repository using the following command.
```
curl -fsSL https://deb.nodesource.com/setup_21.x | sudo -E bash -
```
 
Then we run a node install
```
sudo apt-get install -y nodejs
```
![node_installation](./img/1.nodejs_install.png)

### **Setting up the application**
We then create a directory that will house our codes and packages and all subdirectories to represent components of our application.
```
mkdir todo
```
Inside this directory we will instantiate our project using `npm init`. This enables javascript to install packages useful for spinning up our application.
```
npm init
```

![npm_installation](./img/2.npm_init.png)

## Express installation
We will be installing express which is nodejs framework and will be helpful when creating routes for our application via HTTP requests.<br />
```
npm install express
```
![express_installation](./img/3.npm_express.png)

Create an `index.js` file which will contain code useful for spinning up our express server
```
touch index.js
```

Install the `dotenv` module which is a module that loads environment variables from a `.env` file into `process.env`. The `.env` files are useful for hiding important credentials which shouldnt be exposed.
```
npm install dotenv
```
![npm_dotenv](./img/4.npm_dotenv.png)

Open the `index.js` file 
```
vim index.js
```
Paste in the code below.
```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

```

To spin up our server, in the same directory as your `index.js` file, run: 
```
node index.js
```

This code is useful for spinning up our application via the port specified in the code.

Allow our port `(5000)` as part of the inbound rules for the security group attached to our EC2 instance to ensure that our server is accessible via the internet.

Paste our public ip address on the browser with the port to see if the server is properly configured.
![hosted_express](./img/5.express_running.png)

## Defining Routes For our Application
We will create a `routes` folder which will contain code pointing to the three main endpoints used in our todo application. 

There are three actions that our To-Do application needs to be able to do:

1. Create a new task
2. Display list of all tasks
3. Delete a completed task

Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.

The `POST, GET, DELETE` requests will be helpful in interacting with our client_side and database via restful apis.

```
mkdir routes
cd routes
```
Create a `api.js` file
```
vim api.js
```
Paste the code below in `api.js` file. It is an example of a simple route that fires various endpoints. 

```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

## Creating models
We will be creating the models directory which will be used to define our database schema. A `schema` is a blueprint of how our database will be structured which include other fields which may not be required to be stored in the database. These are known as *virtual properties*.

To create a `schema` and a `model`, install `mongoose` which is a Node.js package that makes working with mongodb easier.

Switch to `todo` directory, and run:
```
npm install mongoose
```

![mongoose](./img/6.npm_mongoose.png
)

Create a `models` directory, and then create a `todo.js` file in it .
```
mkdir models && cd models && touch todo.js
```
Open the `todo.js` file
```
vim todo.js
```
Paste the code below inside the `todo.js` file
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```

Since we have defined a schema for how our database should be structured, we then update the code in our `api.js` to fire specific actions when an endpoint is called.

In `routes` directory open `api.js`, and delete the code inside with :%d command
```
vim api.js
```
Then paste the code below into it
```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```