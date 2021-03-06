[TOC "float:right"]
<center>lua代码规范</center>
===

## 命名惯例
1. 所有lua文件命名时使用小写字母。
2. 类名、变量名等使用驼峰法，eg：playerName。
3. 文件内局部变量加s前缀。
4. 全局函数用g前缀。
5. 枚举值用e前缀。
6. 常量：大写加下划线，eg：KIND_PET_FOOD
7. 模块：小写加下划线：eg：item_factory_lua

## 文件组织
1. 文件开头加上此文件的功能、职责的简要描述；
	例如：
    -- 功能描述
    每个文件都加module 限定词； 导入的模块都加 local 限定词；或者使用(module(..., package.seeall))，这样便于进行热更新
    
2. 所有提供外部函数都加如下格式的注释。
	例如：
    ```
    -- 此函数检测是否可以从A(oldx, oldy)点走到B点（newx, newy）
    -- @param oldx 当前所在点x
    -- @param oldy 当前所在点y
    -- @param newx 目标点x
    -- @param newy 目标点y
    -- @return 若可以到达，返回true；否则返回false
    function Object:checkBar(oldx, oldy, newx, newy)
    ...
	end
    ```
3. 函数与函数间、以及一些定义之间加上空行。

4. 函数内的临时变量、文件内的局部函数都加上 local 限定词。

5. 函数的行数过长（大于100行）时，尽量拆分为多个子函数；函数中一些晦涩的部分，一定要加上注释。

6. 短小的注释使用 --； 较长的注释使用 --[[ ]]。

8. （暂时不用考虑）Lua类设计时，用元表来实现oop。不要直接增加函数成员，因为直接增加函数成员会导致内存增加并且在jit下执行效率和用元表方式无差异。

9. 文件使用UTF8格式

## 分隔和缩进
* 使用空行
  在下述情况下使用单行的空白行来分隔：
  1. 在方法之间
  2. 在方法内部代码的逻辑段落小节之间
  3. 在注释行之前，注释之前增加一行或者多行空行。
  
* 使用空格符
  除正常的成分之间以空格符分隔名（如数据类型和变量名之间），在下述情况下也应使用一个空格符来分隔：
  1. 运算符和运算符之间，如： c = a + b；
  2. 在参数列表中的逗号后面，如：
  	```
    function m1(year, month)
  	end
    ```
  3. 在for语句时，如：
  	```
    for k, v in pairs(t) do 
    end
    ```
  4. 在下列情况下不要使用空格。
  	例如： 
    函数定义时： 
    ```
    function test1(a) 
    end
    ```
    不要这样： 
    ```
    function test1( a ) 
    end
    ```
    函数调用时： 
    ```
    test1(3) 
    ```
    不要这样： 
    ```
    test1( 3 ) 
    ```
    不要如此的原因在于：
    	a).容易忘记相关空格，导致风格不统一，这样还不如不加； 
        b).lua解析语法时是采用空格等分割来解析的，某些情况下，若不小心加空格会导致非预期的结果。 

## 代码建议
* 尽可能使用local修饰变量（重要的事情要说三遍！）
	原因：
    使用local的变量会在作用域结束时释放其内存
    使用local的变量会比全局变量的存取更快
    全局变量会污染全局的命名空间，可能会导致诡异的bug出现

* assert函数开销不小，请慎用。

* 尽量减少表中的成员是另一个表的引用。 考虑lua的垃圾收集机制、内存泄露等。 

* 高级特性尽可能不用 

* 写代码时尽可能写的简单，考虑性能时先做好推断，看看能提升多少，增加的复杂度以及造成的代码晦涩有多严重，然后再决定如何做 

* 加载的xml数据表，尽可能的做好数据校验，若校验失败，要出发断言，使服务器无法启动；不要等出错时，回过头来检查是数据表问题还是逻辑问题。 

* 出错时，记录好错误日志。 

* 有的函数开销比较大，而调用的频率很低，那么可以不对他做优化；
  反之，有的函数开销较小，但是调用的频率很高，从如何降低调用频率以及减少函数开销两个角度去思考，然后定下优化方案     
    
* 直接判断真假值
    ```
    -- 不推荐
    if  obj  ~=  nil  and  willBreak  ==  false  then
        -- ...
    end
    
    -- 推荐
    if  obj and  not  willBreak then
        -- ...
    end
    ```
    原因：Lua在逻辑判断时将所有非false和nil的逻辑判断视为真，反之视为假，不需要再与布尔值和nil进行比对。    但是，在需要对false和nil进行区分时，需要写明==：obj == nil和obj == false。
    
* 默认参数的实现
	范式：param = param or defaultValue
	```
    function  setName(name)

        name  =  name or  'noName'

        -- ...

    end
    ```
    原因：or会在第一次为true的时候断路，返回其判断的最后一个值。所以当name为空时，name or 'noName'返回为'noName'，这会将name的值自动设置为noName。
    
* 一行代码实现表的拷贝
    ```
    u  =  {unpack(t)}
    ```
    
* 一行代码判断表是否为空
  用#t == 0并不能判断表是否为空，因为#预算符会忽略所有不连续的数字下标和非数字下标。
  正确做法是：
  ```
    if  next(t)  ==  nil  then

        -- 表为空

        -- ...

    end
  ```
  因为表的键可能为false，所以必须与nil比较，而不直接使用~next(t)来判断表是否空。

* 更快的插入代码
    ```
    -- 更慢，不推荐
    table.insert(t,  value)
    -- 更快，推荐
    t[#t+1]  =  value
    ```
原因：[]和#避免了高层的函数调用开销。

























