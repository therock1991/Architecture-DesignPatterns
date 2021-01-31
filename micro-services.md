# Intro

There are 6 Data Management Patterns for Microservices

## Database Per Service

## Shared Database

## Saga Pattern

## API Composition

## CQRS

- https://medium.com/@letienthanh0212/cqrs-and-mediator-in-net-core-project-c0b477eab6e9

## Event Sourcing

# API Gateways and BFF

# Notfication

- https://blog.gojekengineering.com/how-we-manage-a-million-push-notifications-an-hour-549a1e3ca2c2?source=bookmarks---------26----------------------------

# Examples

## Timestamp Microservice

- You need to write a microservice that will return a JSON with the date in Unix format and in a human-readable date format. The JSON format is like the example output, “{“unix”:1451001600000, “utc”:“Fri, 25 Dec 2015 00:00:00 GMT”}”.

- The response depends on the URL. If the API endpoint is hit with no additional information, it returns the JSON with the current time.

- If the endpoint is hit with a date in unix format, it should calculate the human readable format, and vice versa.

```javascript
app.get("/api/timestamp/", (req, res) => {
  res.json({ unix: Date.now(), utc: Date() });
});

app.get("/api/timestamp/:date_string", (req, res) => {
  let dateString = req.params.date_string;

  //A 4 digit number is a valid ISO-8601 for the beginning of that year
  //5 digits or more must be a unix time, until we reach a year 10,000 problem
  if (/\d{5,}/.test(dateString)) {
    const dateInt = parseInt(dateString);
    //Date regards numbers as unix timestamps, strings are processed differently
    res.json({ unix: dateInt, utc: new Date(dateInt).toUTCString() });
  } else {
    let dateObject = new Date(dateString);

    if (dateObject.toString() === "Invalid Date") {
      res.json({ error: "Invalid Date" });
    } else {
      res.json({ unix: dateObject.valueOf(), utc: dateObject.toUTCString() });
    }
  }
});
app.get("/api/timestamp/", (req, res) => {
  res.json({ unix: Date.now(), utc: Date() });
});

app.get("/api/timestamp/:date_string", (req, res) => {
  let dateString = req.params.date_string;

  //A 4 digit number is a valid ISO-8601 for the beginning of that year
  //5 digits or more must be a unix time, until we reach a year 10,000 problem
  if (/\d{5,}/.test(dateString)) {
    const dateInt = parseInt(dateString);
    //Date regards numbers as unix timestamps, strings are processed differently
    res.json({ unix: dateInt, utc: new Date(dateInt).toUTCString() });
  } else {
    let dateObject = new Date(dateString);

    if (dateObject.toString() === "Invalid Date") {
      res.json({ error: "Invalid Date" });
    } else {
      res.json({ unix: dateObject.valueOf(), utc: dateObject.toUTCString() });
    }
  }
});
```

## Request Header Parser Microservice

```javascript
app.get("/api/parser", function (req, res) {
  var ip = req.ip;
  //get ip address with express build-in method req.ip

  var language = accepts(req).languages()[0];
  //get an array of preferred languages from http req headers, and take the first one, which is most preferred language

  var uaHeader = req.headers["user-agent"];
  //get user-agent string

  var agent = uaParser.parseOS(uaHeader).toString();
  //parse OS part of user-agent string to a string

  res.json({ ipaddress: ip, language: language, software: agent });
});
```

## Short Url

- Our project will need some npm packages and below is the list of
  those packages.
  - Express : A node.js framework that makes it easy to build web
    applications.
  - Mongodb : Official MongoDB driver for Node.js
  - Mongoose : An object modeling tool designed to work in an
    asynchronous environment. We will use mongoose to define database
    schemas and interact with the database.
  - Cors : CORS is a node.js package for providing a Connect/Express middleware that can be use to enable CORS with various options.
  - Body-parser :Node.js body parsing middleware. Parse incoming request bodies in a middleware before your handlers, available under the req.body property.
  - Valid-url : This module collects common URI validation routines to make input validation, and untainting easier and more readable.
  - Shortid : Amazingly short non-sequential url-friendly unique id generator.

```javascript
"use strict";

var express = require("express");
var MongoClient = require("mongodb").MongoClient;
var ObjectId = require("mongodb").ObjectId;
var mongoose = require("mongoose");
var shortId = require("shortid");
var bodyParser = require("body-parser");
var validUrl = require("valid-url");
require("dotenv").config();
var cors = require("cors");
var app = express();

// Basic Configuration
var port = process.env.PORT || 3000;

app.use(
  bodyParser.urlencoded({
    extended: false,
  })
);
app.use(cors());
app.use(express.json());

const uri = process.env.MONGO_URI;

mongoose.connect(uri, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  serverSelectionTimeoutMS: 5000, // Timeout after 5s instead of 30s
});

const connection = mongoose.connection;

connection.once("open", () => {
  console.log("MongoDB database connection established successfully");
});

app.use("/public", express.static(process.cwd() + "/public"));
app.get("/", function (req, res) {
  res.sendFile(process.cwd() + "/views/index.html");
});

//Create Schema
const Schema = mongoose.Schema;
const urlSchema = new Schema({
  original_url: String,
  short_url: String,
});
const URL = mongoose.model("URL", urlSchema);

app.post("/api/shorturl/new", async function (req, res) {
  const url = req.body.url_input;
  const urlCode = shortId.generate();

  // check if the url is valid or not
  if (!validUrl.isWebUri(url)) {
    res.status(401).json({
      error: "invalid URL",
    });
  } else {
    try {
      // check if its already in the database
      let findOne = await URL.findOne({
        original_url: url,
      });
      if (findOne) {
        res.json({
          original_url: findOne.original_url,
          short_url: findOne.short_url,
        });
      } else {
        // if its not exist yet then create new one and response with the result
        findOne = new URL({
          original_url: url,
          short_url: urlCode,
        });
        await findOne.save();
        res.json({
          original_url: findOne.original_url,
          short_url: findOne.short_url,
        });
      }
    } catch (err) {
      console.error(err);
      res.status(500).json("Server erorr...");
    }
  }
});

app.get("/api/shorturl/:short_url?", async function (req, res) {
  try {
    const urlParams = await URL.findOne({
      short_url: req.params.short_url,
    });
    if (urlParams) {
      return res.redirect(urlParams.original_url);
    } else {
      return res.status(404).json("No URL found");
    }
  } catch (err) {
    console.log(err);
    res.status(500).json("Server error");
  }
});

app.listen(port, () => {
  console.log(`Server is running on port : ${port}`);
});
```
## Exercise tracker
## File metadata
## References

- https://dzone.com/articles/6-data-management-patterns-for-microservices-1
