概述
-----------

	ePL可编程语言允许任务设计人员将复杂的算法表达出来，并通过悬赏来获取结果。
	由于是基于C并吸收了很多C语言的基础运算符/函数，所以ePL语言的使用如C语言一样灵活。
	
	为了确保任务不会出现无限的死循环，所以ePL语言内部没有FOR,WHILE,和 DO循环。
	作为替换，任务设计人员可以通过ePL的REPEAT函数。
	
	任务设计人员需要定义特定的验证逻辑，这样在最大执行时间（WCET）内，
	可以确保Miner提交的赏金和POW结果可以被分布式网络内的所有节点所验证。
	
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
         
        ePL虚拟机所有存储都需要使用全局变量，且都是标准的32位和64位数据类型（int32_t, uint32_t, int64_t, uint64_t,
	float, double）。不支持本地存储。

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
        如果一个数值分配给一个数组变量，当它们时两个不同的数据类型时，该数值会被转换为
	与数组变量同样的数据类型。
	
        计算指数的变量可以接受无符号整数型/只要他们的指数；但是，需要执行变量的边界检查，这将会影响到性能。
	因此，请尽可能少的使用它们。例如：
	
		u[0] = u[u[3]]; 分析为: u[0] = ((u[3]<MAX_ALLOWED) ? u[u[3]] : 0)
		u[u[3]] = u[0]; 分析为: if (u[3]<MAX_ALLOWED) { u[u[3]] = u[0]; }

		
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
		
        这些变量允许任务设计人员随机化它们的算法输入以及触发函数的特定时间间隔。
	（i.e.当运行/迭代为0的时候运行init函数）
	

提交数据进行验证
-----------------------------
 
        许多小型的、简单的算法可以使用全随机的m[]数组中的变量进行全逻辑校验。但是更加复杂的算法必须提供简化的逻辑来执行验证。
        这种简化的逻辑通常需要在运行验证之前预填充一些变量数组。这些数组又会由miner发送给XEL节点来进行验证。
	
        例如，在TSP算法中，发现的路径数据的miner需要发数据给节点，并由节点根据设定的校验逻辑精准地校验是否符合悬赏/POW的要求。
	
        使用提交的数据是可选的，但是，当"verify"函数没有包含完整的算法时，它们通常还是需要用到的。
	
        当前，所有提交的数据必须是无符号整数型的u[]数组，任务设计人员需要提交上传数据的值和数组u[]需要提取的起始索引。
	
	
        向节点提交数据需要两个声明：
		
		submit_sz   XXXX  // Identifies # of Unsigned Ints to submit to the node
		submit_idx  XXXX  // Identifies starting index in u[] array to extract submitted data from
		
		Note: submit_sz is currently limited to a max size of (TBD - Need To Determine)
        
	在执行校验逻辑之前，提交到节点的数值将会更新到对应的u[]中。
	

迭代数据储存
----------------------

	ElasticPL has the ability to store a limited amount of data to be used to
	intialize subsequent iterations of the algorithm.  This allows job authors
	to create algorithms that build off the accepted Bounty Solutions from prior
	iterations in the same work package.
	
	The use of iterations and stored data is optional.
	
	Currently, the data to be stored is the same as the data submitted for
	verification of a Bounty solution (See the SUBMIT DATA FOR VALIDATION
	section for details on how to submit verification data).

	The data will only be stored for accepted Bounty solutions.

	If the ElasticPL job uses iterations:

		Stored data is available to the ElasticPL job by accessing the s[] array.
		
		For Iteration 0, the s[] array will be initializized to zeros.

		For all	other iterations, s[0] - s[storage_sz - 1] will be prefilled with
		the stored Unsigned	Int values for Boutny solutions for the prior iteration.


	If the ElasticPL job does not use iterations, the s[] array should not be used.
	

函数
---------

	All ElasticPL statements (except declaring global variable arrays and storage)
	must be contained within functions.
	
	No functions within ElasticPL are allowed to call the "main" function.

	The "main" function is the only function within ElasticPL that is allowed to
	call the "verify" function.

	Recursive function calls are not allowed.
	
	Functions can only be nested 256 levels deep.
	
	Function names can be made up of numbers (0-9), letters (a-z), and
	underscores (_).  However, names cannot begin with any reserved word
	in the ElasticPL language.
	
	Functions are declared as follows:
	
		function name_of_function {
			<statement1>;
			<statement2>;
			etc...
		}
		
		Note: Statements must be enclosed in {} brackets.
	
	To call a function, use the function name and parenthesis as follows:
	
		name_of_function();
	
	Functions can appear in any order within the program.
	
	All ElasticPL programs must have a "main" function.
	
	All ElasticPL programs must have a "verify" function.


