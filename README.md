# ng-fire-alarm
> Distributed via

[![Version     ](https://img.shields.io/gem/v/ng-fire-alarm.svg)                                    ](https://rubygems.org/gems/ng-fire-alarm)
[![Bower Version](https://badge.fury.io/bo/ng-fire-alarm.svg)                                       ](https://badge.fury.io/bo/ng-fire-alarm)

> Firebase binding use $q in AngularJS

[![Travis CI   ](https://travis-ci.org/tomchentw/ng-fire-alarm.svg?branch=master)                   ](https://travis-ci.org/tomchentw/ng-fire-alarm)
[![Quality     ](https://img.shields.io/codeclimate/github/tomchentw/ng-fire-alarm.svg)             ](https://codeclimate.com/github/tomchentw/ng-fire-alarm)
[![Coverage    ](https://img.shields.io/coveralls/tomchentw/ng-fire-alarm.svg)                      ](https://coveralls.io/r/tomchentw/ng-fire-alarm)
[![Dependencies](https://gemnasium.com/tomchentw/ng-fire-alarm.svg)                                 ](https://gemnasium.com/tomchentw/ng-fire-alarm)


## Project philosophy

### Develop in LiveScript
[LiveScript](http://livescript.net/) is a compile-to-js language, which provides us more robust way to write JavaScript.  
It also has great readibility and lots of syntax sugar just like you're writting python/ruby.

### Use new API from $q
We use newly introduced $q API to [`notify`](https://github.com/angular/angular.js/blob/master/CHANGELOG.md#120rc1-spooky-giraffe-2013-08-13) you about the *value/order* changes, we let you decide how you deal with your data with `$scope`.

### Firebase Collection support
We know that you want to take advantage of collection in `Firebase`, but still want to preserve the right order, or order by any properties in each item. `ng-fire-alarm` allows to transform collection **object** into native js array for you.

### Seperation of Concerns
We follow seperation of concerns by seperating control(reference) object and data source to [`Alarm`](https://github.com/tomchentw/ng-fire-alarm#alarm-object) object and [`Fire`](https://github.com/tomchentw/ng-fire-alarm#fire-objects) object(s). So a `ng-change`, `ng-submit`, `ng-click` can be directly bound to Alarm object, while your Fire object(s) are clean and just like what you see in Firebase dashboard.


## Installation

### Just use it

* Download and include [`ng-fire-alarm.js`](https://github.com/tomchentw/ng-fire-alarm/blob/master/ng-fire-alarm.js) OR [`ng-fire-alarm.min.js`](https://github.com/tomchentw/ng-fire-alarm/blob/master/ng-fire-alarm.min.js).  

Then include them through script tag in your HTML.

### **Rails** projects (Only support 3.1+)
Add this line to your application's Gemfile:

```ruby
gem 'ng-fire-alarm'
```

And then execute:

    $ bundle

Then add these lines to the top of your `app/assets/javascripts/application.js` file:

```javascript
//= require angular
//= require ng-fire-alarm
```

And include in your `angular` module definition:

```javascript
var module = angular.module('my-awesome-project', ['ng-fire-alarm']).
```


## Usage

We add a new method to `Firebase.prototype`:

### $toAlarm

The key to transform your `Firebase` reference into a `AngularJS` powered [alarm object](https://github.com/tomchentw/ng-fire-alarm/blob/master/README.md#alarm-object), it take one parameter:

#### Options

##### collection: _true/otherwise_.

Pass true will transform collection object into native js array for you.

#### `Alarm` object
A wrapped object over Firbease reference that is returned by calling `Firebase.prototype.$toAlarm`. 

##### Attributes:

* `$promise`: it will be notified everytime the value/order changes.

##### Convenience Method:

* `$thenNotify`: Same as `$promise.then(void, void, callback)`.  
Register a callback that notify you each time the alarm rings.  
Unlike `$promise.then`, this method is self-chained and will return alarm object to allow chaining.  
See examples below.

##### Query Methods:

Ther're wrapper for `Firebase.prototype.limit/startAt/endAt` function, but it'll update internal Firebase reference and it'll populate new data through your callback registered via `$thenNotify`.

* [$limit](https://www.firebase.com/docs/javascript/firebase/limit.html)
* [$startAt](https://www.firebase.com/docs/javascript/firebase/startat.html)
* [$endAt](https://www.firebase.com/docs/javascript/firebase/endat.html)

Examples:

```javascript
var ROOT = new Firebase('https://ng-fire-alarm.firebaseio.com/');

function UsersListCtrl ($scope) {
  $scope.users = [];

  $scope.usersAlarm = ROOT.child('users').$toAlarm({collection: true});

  $scope.usersAlarm.$thenNotify(function(users) {
    $scope.users = users;
  });
}
```

Fast prototyping!

```html
<ul ng-controller="UsersListCtrl" infinite-scroll="usersAlarm.$limit(users.length + 25)">
  <li ng-repeat="user in users">
    {{ user.name }}
  </li>
</ul>
```

##### Write Methods:
They're wrapper for `Firebase.prototype.remove/push/update/set/setPriority/setWithPriority` function, but it'll return a `promise` object instead of passing in a callback function.

* [$remove](https://www.firebase.com/docs/javascript/firebase/remove.html)
* [$push](https://www.firebase.com/docs/javascript/firebase/push.html)
* [$update](https://www.firebase.com/docs/javascript/firebase/update.html)
* [$set](https://www.firebase.com/docs/javascript/firebase/set.html)
* [$setPriority](https://www.firebase.com/docs/javascript/firebase/setpriority.html)
* [$setWithPriority](https://www.firebase.com/docs/javascript/firebase/setwithpriority.html)

Examples:

```javascript
var ROOT = new Firebase('https://ng-fire-alarm.firebaseio.com/');

function UserEditCtrl ($scope) {
  $scope.userAlarm = ROOT
    .child('users/`')
    .$toAlarm()
    .$thenNotify(function(user) {  $scope.user = user;  });
}
```

Fast prototyping!

```html
<form ng-controller="UserEditCtrl">
  <input type="text" ng-model="user.name" ng-change="userAlarm.$update(user)">
</form>
```


#### `Fire` object(s)
Object that is passed in to callbacks registered via `$thenNotify`, they can be **primitive, object, or array**:

##### primitive

Primitive is just js primitive. There's **NO** **NO** **NO** wrapper around primitive <del>`{$value: primitive}`</del>.

```javascript
bell.$thenNotify(function (aStringOrANumber) { $scope.myVar = aStringOrANumber; });
```

##### object
  
we've add two properties on it:  

  1. [`$name`](https://www.firebase.com/docs/javascript/datasnapshot/name.html)
  2. [`$priority`](https://www.firebase.com/docs/javascript/datasnapshot/getpriority.html)

##### array

sorted by native Firebase [ordering](https://www.firebase.com/docs/javascript/firebase/setpriority.html).  

  Each item in array, if they're object, will have these properties:

  1. [`$name`](https://www.firebase.com/docs/javascript/datasnapshot/name.html)
  2. [`$priority`](https://www.firebase.com/docs/javascript/datasnapshot/getpriority.html)
  3. [`$index`](https://www.firebase.com/docs/javascript/datasnapshot/foreach.html): object index in array. Useful for reverse ordering  

```html
<div ng-repeat="item in array | orderBy:'$index':true"></div>
```


## Contributing

[![devDependency Status](https://david-dm.org/tomchentw/ng-fire-alarm/dev-status.svg?theme=shields.io)](https://david-dm.org/tomchentw/ng-fire-alarm#info=devDependencies)

1. Fork it ( https://github.com/tomchentw/ng-fire-alarm/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request