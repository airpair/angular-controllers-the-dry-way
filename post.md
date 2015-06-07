##Introduction
As I started getting my hands dirty into my latest business application frontend built entirely in AngularJS plus some other smaller JavaScript libraries, I wanted more and more to share code among controllers. 

##Don't Repeat Yourself
DRY or "Don't Repeat Yourself" is a main principal of well written software applications, basically copying and pasting the same code snippets among different places in your application doubles the ripple effect where you have to change in so many places in response to one modification request. 

The typical advice was to move the shared code into a service and then inject this service into your controller and use it.

This method is fine, however, say that I have a directive that I will be using in more than one place in my application. 

This directive has the same functionality in all these places with a small variation each time. 

Of course, I don't want to create a big controller that handles all the use cases of my directives, what I want is a small lean controller for each directive that reuses the common code in the base service and then amends it to suit the needs of the directive.

##Template Method Design Pattern
One way to implement that is by using "Template Method" design pattern.  Template Method pattern defines a skeleton of the base controller function, and defers some steps to the individual controllers to implement. 

Template method design pattern not only separates the common code to the base controller but also forces specific structure to the client controllers which makes code more maintainable.

Also, in order to have different controllers for each instance of the directive we need to allow the directive to accept the controller as a parameter.

##How to implement this
In the following section of this post I am sharing with you how to do that.

1- Create template method as an Angular service. Use the service to create a constructor function so it can be instantiated and modified by different controllers.

```javascript
//Template Method

var app = angular.module("app", []);

app.service('templateCtrlSvc', function($log) {
  var instance = function() {
        this.init = function ($scope, $log) {
          throw ("you must implement init function");
        }
        this.step_one =  function ($log) {
          $log.info("default step one");
        }
        this.step_two = function($log) {$log.info('default step two ');}
        this.step_three = function($log) {
          $log.info("default step three");
        }
        this.step_four = function ($log) {
          $log.info("default step four");
        }
        this.close = function($log) {
          $log.info("default close \n");
        }
        this.run = function ($scope) {
          this.init($scope,$log);
          this.step_one($log);
          this.step_two($log);
          this.step_three($log);
          this.step_four($log);
          this.close($log);
        }
  }
  return instance;
});

```

2- Create the controllers you need and inject the template method service, here we will create two to demonstrate the idea.

```javascript
//Construct Controllers

app.controller('ctrl1', function($scope, templateCtrlSvc, $log) {
  
  var tmpMthd = new templateCtrlSvc();

});

  app.controller('ctrl2', function($scope, templateCtrlSvc, $log) {

  var tmpMthd = new templateCtrlSvc();

});
```

3- In each controller create create an object of the template method, and override the default functions implementation you wish to modify.

```javascript
//Second Controller

app.controller('ctrl1', function($scope, templateCtrlSvc, $log) {
  
  var tmpMthd = new templateCtrlSvc();
  tmpMthd.init = init;
  tmpMthd.step_two = step_two; 
  tmpMthd.step_four = step_four;
  tmpMthd.run($scope);
  
  function init($scope){
    $log.warn('ctrl1-init');
    $scope.buttonname = 'button one';
    $scope.clicked = function(){ alert('Hello from ' + $scope.buttonname + ' !');}
  }
  function step_two() {$log.warn("ctrl1-custom step two");}
});

  app.controller('ctrl2', function($scope, templateCtrlSvc, $log) {
  //var that = this;
  var tmpMthd = new templateCtrlSvc();
  tmpMthd.init = init;
  tmpMthd.step_four = step_four;
  tmpMthd.run($scope);
  
  function init($scope) {
    $log.warn('ctrl2-init');
    $scope.buttonname = 'button two'
    $scope.clicked = function(){ alert('Hi from ' + $scope.buttonname + '!');}
  }
  function step_four() {
    $log.warn('ctrl2-custom step four');
  }
});
```

4- Now create the directive and allow it to accept the controller name as a parameter.

```javascript
//Directive

app.directive( 'welcomeButton', function($log) {
    return {
      restrict: 'E',
      scope: {} , //isolated scope
      template: '<button ng-click="clicked()">click me</button>',
      controller: '@', //controller is passed as a parameter
      name: 'controllerName',
      link: function ($scope, $element, $attrs) {
          //
      }
    };
  });
```
5- Finally use the directive in your application, provide the controller name as an attribute to the directive.

```html

    <div>
      <welcome-button controller-name="ctrl1"></welcome-button>
    </div>
    <div>
      <welcome-button controller-name="ctrl2"></welcome-button>
    </div>


```

Here is a [plunker](http://plnkr.co/edit/hv2Kzl "Plunker") for a complete working sample. Please open the developer tools console to notice how both directives are constructed by using the common code from template method and how both override some of the steps to provide their own modification.

##Conclusion
Using Template Method design pattern provides structure and allows reusing code among Angular controllers.


