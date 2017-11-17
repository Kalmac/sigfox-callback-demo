<p align="center"><img src ="https://www.sigfox.com/themes/sigfox/logo.svg" width="300"></p>

## Sigfox Node.js Callback Demo

### Purpose

* Logs the message sent by your Sigfox objects
* Display a table of received messages, with their unique id, data payload and relevant metadata

This is a Node.js/Express application, with two routes:

* POST / sigfox to log a message
* GET / to display the dashboard


### Installation

#### Dependencies

Before installing the app itself, check that the main dependencies are installed on your system

##### Node.js

To install, the better is probably to use [nvm (Node version manager)](https://github.com/creationix/nvm) that will let you switch between version of Node.

```
$ curl https://raw.githubusercontent.com/creationix/nvm/v0.33.6/install.sh | bash
$ nvm install v8.9.1
$ nvm use v8.9.1
```

##### MongoDB

Follow the instructions on the [MongoDB website](https://www.mongodb.org/downloads).


##### Packages

* [express](http://expressjs.com) : Fast, unopinionated, minimalist web framework
* [body-parser](http://npmjs.com/body-parser) : Node.js body parsing middleware.
* [debug](http://npmjs.com/debug) : small debugging utility
* [mongojs](http://npmjs.com/mongojs) : Easy to use module that implements the mongo api
* [ejs](http://npmjs.com/ejs) : Embedded JavaScript templates
* [moment](http://npmjs.com/moment) : Parse, validate, manipulate, and display dates

#### Install

````
$ npm install
````

##Run

###MongoDB

Make sure you have mongo up & running :

```
$ sudo mongod
```


#### App
```
$ npm start
```

#### Env vars

You can set the following env vars to adjust your app's behaviour:

* `DEBUG` : Will filter the logs displayed in console. Check the [debug module](https://github.com/visionmedia/debug) documentation for details.
* `DATABASE_URL` : URL of the mongoDB database. Defaults to _mongodb://localhost:27017/sigfox-callback_
* `PORT`: the port your app will be listening to. Defaults to 34000

There are several ways of doing this, one is to create a config.app.js file in the same folder as server.js and paste this content into it : 

	'use strict';

	process.env.DEBUG = '*';
	process.env.DATABASE_URL = 'mongodb://DB_LOGIN:DB_PASSWORD@DB_URL;
	process.env.PORT = '4000'; 
	
Pay attention to the PORT: if you want to use a port lower than 1024 (for ex. 80), you must be root. One other option is to use setcap command to allow Node to bind to port 80 without root privilege. 

### Test requests

#### Check your dashboard

Navigate to http://localhost:34000/ in your browser.

There should be 0 message displayed at the start.

#### POST request

```
$ curl -X POST -d "connect=anything" http://localhost:34000/sigfox
```

You should get the following JSON response:
```
{"result":"♡"}
```

An entry will show up in your dashboard, with invalid data. This is because we didn't provide the full data structure of a Sigfox message.  

If you want to emulate a SIGFOX message, try:  

```
$ curl -X POST -d 'id=simulation&time=1500000000&station=future&data=d474' http://localhost:34000/sigfox
```

A message from the future should now appear on your local dashboard.

### Quick deploy on heroku

_Note:_ You can deploy this demo application wherever suited. Heroku is just a quickstart example.

#### One-Click Deployment

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/nicolsc/sigfox-callback-demo/tree/master)

#### Using CLI (Command Line Interface)

* Make sure you have installed the [Heroku Toolbelt](https://toolbelt.heroku.com/)
* Create an application : `heroku apps:create {whatever name}`. Documentation [here](https://devcenter.heroku.com/articles/creating-apps)
* Deploy your code : `$ git push heroku master`

#### Set up your env
* Add a [sandbox MongoLab add-on](https://elements.heroku.com/addons/mongolab#addon-docs) (free) : `$ heroku addons:add mongolab:sandbox`
* Set the `DATABASE_URL`env var to the URL of your mongo lab db
* `heroku config:get MONGOLAB_URI`
* `$ heroku config:set DATABASE_URL={your mongolab URL}`

All that remains to do is to set up your Sigfox callback on [the Sigfox backend](https://backend.sigfox.com)


#### How to set up a Sigfox callback

* Log into your [Sigfox backend](http://backend.sigfox.com) account
* In the _device type_ section, access to the device type of the object you want to track
* In the sidebar, click on the [Callbacks](http://backend.sigfox.com/devictype/:key/callbacks) option
* Click the _New_ button
* Set your callback as following
  * Type: DATA UPLINK
  * Channel: URL
  * Url syntax :   `http://{your URL}/sigfox?id={device}&time={time}&snr={snr}&station={station}&data={data}&avgSignal={avgSnr}&rssi={rssi}&lat={lat}&lng={lng}`
  * HTTP POST
  * _OK_
  
