<p align="center">
  <h3 align="center">REST API with Slim, MongoDB, JWT </h3>
  <p align="center">a RESTful API boilerplate for Slim framework</p>
  <p align="center">
    <a href="https://travis-ci.org/me-io/slim-api">
      <img src="https://img.shields.io/travis/me-io/slim-api.svg?branch=master&style=flat-square" alt="Build Status">
    </a>
    <a href="LICENSE.md">
      <img src="https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square" alt="Software License">
    </a>
    <a class="badge-align" href="https://www.codacy.com/app/Meabed/slim-api">
      <img src="https://img.shields.io/codacy/grade/266923eec70e41418be8f981a5b4cefe.svg?style=flat-square"/>
    </a>        
    <a href="https://scrutinizer-ci.com/g/me-io/slim-api/?branch=master">
      <img src="https://img.shields.io/scrutinizer/g/me-io/slim-api/master.svg?style=flat-square" alt="Scrutinizer Code Quality">
    </a>
    <a href="https://codecov.io/gh/me-io/slim-api">
      <img src="https://img.shields.io/codecov/c/github/me-io/slim-api/master.svg?style=flat-square" alt="codecov">
    </a>
    <a href="https://packagist.org/packages/me-io/slim-api/">
      <img src="https://img.shields.io/packagist/dm/me-io/slim-api.svg?style=flat-square" alt="Packagist">
    </a>
    <a href="https://www.paypal.me/meabed">
      <img src="https://img.shields.io/badge/paypal-Buy_Me_Coffee-179BD7.svg?style=flat-squares" alt="Buy Me Coffee">
    </a>
  </p>
</p>

## Features
* API CRUD with MongoDB Collection persistence. " KIS & VS" "Keep It Simple and Very Stupid"
* JWT Authentication
* Middleware "CORS, JWT"
* Event Handling
* Endpoint Tests and Unit Tests
* CI - [Travis CI](https://travis-ci.org/)

## Directory Layout

```
.
├── app                    # Core code of the application.
│   ├── Base               # Contains base functionality of the framework
│   └── User               # Resource example for user
├── config                 # Framework configuration files are in this folder
│   └── settings.php       # Main configuartion of the framework.
├── public                 # Contains index.php file which is the entry point for requests
├── storage                # Contains the cache and files generated by the framework
│   └── logs               # Contains out application's logs files.
├── tests                  # Tests are placed in that folder.
├── vendor                 # Contains dependencies handled by composer.
├── CONTRIBUTING.md        # This file guide you how to contribute to this repo.
├── LICENSE.md             # License file for api.
├── README.md              # Introduces our api to user.
├── composer.json          # This file defines the project requirements
└── phpunit.xml            # Contains the configuration for PHPUnit.
```

## Getting Started

First, clone the repo:

```bash
git clone https://github.com/me-io/slim-api
```

### Install dependencies

```bash
cd slim-api
composer install
```

### Configure the Environment

Create `config/ext.settings.php` file:

```php
// override any default settings
return []
```

If you want you can edit database name, database username and database password.

### Run Application

To start making RESTful requests to slim-api start the PHP local server using:

```bash
php -S 0.0.0.0:3500 -t public
```

### Creating token

For creating token we have to use the http://localhost:3500/api/user/login route. Here is an example of creating token with [Postman](https://www.getpostman.com/).

![Imgur](https://i.imgur.com/dkFX1o4.png)

### Create A REST API

Run the following command to create a REST API:

```bash
composer create-project --prefer-dist Me-io/slim-api blog
```

#### Database connection

Out of the box slim-api supports MongoDB. So, to set up the connection with MongoDB create a new file `bootstrap/ext.settings.php` and configure the settings like this.

```php
return [
	'mongodb' => [
	'uri'      => 'mongodb://localhost:27017',
	'database' => 'blog', // Collection name
	]
]
```

#### Creating a New Resource

Creating a new resource is very easy and straight-forward. Follow these simple steps to create a new resource. The complete structure for a resource is like this:

```
.
+-- app
|  +-- Post
|      +-- api_routes.php
|      +-- PostController.php
|      +-- PostModel.php  
```

#### Step 1: Create Route

To create route for a resource create a new folder inside `app` folder. Then inside that resource folder create a new route file named `api_routes.php`. For example lets create routes for `Post` reource.

```php
$this->get('/posts', \App\Post\PostController::class . ":getAll");
$this->post('/posts', \App\Post\PostController::class . ":insertAndRetrieve");
$this->get('/posts/{id:[A-Z0-9a-z]+}', \App\Post\PostController::class . ":get");
$this->put('/posts/{id:[A-Z0-9a-z]+}', \App\Post\PostController::class . ":update");
$this->delete('/posts/{id:[A-Z0-9a-z]+}', \App\Post\PostController::class . ":delete");
```

For more info please visit Slim [Routing](https://www.slimframework.com/docs/objects/router.html) page.

#### Step 2: Create Model

Create a model `Post.php` inside `app/Post` directory.

```php
<?php

namespace App\Post;

use App\Base\Model\MongoDB;

class PostModel extends MongoDB
{
    /** 
    * @var string 
    */
    protected $collectionName = 'posts';
}
```

Visit [Mongo PHP](https://docs.mongodb.com/php-library/current/) to learn about how to query mongodb using php.

#### Step 3: Create Controller

Create a controller `PostController.php` inside `app/Post` directory.

```php
<?php

namespace App\Post;

use App\Base\Controller\RestController;

class PostController extends RestController
{
    protected $modelClass = PostModel::class;
}
```

### Events

Slim API events provides a simple observer implementation, allowing you to subscribe and listen for various events that occur in your application. To define your events for a Post resource lets create `_events.php` file inside `app/Post`.

```php
<?php

$events = [
    'post.created' => function ($event, ...$params) {
        // Do processing of the event.
    }
];
return $events;
```

Now lets emmit `post.created` event by the following code.

```php
<?php 

namespace App\Post;

use Slim\Http\Request;
use Slim\Http\Response;
use App\Base\Helper\Event;
use App\Base\Controller\RestController;

class PostController extends RestController
{
    protected $modelClass = PostModel::class;
	
	/**
     * @param Request $request
     * @param Response $response
     * @param $args
     * @return mixed
     * @throws \Exception
     */
	public function insert(Request $request, Response $response, $args) {
		// Create post and then retrieve it
		$post = [
		    "_id" 		  => "523b1153a2aa6a3233a91412",
		    "description" => "Buzzfeed asked a bunch of people...",
		    "title"       => "Cronut Mania: Buzzfeed asked a bunch of people...",
		];

		Event::emit('post.created', $post);
	}
}
```

## Contributing

Anyone is welcome to [contribute](CONTRIBUTING.md), however, if you decide to get involved, please take a moment to review the guidelines:

* [Only one feature or change per pull request](CONTRIBUTING.md#only-one-feature-or-change-per-pull-request)
* [Write meaningful commit messages](CONTRIBUTING.md#write-meaningful-commit-messages)
* [Follow the existing coding standards](CONTRIBUTING.md#follow-the-existing-coding-standards)

## License

The code is available under the [MIT license](LICENSE.md).
