---
title: 策略模式
date: 2020-05-17 20:52:53
tags:
    - design parttern
    - js    
---

> 策略模式定义：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。


## 传统语言中的策略模式

```
var performanceS = function () { };
performanceS.prototype.calculate = function (salary) {
  return salary * 4;
};
var performanceA = function () { };
performanceA.prototype.calculate = function (salary) {
  return salary * 3;
};
var performanceB = function () { };
performanceB.prototype.calculate = function (salary) {
  return salary * 2;
};


var Bonus = function () {
  this.salary = null;
  this.strategy = null;
};

Bonus.prototype.setSalary = function (salary) {
  this.salary = salary; // 设置员工的原始工资
};

Bonus.prototype.setStrategy = function (strategy) {
  this.strategy = strategy; // 设置员工绩效等级对应的策略对象
};

Bonus.prototype.getBonus = function () { // 取得奖金数额
  return this.strategy.calculate(this.salary); // 把计算奖金的操作委托给对应的策略对象
};


var bonus = new Bonus();
bonus.setSalary(10000);
bonus.setStrategy(new performanceS()); // 设置策略对象
console.log(bonus.getBonus());

bonus.setStrategy( new performanceA() ); // 设置策略对象
console.log( bonus.getBonus() );
```


## js中的策略模式

```
var strategies = {
  "S": function (salary) {
    return salary * 4;
  },
  "A": function (salary) {
    return salary * 3;
  },
  "B": function (salary) {
    return salary * 2;
  }
};

var calculateBonus = function (level, salary) {
  return strategies[level](salary);
};
console.log(calculateBonus('S', 20000));
console.log(calculateBonus('A', 10000));
```