"main"函数
---------------

	All programs must have a "main" function.

	This function should include the full algorithm that the Job Author wants
	solved.

	The "main" function can call any other function in the ElasticPL program
	including the "verify" function.
	
	The "main" function is called by the mining software to search for Bounty
	and POW solutions that meet the criteria established by the Job Author.
	
	Ultimately, the "main" function needs to determine if a Bounty and/or POW
	solution was found.  There are 2 ways to make this determination:
	
		Option 1:
		
			The "main" function can call the "verify" function which will contain
			specific logic to check if Bounty / POW solution are valid.
		
		Option 2:
		
			The "main" function can include all the logic to check for valid
			Bounty / POW solutions by using the verify_bty and verify_pow
			statements.
			
			See the "verify" Function details for additional details on how these
			two statement work.
		
		Note: The "main" function can only use Option 1 or Option 2, not both.		

		
"verify" 函数
-----------------

	All programs must have a "verify" function.

	This function should include the the minimal amount of logic required to
	ensure that the submitted solution meets the Bounty and/or POW requirements.
	
	The logic included in the "verify" function must be less than WCET = TBD

	VERIFY_BTY Statement
	----------------

		All ElasticPL programs must have a 'verify_bty' statement that includes
		an expression that can be evaluated to True or False.  The format of the 
		'verify_bty' statment is as follows:
		
			verify_bty( <Expression that evaluates to True or False> )
	
		This expression is used to indicate whether or not a given solution
		satisfies the Bounty requirements.  For Example:
	
			verify_bty (u[1000] == 0)
	
		The above statement indicates that a bounty will be rewarded for solutions
		where there value stored in u[1000] equals zero.  Otherwise, the
		solution does not qualify for a bounty.
	
		Only one 'verify_bty' statement per "verify" (and "main" if applicable)
		is allowed.
		
		The Job Author must ensure there is sufficient logic in the "verify"
		function to validate that the submitted data is in fact a valid solution.
		
		
	VERIFY_POW Statement
	----------------

		All ElasticPL programs must have a 'verify_pow' statement that checks
		if four Unsigned Ints determined by the Job Author produce an MD5 hash
		less than the current POW target.    The format of the 'verify_pow' 
		statment is as follows:
		
			verify_pow( <UINT1>, <UINT2>, <UINT3>, <UINT4> )
		
		The Job Author should choose four Unsigned Int values that will likely
		vary from miner to miner.  This help the author to ensure that miners
		are actually running the full logic to determine Bounty solutions.

		For Example:
	
			verify_pow (u[25], u[1001], u[823], u[123])
	
		The above statement indicates that a pow reward will be granted for
		solutions where the MD5 hash of u[25], u[1001], u[823], u[123] is less
		than the current target value.
	
		Only one 'verify_pow' statement per "verify" (and "main" if applicable)
		is allowed.
		
		The Job Author must ensure there is sufficient logic in the "verify"
		function to validate that the submitted data is in fact a valid solution.
	
	
ePL声明
--------------------
	
	Similar to C, all ElasticPL statements are terminated by a ';'
	
	IF / ELSE Statements
	--------------------
	
		ElasticPL supports IF / ELSE statements.  Their behavior is identical
		to the C programming language.

		
	REPEAT Statement
	----------------
	
		ElasticPL does not allow DO, WHILE, or FOR loops.  Instead, ElasticPL
		uses a 'repeat' statement which ensures all loops terminate and cannot
		run indefinitely.
		
		The format of the 'repeat' statment is as follows:
		
			repeat( <variable1>, <variable2>, <constant> ) { }

			repeat( u[100], u[200], 1000 ) {
				statement1;
				statement2;
				.
				.
				.			
			}
		
		variable1 = Unsigned Int variable to store the loop counter
		            (Array variable must have a constant index)
		variable2 = Unsigned Int variable or constant to store number of iterations to do
		            (Array variable can be a variable or constant index)
		constant  = Unsigned Int constant for Max number of iterations allowed
		            (constant is used to calculate the WCET of the Repeat)
		
		Note:  'repeat' statements can only be nested up to 32 Levels.
			
	
ePL运算符
-------------------

	The following operators are supported by ElasticPL.  These operators behave
	the same as C operators based on the C99 standard:
	
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

	ElasticPL includes several functions from the C math library.  These built-in
	functions behave the same as in C.
	
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
	
	ElasticPL has one custom built-in function below.  More functions will be
	added later.
	
		gcd ( uint x, uint y)        Computes greatest common denominator

	
		
