概述
-----------

	 ePL(ElasticPL) 编程语言允许任务分发人员将复杂的算法表达为可悬赏解决的方式。
	 它参照并吸收了很多C语言的基础运算符/函数。
 
	 为了确保任务不会出现无限循环，ePL语言没有 FOR, WHILE和 DO循环。
	 作为替代，任务分发人员可以使用ePL的REPEAT函数。
 
	 任务分发人员需要定义特定的验证逻辑，使其低于最大执行时间（WCET）内，
 	 以确保Miner提交的悬赏和POW的解决方案可以被分布式网络内的所有节点所验证。
	
程序设计规范
--------------

	一个ePL程序应包含:
		 - 一个或者多个全局数组变量
		 - 存储声明（可选） 
	         - "main"函数
	         - "verify"函数
	         -用户自定义函数（可选）

		
全局变量
----------------
         
        ePL虚拟机所有存储都需要使用全局变量，且都是标准的32位和64位数据类型（int32_t, uint32_t, 
		int64_t, uint64_t,float, double）。不支持本地存储。

        不允许使用自定义变量，数据的存储需要使用一个符合要求数据类型的数组内。数据的声明如下：
	
		array_int    XXXX
		array_uint   XXXX
		array_long   XXXX
		array_ulong  XXXX
		array_float  XXXX
		array_double XXXX
	
		Note 1: XXXX 表示数组中的元素数.
		Note 2: 数组内元素使用统一的数据类型.
		Note 3: 所有全局变量的最大组合存储大小不超过（待定 - 需要确定最大的内存空间）
        
        全局变量的声明都应该在程序的第一行，所有函数之前。
	
        数组的前缀可以使用i, u, l, ul, f, 或者 d，方括号内为基于0的数组索引。例如：
	
		i[0] = 100;
		u[5] = u[3] * 10;
		d[6] = u[5] / i[5];

        虽然不鼓励使用混合的数据类型，但有时候还是会不可避免地会使用到。
        由于ePL没有强制转换的操作，当遇到数组变量与常规变量或常量组合，且都是不同的数据类型时，
        需遵循以下转换优先级：

		int32_t -> uint32_t -> int64_t -> uint64_t -> float -> double
		
	        例如：   d[6] = u[5] / i[5];
		
			u[5] / i[5] 分析为 u[5] / (uint)(i[5]) 
			
			但没有必要写成(u[5] / (uint)(i[5])) 因为 d[6] 的存储类型
			已经决定了操作的结果。	
			
        如果一个数值分配给一个数组变量，当它们时两个不同的数据类型时，该数值会被转换为与数组变量同样的数据类型。
	
	 	拥有动态计算索引值的变量可接受无符号整数型（ Unsigned Int）/长整数型（Long）作为索引值；
		但是，需要执行边界检查，这可能会影响到性能。因此，请尽可能少的使用它们。例如：        
	
		u[0] = u[u[3]]; 等价于: u[0] = ((u[3]<MAX_ALLOWED) ? u[u[3]] : 0)
		u[u[3]] = u[0]; 等价于: if (u[3]<MAX_ALLOWED) { u[u[3]] = u[0]; }

		
VM初始化变量
------------------------
        
        ePL虚拟机在每次的运行/迭代都会更新12个无符号整数型变量。
        这些变量存储在m[0]到m[11]中，定义如下：
	
		m[0]	Random 32 bit Unsigned Int
		.
		.
		.
		m[9]	Random 32 bit Unsigned Int
		m[10]	Run Number
		m[11]	Iteration Number
		
        这些变量允许任务分发人员随机化它们的算法输入以及触发特定时间间隔的函数。（例如当运行/迭代为0的时候运行init函数）
	

提交数据进行验证
-----------------------------
 
        许多小型的、简单的算法可以使用全随机的m[]数组中的变量进行全逻辑校验。但是更加复杂的算法必须提供简化的逻辑来执行验证。
        这种简化的逻辑通常需要在运行验证之前预填充一些变量数组。这些数组又会由miner发送给XEL节点来进行验证。
	
        例如，在TSP算法中，发现的路径数据的miner需要发数据给节点，并由节点根据设定的校验逻辑精准地校验是否符合悬赏/POW的要求。
	
        使用提交的数据是可选的，但是，当"verify"函数没有包含完整的算法时，它们通常还是需要用到的。
	
        当前，所有提交的数据必须是无符号整数型的u[]数组，任务分发人员需要提交上传数据的值和数组u[]需要提取的起始索引。
	
	
        向节点提交数据需要两个声明：
		
		submit_sz   XXXX  // Identifies # of Unsigned Ints to submit to the node
		submit_idx  XXXX  // Identifies starting index in u[] array to extract submitted data from
		
		Note: submit_sz is currently limited to a max size of (TBD - Need To Determine)
        
		在执行校验逻辑之前，提交到节点的数值将会更新到对应的u[]中。
	

