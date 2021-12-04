---
title: AngularJS简单介绍
date: 2016-10-08 03:25
categories: 
    - [前端]
tags: [AngularJS,前端]
---

AngularJS是一款优秀的前端框架，主要解决hmtl在构建web应用上不足的问题。AngularJS核心的特点是模块化，数据的双向绑定，依赖注入。
## 模块化
在html标签中使用ng-app指示符定义一个模块
``` html
<html ng-app="mall-app" style="height: 100%;">                   
```
<!-- more -->
然后在js代码中即可通过下angular.module()方法创建指定的模块。
``` js
var mallApp = angular.module('mall-app', []);
```
在angular的设计中，一个module是多个服务，指示符，过滤器和配置信息的集合，一般来说就是一个app或者lib了。上面的方法还有一点要说明的就是一个参数的时候是获取已经存在的module
``` js
var mallApp = angular.module('mall-app');//获取之前创建的module对象
//多次调用两个参数的方法，将创建多个对象。
```
当然有时候你的module需要依赖其他module，在创建的时候需要在第二个参数中传入依赖module的名称
``` js
var mallApp = angular.module('mall-app', ['ngRoute']);//依赖官方的路由模块
```
这里额外说明一点,如果你未创建指定的module，却使用一个参数的方法去获取已经存在的module，angularjs会报错的。
``` js
var mallApp = angular.module('mall-app', []);//要先创建
var mallApp = angular.module('mall-app');//这里才能获取
```
## 数据的双向绑定
使用angularjs我们可以很方便的将数据绑定到html页面上，首先我们要先认识几个指示符
``` js
ng-app //之前说过的，定义一各module
ng-controler //定义一各控制器，我们可通过该控制器，控制html中的变量
ng-model //将元素值（比如输入框）绑定到变量中
```
然后呢，再介绍一下angularjs的表达式
``` html
{{ expression }}//也就是ng-bind
```
介绍完之后，我们就可以开始了
``` html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script src="http://apps.bdimg.com/libs/angular.js/1.4.6/angular.min.js"></script>
</head>
<body>
<div ng-app="myApp" ng-controller="myCtrl"> //定义module和控制器
<input ng-model="name">//使用ng-model指示符，直接将元素的值帮到变量上
<h1>我的名字是 {{name}}</h1>//表达式的变量
</div>
<script>
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope) {//获得控制器
    $scope.name = "John Dow";//在js中通过$scope获取html中的变量，并赋初值
});
</script>
<p>当你修改输入框中的值时，会影响到模型(model),当然也会影响到控制器对应的属性值。</p>
</body>
</html>
```
有兴趣的同学可以运行看一下，在输入框输入的值，会直接绑定到变量上，在hmtl上显示出来，当然name的值也会绑定到输入框中，这就是数据的双向绑定啦。
## 依赖注入
依赖注入是软体中的一个设计模式，这种模式通过分离了客户端依赖创建的行为，使程序变得松耦合。angularjs提供了很好的依赖注入机制，通过以下5个核心组件作为依赖注入。
- constants
  constant(常量)用来在配置阶段传递数值，注意这个常量在配置阶段是不可用的。
- value
  Value 是一个简单的 javascript 对象，用于向控制器传递值（配置阶段）
- factory
  factory 是一个函数用于返回值。在 service 和 controller 需要时创建。
  通常我们使用 factory 函数来计算或返回值。
- service
  同factory，与factory的区别是返回时使用new关键字
- provider
    AngularJS 中通过 provider 创建一个 service、factory等(配置阶段)。
Provider 中提供了一个 factory 方法 get()，它用于返回 value/service/factory。

下面是一些代码示例
``` html
<html>
   
   <head>
      <meta charset="utf-8">
      <title>AngularJS  依赖注入</title>
   </head>
   
   <body>
      <h2>AngularJS 简单应用</h2>
      
      <div ng-app = "mainApp" ng-controller = "CalcController">
         <p>输入一个数字: <input type = "number" ng-model = "number" /></p>
         <button ng-click = "square()">X<sup>2</sup></button>
         <p>结果: {{result}}</p>
      </div>
      
      <script src="http://apps.bdimg.com/libs/angular.js/1.4.6/angular.min.js"></script>
      
      <script>
         var mainApp = angular.module("mainApp", []);
         
         mainApp.config(function($provide) {
            $provide.provider('MathService', function() {
               this.$get = function() {
                  var factory = {};
                  
                  factory.multiply = function(a, b) {
                     return a * b;
                  }
                  return factory;
               };
            });
         });
			
         mainApp.value("defaultInput", 5);
         
         mainApp.factory('MathService', function() {
            var factory = {};
            
            factory.multiply = function(a, b) {
               return a * b;
            }
            return factory;
         });
         
         mainApp.service('CalcService', function(MathService){
            this.square = function(a) {
               return MathService.multiply(a,a);
            }
         });
         
         mainApp.controller('CalcController', function($scope, CalcService, defaultInput) {
            $scope.number = defaultInput;
            $scope.result = CalcService.square($scope.number);

            $scope.square = function() {
               $scope.result = CalcService.square($scope.number);
            }
         });
			
      </script>
      
   </body>
</html>
```
