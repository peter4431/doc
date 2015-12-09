# 15行代码演示 Lua 调试原理
## 一、断点，并获得当前的变量值

* 看代码如下

	break.lua
 	  
		  1 local socket = require("socket")
		  2 function simple_break(name,param)
		  3   print("运行到行:"..param)
		  4   if param == 14 then
		  5     print("断点并获得变量值:",debug.getlocal(2,2))
		  6     while true do
		  7       socket.sleep(3)
		  8       break
		  9     end
		 10   end
		 11 end
		 12 debug.sethook(simple_break,"l")
		 13 local letter = "Apple"
		 14 letter = "Boy" -- will break here
		 15 letter = "Cat"
	 
	运行结果为:

		peterdeMPB:lua peter$ lua break.lua
		运行到行:13
		运行到行:14
		断点并获得变量值:	letter	Apple
		(3秒后输出下一行)
		运行到行:15
	 
	 
	 
## 二、分解来看

1. 知道运行到哪一行，并进行处理

   hook.lua
	
		1 debug.sethook(print,"l")
		2 local letter = "Apple"
		3 letter = "Boy"
	
	运行结果为:
	
		peterdeMPB:lua peter$ lua hook.lua
		line	2
		line	3
		
2. 死循环加 socket 阻塞中断 Lua 运行进程

	break.lua

		  6  while true do
		  7     socket.sleep(3)
		  8     break
		  9  end
	
	* 此处依赖于 LuaSocket，
	* 如果是本地 Lua,可以使用 Lua 的包管理器 [LuaRocks][1] 来进行安装
	* 如果是 quick-cocos2d-x,可直接使用
	* 正式的调试器，在这里可以连接调试服务器，接收操作来控制调试进程
	
3. 获得当前断点变量的值

	break.lua
	
		  5  print("断点并获得变量值:",debug.getlocal(2,2))

## 三、支持的所有调试Api

* 以上调试基于 Lua 的 debug 库，下面列出所有支持的Api，参考[这里][2]

	1. debug.sethook ([thread,] hook, mask [, count])
	
		注册一个函数,用来在程序运行中某一事件到达 时被调用。有四种可以触发一个 hook 的事件
		* call : Lua 调用一个函数时
		* return : 函数返回时
		* line : Lua 运行到新行时
		* count : 运行指定数据的指令时
		
	2. debug.debug
		
		进入交互式的调试界面
		
	3. getfenv(object)
		
		返回对象的环境变量。
		
	4. gethook(optional thread)	
	
		返回三个表示线程钩子设置的值： 当前钩子函数，当前钩子掩码，当前钩子计数
		
	5. getinfo ([thread,] f [, what])
		
		返回关于一个函数信息的表
		
	5. debug.getlocal ([thread,] f, local)
	
		此函数返回在栈的 f 层处函数的索引为 local 的局部变量 的名字和值。 这个函数不仅用于访问显式定义的局部变量，也包括形参、临时变量等。
		
	6. getmetatable(value)
		
		把给定索引指向的值的元表压入堆栈。如果索引无效，或是这个值没有元表，函数将返回 0 并且不会向栈上压任何东西。
		
	7. getregistry()
	
		返回注册表表，这是一个预定义出来的表， 可以用来保存任何 C 代码想保存的 Lua 值
		
	8. getupvalue (f, up)
	
		此函数返回函数 f 的第 up 个上值的名字和值。 如果该函数没有那个上值，返回 nil。
以 '(' （开括号）打头的变量名表示没有名字的变量 （去除了调试信息的代码块）。

	9. setlocal ([thread,] level, local, value)
	
		这个函数将 value 赋给 栈上第 level 层函数的第 local 个局部变量。 如果没有那个变量，函数返回 nil。如果 level 越界，抛出一个错误。
		
	10. setmetatable (value, table)
	
		将 value 的元表设为 table （可以是 nil）。 返回 value
		
	11. setupvalue (f, up, value)
	
		这个函数将 value 设为函数 f 的第 up 个上值。 如果函数没有那个上值，返回 nil 否则，返回该上值的名字
		
	12. traceback ([thread,] [message [, level]])
	
		如果 message 有，且不是字符串或 nil， 函数不做任何处理直接返回 message。 否则，它返回调用栈的栈回溯信息。 字符串可选项 message 被添加在栈回溯信息的开头。 数字可选项 level 指明从栈的哪一层开始回溯 （默认为 1 ，即调用 traceback 的那里）
		
## 四、后续

1. 后面会以一个小栗子讲到 sublime text 3 插件的开发
2. 最后会讲到基于 sublime text3 的 Lua 调试器的开发
3. 查看调试器的源码 [LuaSoar][3]
4. 以一个小栗子讲 vim 插件的开发

[1]: http://www.luarocks.org/
[2]: http://www.runoob.com/lua/lua-debug.html
[3]: https://github.com/peter4431/LuaSoar