迭代数据存储
----------------------
  	
        ePL能够存储有限的数据，用于初始化算法的后续迭代。这允许任务分发人员从同一个工作包迭代前，去创建算法并建立公认的任务悬赏。

        迭代和数据存储的使用是可选的。
	 
        当前，要存储的数据与提交用于验证奖励解决方案的数据相同（可查看 提交数据进行校验 章节的有关如何提交验证数据部分）

        只存储已接受悬赏方案的数据。

        如果ePL任务使用迭代:

                通过访问s[]数组，存储的数据可用于ePL任务。

                在迭代数为0时，数组s[]会被初始化为0.

                在之后其他迭代，每一次迭代前，悬赏解决方案的数据会被预填充到数据类型为无符号整数型的 s[0] - s[storage_sz - 1]中。

        如果ePL任务没有使用迭代，数组s[]将不会被用到。	

函数
---------
        
        所有ePL语句（声明全局变量数组和存储除外）必须包含在函数中。

        在ePL中不允许调用"main"函数

        在ePL中只允许"main"函数调用"verify"函数

        不允许递归函数的调用

        函数最大嵌套允许256层

        函数的命名可以使用数字（0-9）,字母（a-z），或者下划线（—）.但是，命名禁止使用ePL的预留字。	

        
        函数声明如下：
	
		function name_of_function {
			<statement1>;
			<statement2>;
			etc...
		}
                
	        注：语句必须使用{}括起来。
	
        要调用函数，请使用函数名和括号，如下所示：
	
		name_of_function();
	
        函数在程序中的顺序可以根据需要设定。
	  
        所有的ePL程序必须有一个"main"函数
	
        所有的ePL程序必须有一个"verify"函数


"main"函数
---------------
      
        所有的程序必须有一个"main"函数
	
        任务分发人员想要解决的算法编程，需要全部在主函数中
	
        在ePL程序中，"main"函数可以调用任何其他函数，包括"verify"函数
	
        mining软件通过调用"main"函数来查找任务设计人员建立标准的悬赏和POW的解决方案
	
        最后，"main"函数需要确定是否找到了悬赏 和/或 POW的解决方案。这里有两个方式可以确定：
	
	        选项1:
		
		        在"main"函数中调用"verify"函数，其中"verify"应包含
			有效的、明确的悬赏/POW的解决方案的校验逻辑。
		
	        选项2:
		
		        在"main"函数中包含有效的悬赏/POW的解决方案的校验逻辑，需要使用veriry_bty和verify_pow的声明。
			
	                可以在"verify"函数篇章了解以上两个声明是如何工作的更多细节
		
		
	        注："main"函数只能使用用选项1或者选项2，不能两个同时使用。	
		
"verify" 函数
-----------------
   
        所有的程序必须有一个"verify"函数。
	
        校验函数应该包含小体积精简的校验逻辑要求，来确保提交来的解决方案符合悬赏 与/或 POW的要求。
	
        这个校验函数中的逻辑设计，运行时间不能超过WCET（待定）

	VERIFY_BTY 表达式
	----------------
        
	        所有的ePL程序必须包含'verify_bty'的声明，可以为True或者False.'verify_bty'的表达式格式如下：
		
			verify_bty( <Expression that evaluates to True or False> )
	    
	        这个表达式的作用是表明是否有满足悬赏要求的解决方案。例如：
	
			verify_bty (u[1000] == 0)
	
	        上面的声明表示当存储在u[1000]的数值等于0时，这个解决方案就可以拿到悬赏的奖励。反之，该解决方案不符合悬赏要求。
		
	        每个"verify"（或者"main"，如果使用到）函数只能包含一个'verify_bty'表达式。
	
                任务分发人员必须要确保"verify"函数中有足够且正确的校验逻辑，来验证提交来的解决方案是有效的。
		
	VERIFY_POW 表达式
	----------------

                所有的ePL程序必须有一个'verify_pow'表达式，用于检查任务分发人员产生的4个无符号整数型的MD5哈希值是否小于当前的POW目标值。
	        'verify_pow'表达式的格式如下：
		
			verify_pow( <UINT1>, <UINT2>, <UINT3>, <UINT4> )
		
	        任务分发人员需要选择4个无符号整数型数值，因Miner而异。
		这有助于任务分发人员确保miners可以运行完整的校验逻辑来准确获取悬赏的解决方案。

		例如:
	
			verify_pow (u[25], u[1001], u[823], u[123])
	
	        上面的表达式表示如果 u[25], u[1001], u[823], u[123]中的MD5哈希值小于当前的目标数值，
		那么解决方案将获得POW奖励。

	        每个'verify'（或者'main'，如果使用到）函数只能包含一个'verify_pow'表达式。

		任务分发人员必须要确保"verify"函数中有足够且正确的校验逻辑，来验证提交来的解决方案是有效的。
	
