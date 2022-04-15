# MEAN STACK DEPLOYMENT TO UBUNTU IN AWS
## Step 1: Install NodeJs
Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.

Update ubuntu

```javascript 
sudo apt update 
```
Upgrade ubuntu

```javascript 
sudo apt upgrade
```
Add certificates

```javascript 
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
```

```javascript 
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
Install NodeJS

```javascript 
sudo apt install -y nodejs
```
Step 2: Install MongoDB
MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.
mages/WebConsole.gif

```javascript 
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
Install MongoDB

```javascript 
sudo apt install -y mongodb
```
Start The server

```javascript 
sudo service mongodb start
```


Verify that the service is up and running

```javascript 
sudo systemctl status mongodb
```
Install npm – Node package manager.

```javascript 
sudo apt install -y npm
```

Install body-parser package

We need ‘body-parser’ package to help us process JSON files passed in requests to the server.

```javascript 
sudo npm install body-parser
```

Create a folder named ‘Books’

```javascript 
mkdir Books && cd Books
```

In the Books directory, Initialize npm project

```javascript
npm init
```

Add a file to it named server.js

```javascript 
vi server.js
```
Copy and paste the web server code below into the server.js file.

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

## Screenshot
<img width="1800" alt="Screenshot 2022-04-06 at 00 06 39" src="https://user-images.githubusercontent.com/33035619/163558840-40b7b2e3-1ca1-4bcd-b4f7-0b23cd05fd93.png">
<img width="1800" alt="Screenshot 2022-04-07 at 09 36 14" src="https://user-images.githubusercontent.com/33035619/163558847-97a54d79-0982-4eac-b18e-159e6bf3bbbe.png">



## INSTALL EXPRESS AND SET UP ROUTES TO THE SERVER
### Step 3: Install Express and set up routes to the server
Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.

We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.

```javascript 
sudo npm install express mongoose
```
In ‘Books’ folder, create a folder named apps

```javascript 
mkdir apps && cd apps
```
Create a file named routes.js

```javascript 
vi routes.js
```

Copy and paste the code below into routes.js

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

In the ‘apps’ folder, create a folder named models

```javascript 
mkdir models && cd models
```

Create a file named book.js

```javascript 
vi book.js
```

Copy and paste the code below into ‘book.js’

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

### Step 4 – Access the routes with AngularJS
AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.

Change the directory back to ‘Books’

```javascript 
cd ../
```
Create a folder named public

```javascript 
mkdir public && cd public
```

Add a file named script.js

```javascript 
vi script.js
```

Copy and paste the Code below (controller configuration defined) into the script.js file.

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

In public folder, create a file named index.html;

```javascript 
vi index.html
```
Cpoy and paste the code below into index.html file.

```javascript 
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

Change the directory back up to Books

```javascript 
cd ..
```

Start the server by running this command:

```javascript 
node server.js
```

The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what curl command returns locally.

```javascript 
curl -s http://localhost:3300
```

It shall return an HTML page, it is hardly readable in the CLI, but we can also try and access it from the Internet.

For this – you need to open TCP port 3300 in your AWS Web Console for your EC2 Instance.

You are supposed to know how to do it, if you have forgotten – refer to Project 1 (Step 1 — Installing Apache and Updating the Firewall)

Your Sercurity group shall look like this:



Now you can access our Book Register web application from the Internet with a browser using Public IP address or Public DNS name.

Quick reminder how to get your server’s Public IP and public DNS name:

You can find it in your AWS web console in EC2 details
Run curl -s http://169.254.169.254/latest/meta-data/public-ipv4 for Public IP address or curl -s http://169.254.169.254/latest/meta-data/public-hostname for Public DNS name.
This is how your Web Book Register Application will look like in browser:



Congratulations!
You have now completed all ‘PBL Progressive’ projects and are ready to move on to more complex and fun ‘PBL Professional’ projects!!!

## Screenshots 

<img width="1800" alt="Screenshot 2022-04-15 at 11 32 42" src="https://user-images.githubusercontent.com/33035619/163561676-2afcd356-efe1-44e7-9753-ed122d90af3d.png">
-2260fb0a5773.png">
>
     
<img width="1800" alt="Screenshot 2022-04-15 at 11 22 16" src="https://user-images.githubusercontent.com/33035619/163561591-90b7f0c7-1872-4282-8412-2260fb0a5773.png">

<img width="1800" alt="Screenshot 2022-04-15 at 11 37 51" src="https://user-images.githubusercontent.com/33035619/163561563-e06ae458-8404-4e85-b15c-d1244715f9ca.png">

