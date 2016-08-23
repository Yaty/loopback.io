---
title: "Defining mixins"
lang: en
layout: page
keywords: LoopBack
tags:
sidebar: lb2_sidebar
permalink: /doc/en/lb2/Defining-mixins.html
summary:
---

**See also**:

* [loopback-example-mixins](https://github.com/strongloop/loopback-example-mixins)
* [loopback-ds-timestamp-mixin](https://www.npmjs.com/package/loopback-ds-timestamp-mixin)

## Overview

Use mixins to apply common logic to a set of models.
For example, a timestamp mixin could inject "created" and "modified" properties to model definitions.

You can apply mixins to any model, including [built-in models](/doc/en/lb2/Extending-built-in-models.html).

### Built-in model mixins

**Basic model**

By default, the basic LoopBack [Model object](http://apidocs.strongloop.com/loopback/#model) has properties and methods "mixed in" from:

* [Inclusion object](http://apidocs.strongloop.com/loopback-datasource-juggler/#inclusion) - Enables you to load relations of several objects and optimize numbers of requests.
* [Validateable object](http://apidocs.strongloop.com/loopback-datasource-juggler/#validatable) - provides validation methods.
  See [Validating model data](https://docs.strongloop.com/display/APIC/Validating+model+data).

When you define relations between models, the [RelationMixin object](http://apidocs.strongloop.com/loopback-datasource-juggler/#relationmixin) object also gets mixed in to the model object.

**Connected model**

In addition to the methods of the [Basic model object](https://docs.strongloop.com/display/APIC/Basic+model+object),
the following are mixed in when a model is connected to a data source:

* [RelationMixin class](http://apidocs.strongloop.com/loopback-datasource-juggler/#relationmixin)
* [PersistedModel class](http://apidocs.strongloop.com/loopback/#persistedmodel)

## Create a mixin script

Mixin scripts are JavaScript files in one of the following folders, depending on the scope of the mixin:

* `common/mixins/_mixinName_.js`, for example `common/mixins/timeStamp.js`.
* `server/mixins/_mixinName_.js`, for example `server/mixins/timeStamp.js`.

If the mixin applies to both client and server models, put it in the `common/mixins` directory.
If it applies only to server models, put it in the `server/mixins` directory.

{% include note.html content="

**Good to Know**

The above locations are just recommendations.
You are free to put mixin scripts in any project directory as long as you set the location with the `mixins` property in 
[`model-config.js`](https://docs.strongloop.com/display/public/LB/Defining+mixins#Definingmixins-Referencemixinsinmodel-config.js).

" %}

You can use mixins to perform different common actions on models such as observing changes using [operation hooks](/doc/en/lb2/Operation-hooks.html) and adding model attributes.

For example:

**common/mixins/timestamp.js**

```javascript
module.exports = function(Model, options) {
  // Model is the model class
  // options is an object containing the config properties from model definition
  Model.defineProperty('created', {type: Date, default: '$now'});
  Model.defineProperty('modified', {type: Date, default: '$now'});
}
```

Above `Timestamp` mixin adds two properties: `'created'` and `'modified'` of type: `date`, with default values set to the current date, to `Model.json`.

Where as, in the example below, the mixin observes the change using [`before save`](/doc/en/lb2/Operation-hooks.html) operation hook and manipulates input
(see complete example: [loopback-example-mixins](https://github.com/strongloop/loopback-example-mixins)):

**/server/mixins/squirrel.js**

```javascript
module.exports = function(Model, options) {
  'use strict';
  Model.observe('before save', function event(ctx, next) { //Observe any insert/update event on Model
    if (ctx.instance) {
      ctx.instance.squirrel = true;
    } else {
      ctx.data.squirrel = true;
    }
    next();
  });
};
```

## Reference mixins in model-config.js

The configuration file [`server/model-config.json`](/doc/en/lb2/model-config.json.html) specifies the list of directories to be searched for mixin scripts.
The default LoopBack configuration is:

**server/model-config.json**

```javascript
{
  "_meta": {
    "sources": [
      "loopback/common/models",
      "loopback/server/models",
      "../common/models",
      "./models"
    ],
    "mixins": [
      "loopback/common/mixins",
      "loopback/server/mixins",
      "../common/mixins",
      "./mixins"
    ]
  },
  ...
}
```

## Enable a model with mixins

To apply a mixin to a model, add "mixins" to the model definition JSON file. The value of mixins is an object keyed by normalized mixin names.
The value for each mixin is passed into the script as the options argument and these options are implemented by mixins.

**common/models/note.json**

```javascript
{
  "name": "note",
  "base": "PersistedModel",
  ...
  "mixins": {
     "Timestamp": {
       "myOption": 1,
       "anotherOpt": 2
     }
   },
  "properties": {
    ...
  },
  ...
}
```

In the example above, `common/mixins/timestamp.js` will be invoked with `(note, {"myOption": 1, "anotherOpt":2})`