ePL概述
--------------------
      
        如同C语言，所有的ePL声明都需要在结尾加一个';'
	
	IF / ELSE 语句
	--------------------
	
	        ePL支持IF / ELSE语句，功能和用法与C语言一样。
		
	REPEAT 语句
	----------------
	        
	        ePL不支持DO，WHILE,还有FOR循环语句。
		作为替代，ePL使用的是'repeat'语句，来确保所有的循环能运行到结束，但又不能无限期的循环下去。
	
	        'repeat'语句的格式如下：
		
			repeat( <variable1>, <variable2>, <constant> ) { }

			repeat( u[100], u[200], 1000 ) {
				statement1;
				statement2;
				.
				.
				.			
			}
		
		variable1 = 无符号整数型变量用于存储循环次数
		            (数组变量必须具有常量索引)
		variable2 = 无符号整数型变量用于存储需要执行的迭代次数
		            (数组变量必须具有常量索引)
		constant  = 无符号整数型变量用于存储允许迭代次数的最大值
		            (常量用于计算'repeat'的WCET次数)
		
	        注：'repeat'语句最多只能嵌套32层
			
	
ePL运算符
-------------------
 
        以下操作符ePL都支持。它们的功能和用法同基于C99标准的C的功能和用法一样。
	
	Precedence  Operator  Description            Order
	------------------------------------------------------------
	    0         a++     Postfix Increment      (Left to Right)
	    0         a--     Postfix Decrement      (Left to Right)
	    1         ++a     Prefix Increment       (Right to Left)
	    1         --a     Prefix Decrement       (Right to Left)
	    1         -a      Unary Minus            (Right to Left)
	    1         !       Logical NOT            (Right to Left)
	    1         ~       Bitwise NOT            (Right to Left)
	    2         *       Multiplication         (Left to Right)
	    2         /       Division               (Left to Right)
	    2         %       Remainder              (Left to Right)
	    3         +       Addition               (Left to Right)
	    3         -       Subtraction            (Left to Right)
	    4         <<      Bitwise Left Shift     (Left to Right)
	    4         <<<     Bitwise Left Rotation  (Left to Right)
	    4         >>      Bitwise Right Shift    (Left to Right)
	    4         >>>     Bitwise Right Rotation (Left to Right)
	    5         <       Less Than              (Left to Right)
	    5         <=      Less Than Or Equal     (Left to Right)
	    5         >       Greater Than           (Left to Right)
	    5         >=      Greater Than Or Equal  (Left to Right)
	    6         >=      Equal                  (Left to Right)
	    6         >=      Not Equal              (Left to Right)
	    7         &       Bitwise AND            (Left to Right)
	    8         ^       Bitwise XOR            (Left to Right)
	    9         |       Bitwise OR             (Left to Right)
	    10        &&      Logical AND            (Left to Right)
	    11        ||      Logical OR             (Left to Right)
	    12        a?b:c   Ternary Conditional    (Right to Left)
	    13        =       Assignment             (Right to Left)
	    13        +=      Add and Assignment     (Right to Left)
	    13        -=      Sub and Assignment     (Right to Left)
	    13        *=      Mul and Assignment     (Right to Left)
	    13        /=      Div and Assignment     (Right to Left)
	    13        %=      Mod and Assignment     (Right to Left)
	    13        <<=     L Shift and Assignment (Right to Left)
	    13        >>=     R Shift and Assignment (Right to Left)
	    13        &=      Bit AND and Assignment (Right to Left)
	    13        ^=      Bit XOR and Assignment (Right to Left)
	    13        |=      Bit OR and Assignment  (Right to Left)

		
ePL内置函数
----------------------------
        
        ePL语言内置了几个C语言math库的函数。这些函数的功能和用于与C语言中一样。
	
		sinh( double d )             Computes hyperbolic sine
		sin( double d )              Computes sine
		cosh( double d )             Computes hyperbolic cosine
		cos( double d )              Computes cosine
		tanh( double d )             Computes hyperbolic tangent
		tan( double d )              Computes tangent
		asin( double d )             Computes arc sine
		acos( double d )             Computes arc cosine
		atan2( double d )            Computes arc tangent, using signs to determine quadrants
		atan( double d )             Computes arc tangent
		exp( double d )              Computes e raised to the given power
		log10( double d )            Computes common (base-10) logarithm
		log( double d )              Computes natural (base-e) logarithm
		pow( double x, double y )    Computes a number raised to the given power 
		sqrt( double d )             Computes square root
		ceil( double d )             Computes smallest integer not less than the given value
		floor( double d )            Computes largest integer not greater than the given value
		fabs( double d )             Computes absolute value of a floating-point value
		abs( int i )                 Computes absolute value of an integral value
		fmod ( double x, double y )  Computes remainder of the floating-point division operation
	
        ePL语言拥有一个自定义的内置函数。更多的函数后面会再添加。
	
		gcd ( uint x, uint y)        Computes greatest common denominator

	
		
