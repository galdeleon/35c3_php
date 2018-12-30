# 35c3_php
Solution for 35c3 php challenge

The challenge consists of a single php file. 

1. It reads the user input from STDIN.
2. It reads the content of the flag file into $flag variable.
3. It declares a class named 'B', with a destructor function that echos the content of $flag.
4. It uses PHP serialization library to unserialize the user input.
5. It throws an exception.


PHP serialization library is the source of many vulnerabilities. A common logical bug involves actions done from magic methods such as 
\__destruct.

We can serialize an instance of 'B' class, which means that its destructor will get called at some point and echo the content of the flag.

O:1:"B":0:{};

Still, that didn't work. This is because the exception that gets thrown in the next line. I'm not sure if this caused the destructor not to execute, or simply doesn't allow any more echo calls at that point since an exception is thrown.

To overcome this, I wanted to make sure the destruct method gets called prior to the exception being thrown. To do so, I crafted the following serialized string:

a:2:{i:1;O:1:"B":0:{};i:2;o:1:"C":0:{};}

This will unserialize an array, consisting of a 'B' object as its first item. For the second item, we will try to unserialize an object of class 'C' which isn't declared. That will throw an exception (which is silently avoided because of the '@' prior to the unserialize call). Still, the unserialize call already instantiated an instance of 'B' class thus it have to call its destruct method and echo the content of $flag.


gal@ubuntu:/var/www/html$ nc 35.242.207.13 1
a:2:{i:1;O:1:"B":0:{};i:2;o:1:"C":0:{};}
35C3_php_is_fun_php_is_fun
PHP Fatal error:  Uncaught Exception: Well that was unexpected… in /home/user/php.php:16
Stack trace:
#0 {main}
  thrown in /home/user/php.php on line 16
gal@ubuntu:/var/www/html$
