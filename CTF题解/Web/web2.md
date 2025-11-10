【攻防世界】Web新手题 Web2

题目来源：CTF

题目描述：解密

题解过程：

​	1、不用想，直接代码审计，一定要读懂代码的运行流程，看不懂就要去学，没有什么是天生就会的。

题目源代码：

```php
<?php
$miwen="a1zLbgQsCESEIqRLwuQAyMwLyq2L5VwBxqGA3RQAyumZ0tmMvSGM2ZwB4tws";

function encode($str){
    $_o=strrev($str);	//反转字符串
    // echo $_o;
        
    for($_0=0;$_0<strlen($_o);$_0++){   // for循环，这里就是加密算法过程，对每个字符进行加密
       
        $_c=substr($_o,$_0,1);	//substr函数：取字符串函数
        $__=ord($_c)+1; 		//进行ord编码并加1
        $_c=chr($__);			//转换成字符
        $_=$_.$_c;   			//变量连接
    } 
    return str_rot13(strrev(base64_encode($_)));//先进base64编码，再反转字符串，再进行rot13加密
}

highlight_file(__FILE__);
/*
   逆向加密算法，解密$miwen就是flag
*/
?>
```

​	2、读懂代码之后就简单了，直接写脚本，我们使用php语言，更贴合实际，也更易看懂。

```php
<?php
$miwen = "a1zLbgQsCESEIqRLwuQAyMwLyq2L5VwBxqGA3RQAyumZ0tmMvSGM2ZwB4tws";
$decode = base64_decode(strrev(str_rot13($miwen)));
for ($i=0;$i<strlen($decode);$i++)
{
    $c = substr($decode,$i,1);
    $ord = ord($c)-1;
    $chr = chr($ord);
    $_. = $chr;			//这里就等同于$_=$_.$chr
}
echo strrev($_);
?>
```

​	运行脚本后得出flag：

```
flag:{NSCTF_b73d5adfb819c64603d7237fa0d52977}
```

​	总结：

​		这里就是一个简单的逆向题，密文和加密过程都告诉我们了，只需要构造解密过程的脚本就好，想成为CTF大佬，必须学会一门语言，而且要精学。