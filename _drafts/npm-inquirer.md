---
layout: post
title: inquirer
date: 2016-02-12 00:10:23.000000000 +09:00
type: post
categories:
- JavaScript
- NPM
---
> A collection of common interactive command line user interfaces
> (from [Inquirer.js](https://github.com/SBoudrias/Inquirer.js))

![](https://camo.githubusercontent.com/c60910b1ebcfd4448a986fa22d0d412b7f2d770c/68747470733a2f2f692e636c6f756475702e636f6d2f4c6352477058493043582d3330303078333030302e706e67)

## Usage
```js
/**
 * Input prompt example
 */

"use strict";
var inquirer = require("inquirer");

var questions = [
  {
    type: "input",
    name: "first_name",
    message: "What's your first name"
  },
  {
    type: "input",
    name: "last_name",
    message: "What's your last name",
    default: function () { return "Doe"; }
  },
  {
    type: "input",
    name: "phone",
    message: "What's your phone number",
    validate: function( value ) {
      var pass = value.match(/^([01]{1})?[\-\.\s]?\(?(\d{3})\)?[\-\.\s]?(\d{3})[\-\.\s]?(\d{4})\s?((?:#|ext\.?\s?|x\.?\s?){1}(?:\d+)?)?$/i);
      if (pass) {
        return true;
      } else {
        return "Please enter a valid phone number";
      }
    }
  }
];

inquirer.prompt( questions, function( answers ) {
  console.log( JSON.stringify(answers, null, "  ") );
});
```
