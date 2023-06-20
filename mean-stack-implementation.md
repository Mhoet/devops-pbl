# WEB STACK IMPLEMENTATION - MEAN  (MongoDB, **ExpressJS**, Angular and Node.js) IN AWS
### Using AWS virtual machine EC2 t3.micro instance (same as in my previous [MERN STACK IMPLEMENTATION](https://github.com/Mhoet/devops-pbl/blob/main/mern-stack-implementation.md))

#### On this project I will be deploying a ``Book-Record`` App.

## STEPS (BACKEND)
- After updating and upgrading ubuntu, I added certificates using the below code:
-```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -```
![Screenshot 2023-05-28 125912](https://github.com/Mhoet/devops-pbl/assets/81827857/9a53c6d3-957f-4588-b61a-8a13be87c6bc)
-  I installed Node.Js and verified the installation using 
```sudo apt install -y nodejs```

![Screenshot 2023-05-28 130023](https://github.com/Mhoet/devops-pbl/assets/81827857/80d9f1ab-b49a-46c2-9492-334e60608418)
- To install mongodb, I started by adding a new **key** and **repo** for the version of mongodb I wanted to install.
 ```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6

echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```

![Screenshot 2023-05-28 130154](https://github.com/Mhoet/devops-pbl/assets/81827857/2093956e-6933-4c06-842c-61f60497344a)

![Screenshot 2023-05-28 130239](https://github.com/Mhoet/devops-pbl/assets/81827857/cfb3369a-6af2-4b59-b23b-9c58322454f3)
- Then, I attempted to install mongo db. I hit a blocker because my mongo db was failing to install until I found out that the installation steps I was following is for version of ubuntu (20.04) but for I had ubuntu (22.04)
- I uninstalled that and installed 20.04 because my project was based on this version. It was successful this time.
![Screenshot 2023-05-28 133340](https://github.com/Mhoet/devops-pbl/assets/81827857/53dcccf1-f78a-44cd-81fa-f6f942484d74)
- I started the mongo server, verified the service was running then, I stalled **node package manager** 
![Screenshot 2023-05-28 143626](https://github.com/Mhoet/devops-pbl/assets/81827857/65d33318-6bac-4ecb-924d-2a1ca98a95a8)
- I learned that I needed  ‘body-parser’ package to help process JSON files passed in requests to the server. So, I installed it using ```sudo npm install body-parser```
- I create a folder named **‘Books’** ```mkdir Books && cd Books```
- In the Books directory, I initialized a npm project ```npm init```
- I added a file to it named  `server.js`and added the below code to it
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
``` 
![Screenshot 2023-05-28 144227](https://github.com/Mhoet/devops-pbl/assets/81827857/3808057a-63a4-43c6-8d5d-b20248f700dc)
![Screenshot 2023-05-28 144314](https://github.com/Mhoet/devops-pbl/assets/81827857/fa495968-139e-4fd9-9695-f73a4dd1c0a3)
  
### INSTALLING EXPRESS AND SETTING UP ROUTES TO THE SERVER
- I installed express and mongoose ```sudo npm install express mongoose```
- In ‘Books’ folder, I created a folder named  `apps`  using ```mkdir apps && cd apps```
- I create a file named  `routes.js` ```vi routes.js``` and added the routing code to it.
![Screenshot 2023-05-28 144815](https://github.com/Mhoet/devops-pbl/assets/81827857/0814c127-cf2b-4530-8ebd-c5a26d5c3904)
Complete code below:
```
const Book = require('./models/book');

module.exports = function(app){
  app.get('/book', function(req, res){
    Book.find({}).then(result => {
      res.json(result);
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while retrieving books');
    });
  });

  app.post('/book', function(req, res){
    const book = new Book({
      name: req.body.name,
      isbn: req.body.isbn,
      author: req.body.author,
      pages: req.body.pages
    });
    book.save().then(result => {
      res.json({
        message: "Successfully added book",
        book: result
      });
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while saving the book');
    });
  });

  app.delete("/book/:isbn", function(req, res){
    Book.findOneAndRemove(req.query).then(result => {
      res.json({
        message: "Successfully deleted the book",
        book: result
      });
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while deleting the book');
    });
  });

  const path = require('path');
  app.get('*', function(req, res){
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
  });
};
```
- In the ‘apps’ folder, I created a folder named  `models` using ```mkdir models && cd models```
- I create a file named  `book.js` and added the schema code to it.
![Screenshot 2023-05-28 145044](https://github.com/Mhoet/devops-pbl/assets/81827857/16045ab7-811e-4c75-84af-2f0950118772)
### Accessing the routes with  [AngularJS](https://angularjs.org/)
- In the root directory (Books), I create a folder  ```mkdir public && cd public``` and added a file ```vi script.js```.
-  I added the **controller configuration** into the script.js file.
![Screenshot 2023-05-28 145218](https://github.com/Mhoet/devops-pbl/assets/81827857/df899ea6-70af-4149-bfef-3a7d39ad64f4) 
Complete code below:
```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
- In the  `public`  folder, create a file ```vi index.html``` and added the code to style it.
![Screenshot 2023-05-28 145341](https://github.com/Mhoet/devops-pbl/assets/81827857/08a76ed3-45b0-4420-a0f4-0ba8ed0d432a)
```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```
- In the **root** directory I started the server and set up port **3300** on the security group in AWS
![Screenshot 2023-05-28 145530](https://github.com/Mhoet/devops-pbl/assets/81827857/3284a75a-6aea-408e-9b80-8f32822b1d40)

### It was up and running successfully and I could ADD and DELETE books.

![Screenshot 2023-05-28 150626](https://github.com/Mhoet/devops-pbl/assets/81827857/5f497da3-2468-475c-87c0-9ced707517a4)


Credit: darey.io
