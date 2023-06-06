
# WEB STACK IMPLEMENTATION - MERN  (MongoDB, **ExpressJS**, React and Node.js) IN AWS
### Using AWS virtual machine EC2 t3.micro instance (same as in my previous [LEMP STACK IMPLEMENTATION](https://github.com/Mhoet/devops-pbl/blob/main/lemp-stack-implmentation.md))

#### On this project I will be deploying a ``To-Do`` App.

## STEPS RECREATED (BACKEND)
- I got the location of Node.js software from [Ubuntu repositories](https://github.com/nodesource/distributions#deb). using the below code:
```curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -```
-  I installed Node.Js and verified the installation using 
```sudo apt-get install -y nodejs```, ```npm -v ```, ```node-v ```

-  I then created a new directory for the project using 
```mkdir Todo```
-  In the Todo directory, I initialized ```npm init``` and created the package.json file
![Screenshot (182)](https://github.com/Mhoet/devops-pbl/assets/81827857/999b9931-b513-4b32-befe-25da3324eb37)

![Screenshot (183)](https://github.com/Mhoet/devops-pbl/assets/81827857/d8d02a2f-1e4a-4a02-89e1-fd55ab578621)
- Next, I Installed `ExpressJs` and created a`Routes` directory using npm:
```npm install express```
- I created a file  `index.js`  with the command below
- I installed the  `dotenv`  module
![Screenshot (184)](https://github.com/Mhoet/devops-pbl/assets/81827857/54ee64a1-b3c0-493d-844e-1fc7a773d507)

- In the index.js file I configured  my application's startup and routing. 
- I updated my security group on AWS to for port 5000
![Screenshot (185)](https://github.com/Mhoet/devops-pbl/assets/81827857/a411e2a8-ddee-44aa-83fe-494082e934e5)
- Then, I started my server and confirmed it was working as expected.
![Screenshot (186)](https://github.com/Mhoet/devops-pbl/assets/81827857/f635ae5a-9844-40d2-885f-a9eca28995e1)

![Screenshot (187)](https://github.com/Mhoet/devops-pbl/assets/81827857/030e8fcc-3214-44bf-a1e4-439cefb0bec9)
- Next, I created a directory**routes** and a file **api.js** to set up APIs for:
	- Creating a new task - POST
	- Displaying list of all tasks - GET
	- Deleting a completed task - DELETE
![Screenshot 2023-05-22 161227](https://github.com/Mhoet/devops-pbl/assets/81827857/cf09d584-749a-41c8-9d9a-de1cc0739a91)
![Screenshot 2023-05-22 161000](https://github.com/Mhoet/devops-pbl/assets/81827857/1959bff2-3e41-4dd6-8c94-775f08d6f6c8)
- I then went back to the Todo Folder, installed Mongoose, created the models directory and a file todo.js.

![Screenshot 2023-05-22 161451](https://github.com/Mhoet/devops-pbl/assets/81827857/176a6f45-cca3-4a93-9aa7-9fc86c581f2a)
- In the todo file, I created a mongoose schema for my To-Do App.
![Screenshot 2023-05-22 161842](https://github.com/Mhoet/devops-pbl/assets/81827857/7c70165d-a3e6-47e8-b394-a2c2ffd992e7)
- In the routes directory and in the api.js file, I created the algorithm for the POST, GET, DELETE
![Screenshot 2023-05-22 162412](https://github.com/Mhoet/devops-pbl/assets/81827857/b78f6c52-84b2-4ae1-bce4-5a3c9aa6145f)
My screenshot does not have the complete code, here's the code below:
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

- Next, I created an account on [mongodb.com](https://www.mongodb.com/atlas-signup-from-mlab) with AWS as my cloud provider
-  I built a database on it and connected it by adding the connection string to the .env file in my project.
![Screenshot 2023-05-22 164932](https://github.com/Mhoet/devops-pbl/assets/81827857/eb6bdacc-76a4-4c2e-a34b-02ae5eeeb32c)
- I updated the index.js file to properly connect to my database
![Screenshot 2023-05-23 164926](https://github.com/Mhoet/devops-pbl/assets/81827857/8bf7946f-34b9-482b-b1d1-597e664510e4)
```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
- I restarted my server and my database was connected successfully
![Screenshot 2023-05-23 170620](https://github.com/Mhoet/devops-pbl/assets/81827857/f576607c-935d-4681-b72b-e73d528a503a)
- After creating 3 items on my To-Do list, I was able to DELETE 1 and GET the remaining 2 using **POSTMAN**.
![Screenshot 2023-05-24 124938](https://github.com/Mhoet/devops-pbl/assets/81827857/65dd116a-c976-4537-a12f-d54dd79ae7b6)

## STEPS RECREATED (FRONTEND)
- In the Todo directory, I created a new directory **client** and in it the react-app
![Screenshot 2023-05-24 142301](https://github.com/Mhoet/devops-pbl/assets/81827857/040abf58-2345-48a0-8ece-28cf2cdcbe3c)
- I installed  [concurrently](https://www.npmjs.com/package/concurrently) so that I could run more than one command simultaneously from the same terminal window.
- I also installed  [nodemon](https://www.npmjs.com/package/nodemon). so that when there is any change in the server code, nodemon will restart it automatically and load the new changes.
It helped me ran and monitored the server.
![Screenshot 2023-05-24 142417](https://github.com/Mhoet/devops-pbl/assets/81827857/8c5a6e38-dda6-4e69-88f6-14b615c7343a)
- In the Todo directory, I updated the scripts section of my package.json file to set **start** and  nodemon **watch**
![Screenshot 2023-05-24 143001](https://github.com/Mhoet/devops-pbl/assets/81827857/75097fff-47e1-4e8b-927d-30be60a8307d)
- In the client directory and in the package.json file, I added proxy for localhost:5000
![Screenshot 2023-05-24 143300](https://github.com/Mhoet/devops-pbl/assets/81827857/b6377b3a-6d3d-4dc2-a27a-d7c12271d829)
- In the root (Todo) directory, I started my app with ```npm run dev``` and it was successful
![Screenshot 2023-05-24 151521](https://github.com/Mhoet/devops-pbl/assets/81827857/be38f5fa-16e2-49ab-a1cd-0d2c496b5b8e)
- In the** client/src** directory I created a **components** directory and three files, ListTodo.js, Todo.js and Input.js. 
![Screenshot 2023-05-24 152525](https://github.com/Mhoet/devops-pbl/assets/81827857/4714d03d-d584-46bd-8a5f-a7cb0b944f27)
- I added the below code for adding task to input.js
![Screenshot 2023-05-24 152538](https://github.com/Mhoet/devops-pbl/assets/81827857/708bcea6-782e-47e9-b20f-db8f2e869619)
Complete code below: 
```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
```
- In the **src** directory, I installed axios using ```npm install axios```
- In the **ListTodo.js** I added code to delete tasks.
![Screenshot 2023-05-24 153005](https://github.com/Mhoet/devops-pbl/assets/81827857/47f9af6d-0bdc-4637-afc0-0ff24ff15d70)

- In the **Todo.js** I added the code for managing the state of the To-Do actions.
![Screenshot 2023-05-24 153456](https://github.com/Mhoet/devops-pbl/assets/81827857/31c1defc-1af6-41eb-9fdd-dc0828019b7a)
Complete code below:
```
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
```
- In the **src* directory and in App.js file I added the below code for startup.
- 
![Screenshot 2023-05-24 153743](https://github.com/Mhoet/devops-pbl/assets/81827857/0ef15777-f11b-4ffc-a3f2-d5a6848618eb)
- In the **src* directory and in App.css file I added the below code for styling.
![Screenshot 2023-05-24 153925](https://github.com/Mhoet/devops-pbl/assets/81827857/cb4cd8d2-b0f7-4c9f-9441-21e004c37fd0)
Complete code below:
```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```
- In the **src* directory and in index.css file I added the below code for styling.
![Screenshot 2023-05-24 154033](https://github.com/Mhoet/devops-pbl/assets/81827857/62a778e9-b858-49a5-a496-513a31ec9348)
### Using the command ```npm run dev```, I restarted my app and it was up and running successfully
![Screenshot 2023-05-24 155105](https://github.com/Mhoet/devops-pbl/assets/81827857/ce95fa82-d926-4989-819f-650aba3cd6fd)

![Screenshot 2023-05-28 124627](https://github.com/Mhoet/devops-pbl/assets/81827857/2a8e1e26-3309-48a4-b4d3-56c116d1c985)

Credit: darey.io
