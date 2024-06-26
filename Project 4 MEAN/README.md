# MEAN STACK DEPLOYMENT (MySQL, Express, Angular, Node.js)

## General Overview

- MongoDB (Document database) – Stores and allows retrieval of data.

- Express (Back-end application framework) – Makes requests to Database for Reads and Writes.

- Angular (Front-end application framework) – Handles Client and Server Requests

- Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user

## Prerequisites

- Cloud Service Provider - AWS, Azure, GCP, etc.

- Launch a Linux Instance (Ubuntu preferably).

- Prior knowledge on how to SSH into a virtual host.

- Knowledge on how configure inbound rules for security groups using the AWS console.

## STEP 1: INSTALL NODEJS

`sudo apt update && sudo apt install nodejs npm`

![Alt text](<images/install npm.png>)

Once done, verify the installation by running:

`nodejs -v`

![Alt text](<images/node -v.png>)

**ALTERNATIVELY, WE COULD USE;**

Run the following command as a user with sudo privileges to download and execute the NodeSource installation script:  
`curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -`

The script will add the NodeSource signing key to your system, create an apt repository file, install all necessary packages, and refresh the apt cache.  
Once the NodeSource repository is enabled, install Node.js and npm:  
`sudo apt install nodejs`

![Alt text](<images/install nodejs.png>)

The nodejs package contains both the node and npm binaries.

To be able to compile native addons from npm you’ll need to install the development tools:  
`sudo apt install build-essential`

![Alt text](<images/install build essential.png>)

## STEP 2: INSTALL MONGODB
Run:

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

- Install MongoDB
`sudo apt install -y mongodb`

![Install-mongodb](./Images/Install-mongodb.png)

- Start The server
`sudo service mongodb start`

Verify that the service is up and running  
`sudo systemctl status mongodb`

![Start-mongodb](./Images/Start-mongodb.png)

- Install npm – Node package manager
`sudo apt-get install -y nodejs`

![install-npmnodejs](./Images/Install-nodenpm.png)

- Install body-parser package: `body-parser` package to help us process JSON files passed in requests to the server.
  
`sudo npm install body-parser`

![Install-parser](./Images/Install-bodyparser.png)

- Create a folder named ‘Books’  

`mkdir Books && cd Books`

- In the Books directory, Initialize npm project  
`npm init`
 
![Npm-init](./Images/npm%20init.png)

- Add a file to it named `server.js` 

`nano server.js`

- Copy and paste the web server code below into the server.js file.

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

![Server-js](./Images/Serverjs.png)

## STEP 3: INSTALL EXPRESS AND SET UP ROUTES

- Mongoose package which provides a straightforward, schema-based solution to model your application data will be used.

`sudo npm install express mongoose`

![install-Mongoose](./Images/Install-mongoose.png)

- In `Books` folder, create a folder named apps  

`mkdir apps && cd apps`

- Create a file named `routes.js`

`nano routes.js`

- Copy and paste the code below into `routes.js`

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

![Routes](./Images/Routesjs.png)

- In the `apps` folder, create a folder named `models`

`mkdir models && cd models`

- Create a file named `book.js`  
`nano book.js`

- Copy and paste the code below into `book.js`

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

![Booksjs](./Images/Bookjs.png)

## STEP 4 – ACCESS THE ROUTES WITH ANGULARJS

- Change the directory back to `Books`

`cd ../..`

- Create a folder named `public`

`mkdir public && cd public`

- Add a file named `script.js`

`nano script.js`

- Copy and paste the Code below (controller configuration defined) into the `script.js` file.

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

![Public-Scriptjs](./Images/Public-scriptjs.png)

- In the public folder, create a file named `index.html`;

`nano index.html`

- Copy and paste the code below into `index.html` file.

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

![Index-html](./Images/Indexhtml.png)

- Change the directory back up to `Books`

`cd ..`

- Start the server by running this command:

`node server.js`

![Node-serverjs](./Images/Nodeserverjs.png)

The server is now up and running, we can connect it via port 3300.

Go to `http://PublicIP-or-PublicDNS:3300` on your browser.

***A quick reminder on how to get your server’s Public IP and public DNS name:***

- You can find it in your AWS web console in EC2 details.

- Run `curl -s http://169.254.169.254/latest/meta-data/public-ipv4` for Public IP address.

- `curl -s http://169.254.169.254/latest/meta-data/public-hostname` for Public DNS name.

This is how your WebBook Register Application will look in the browser:

![Interface](./Images/interface.png)