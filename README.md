# MEAN-STACK-PROJECT
DEVOPS MASTERCLASS PROJECT 4 - MEAN STACK 

This is a project aimed at implementing  a simple Book Register web form using MEAN stack.
We start by installing NodeJS on an AWS EC2 Instance running Ubuntu Linux.
First we get the latest upgrade and update of Ubuntu by running the commands below from the shell:
```bash
sudo apt update

sudo apt upgrade
```
Then we added the certificates with the commands below:
```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
Then we proceeded to install NodeJs using the command below:
```bash
sudo apt install -y nodejs
```
Next we install MongoDb with the following sets of commands

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
Output of the above commands download the keys and MongoDB from online repository and is shown [here](https://user-images.githubusercontent.com/61512079/175114305-40b672ae-0f2f-4c41-b244-42c5c2c71eca.PNG).

Next we installed MongoDB using the downloaded package with the following command:
```bash
 sudo apt install -y mongodb
 ```
 While running the above command,  i encountered two errors:
 1. Unmet dependencies of ssl version 1.1. See [here](![image](https://user-images.githubusercontent.com/61512079/175122137-1fe0b6cf-6f2b-4e49-b9df-b24876c4c6a8.png)
    This was resolved by running the command below to install openssl 1.1:
    ```bash
    sudo apt-get install libssl1.1
    ```
 2. After resolving the dependency error, I got this error about the package mongodb not available.  This was resolved by changing the package name to mongodb-org as shown below:
  ```bash
  sudo apt install -y mongodb-org
 ```
 After installing the mongodb, the service is restarted with the command below:
 ```bash
 sudo service mongod start
 ```
 The service is verified to be up using the command:
 ```bash
 sudo systemctl status mongod
 ```
 [MongoDB runing status](https://user-images.githubusercontent.com/61512079/175124181-cafe2670-f806-48e7-9fae-30b28293d4b7.png)

Next is the installation of Node Package manager(npm):
```bash
sudo apt install -y npm
```
Another installation needed was ##body-parser## package to help us process JSON files passed in requests to the server.
```bash
sudo npm install body-parser
```
Next we create a folder 'Books' and changed directory to it as shown:
```bash
mkdir Books && cd Books
```
Then the npm was initialise as shown below:
```bash
npm init
```

Inside the book directory, we create a file server.js and copy-paste the code below into it:
```javascript
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
Next is the installation of Express and setting of routes to the server.  The Express will be used to pass book information to and from our MongoDB database. We also used Mongoose to establish a schema for the database to store data of our book register.
```bash
sudo npm install express mongoose
```
[Express and Mongoose Installation](https://user-images.githubusercontent.com/61512079/175132604-296925c4-80a4-42d1-a08b-25aee149214f.PNG")

Create a directory called 'apps' and change to it:
```bash
mkdir apps && cd apps
```
Inside the 'apps' directory, create a file called routes.js and paste the following codes in it:
```javascript
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
```

Also in the apps folder, create a directory called models and change to it:
```bash
mkdir models && cd models
```
Then in the 'models' directory, create another file 'book.js' and paste the code below to it:
```javascript
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```

Next we use ##AngularJS## to connect our web page with Express and perform actions on our book register.
Change directory back to Books and create another directory, public and change to it :
```bash
mkdir public && cd public
```
Then create a file script.js and paste the following code(controller configuration defined) into it:
```javascript
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
Also in the 'public' folder create another file 'index.html and add the following code to it:
```html
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

Finally, run the server with the following command:
```bash
node server.js
```

This is tested from the browser by opening: http://3.21.164.24:3300/  (where 3.21.164.24 is the IP address of the AWS EC2 Instance we are using)\
The test page opened correctly as shown [here](![image](https://user-images.githubusercontent.com/61512079/175143271-81d77c58-4497-496d-90fb-1a13f6747007.png)


