# 12 行代码演示 Lua 调试原理

## 零、Lua 没有自带调试器

Lua 没有自带调试器，但是提供了一套调试库，即 debug 库。搞清楚这套调试库有助于我们使用现在市面上的 Lua 调试器，有助于理解脚本的调试原理，部分接口在平常使用 Lua 进行编码时，也能很方便的解决一些问题。

<!--more-->

## 一、断点，并获得当前的变量值

* 看代码如下

	break.lua
 	  
		 1 function simple_break(name,param)
		 2   print("运行到行:"..param)
		 3   if param == 11 then
		 4     print("断点并获得变量值:",debug.getlocal(2,1))
		 5     local time = os.clock()
		 6     while os.clock() - time <= 3 do end -- 死循环创造断点
		 7   end
		 8 end
		 9 debug.sethook(simple_break,"l")
		10 local letter = "Apple"
		12 letter = "Cat"
	 
	使用命令行执行运行结果为(括号内容为注释说明):

		 Mac-mini:~ wyang$ lua break.lua
		 运行到行:10
		 运行到行:11
		 断点并获得变量值:	letter	Apple
		 (这里等待 3 秒)
		 运行到行:12
		 Mac-mini:~ wyang$
	 
	注意，代码需要在命令行里面运行，在部分 IDE 如 Sublime 中有方便运行 Lua 脚本的功能，但是直接在 Sublime 中运行会导致：一开始程序并没有输出，3秒以后，所有以上内容全部出现。
	
## 二、环境配置


1. Mac 上的环境配置

	没有安装 HomeBrew 的同学请到[官网][1]安装,很简单就一条命令。
	
	然后执行以下语句即可安装 Lua 即可:


		brew install lua
		
2. Windows 上的环境配置

	到[这里][2]下载 Lua 的二进制文件，解压后将可执行程序的目录添加到环境变量 path 里，然后重启命令行程序即可。
	
 
## 三、分解来看

1. 知道运行到哪一行，并进行处理

   hook.lua
	
		debug.sethook(print,"l")
		local letter = "Apple"
		letter = "Boy"
	
	运行结果为:
	
		peterdeMPB:lua peter$ lua hook.lua
		line	2
		line	3
   代码第一行中，掩码 "l" 的意义为 "line",这一行的意义为:当执行到新的行时,调用函数 "print"。

		
2. 死循环阻塞中断 Lua 运行进程

	break.lua

        local time = os.clock()
        while os.clock() - time <= 3 do end -- 死循环创造断点
	
	正式的调试器，在这里可以连接调试服务器，接收操作来控制调试进程。
	
3. 获得当前断点变量的值

	break.lua
	
		 print("断点并获得变量值:",debug.getlocal(2,1))
		
	即：获得第 2 层函数的第 1 个局部变量的值，并打印出来，可以尝试修改获得第一层函数的值试试看。

## 四、支持的所有调试Api

* 以上调试基于 Lua 的 debug 库，下面列出所有支持的Api，参考[这里][3]

	1. debug.sethook ([thread,] hook, mask [, count])
	
		注册一个函数,用来在程序运行中某一事件到达 时被调用。有四种可以触发一个 hook 的事件
		* call : Lua 调用一个函数时触发,掩码为 c
		* return : 函数返回时触发,掩码为 r
		* line : Lua 运行到新行时触发,掩码为 l

		例如：要在调用函数时，函数返回时，运行到新行时都触发事件，应该这样写:

			debug.sethook(hookfunc,"crl")
		
	2. debug.debug
		
		进入交互式的调试界面。
		
	3. getfenv(object)
		
		返回对象的环境变量。
		
	4. gethook(optional thread)	
	
		返回三个表示线程钩子设置的值： 当前钩子函数，当前钩子掩码，当前钩子计数。
		
	5. getinfo ([thread,] f [, what])
		
		返回关于一个函数信息的表。
		
	5. debug.getlocal ([thread,] f, local)
	
		此函数返回在栈的 f 层处函数的索引为 local 的局部变量 的名字和值。 这个函数不仅用于访问显式定义的局部变量，也包括形参、临时变量等。
		
	6. getmetatable(value)
		
		返回给定对象的 metatable，如果给定对象没有 mdtatable 则返回 nil。
		
	7. getregistry()
	
		返回注册表表，这是一个预定义出来的表， 可以用来保存任何 C 代码想保存的 Lua 值。
		
	8. getupvalue (f, up)
	
		此函数返回函数 f 的第 up 个上值的名字和值。 如果该函数没有那个上值，返回 nil。
以 '(' （开括号）打头的变量名表示没有名字的变量 （去除了调试信息的代码块）。

	9. setlocal ([thread,] level, local, value)
	
		这个函数将 value 赋给 栈上第 level 层函数的第 local 个局部变量。 如果没有那个变量，函数返回 nil。如果 level 越界，抛出一个错误。
		
	10. setmetatable (value, table)
	
		将 value 的元表设为 table （可以是 nil）。 返回 value。
		
	11. setupvalue (f, up, value)
	
		这个函数将 value 设为函数 f 的第 up 个上值。 如果函数没有那个上值，返回 nil 否则，返回该上值的名字。
		
	12. traceback ([thread,] [message [, level]])
	
		如果 message 有，且不是字符串或 nil， 函数不做任何处理直接返回 message。 否则，它返回调用栈的栈回溯信息。 字符串可选项 message 被添加在栈回溯信息的开头。 数字可选项 level 指明从栈的哪一层开始回溯 （默认为 1 ，即调用 traceback 的那里）。
		
## 五、后续

1. 后面会以一个小例子讲 sublime text 3 插件的开发
2. 最后会讲到基于 sublime text3 的 Lua 调试器的开发

[1]: http://brew.sh/
[2]: https://sourceforge.net/projects/luabinaries/files/5.2.4/
[3]: http://www.runoob.com/lua/lua-debug.html
