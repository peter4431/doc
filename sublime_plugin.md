# 自定义Sublime Text 3 插件

> 快捷键调用插件打开调试界面，并调用本地脚本

## 零、一个最简插件模板
* 这个模板仅仅实现了打印输出信息，在当前sublime打开的文件中插入 "Hello world"
* 模板使用
	1. 从 github 上检出模板，或者下载
	
			git clone git@github.com:peter4431/template_plugin.git
		
	2. 依次点击 Sublime Text > Preferences > Browser Packages
	3. 将检出的 template_plugin 文件夹放入上一步打开的文件夹中
	4. 使用 Sublime Text 新建一个文件，按 shift + F2 即可看到效果
	5. ctrl + ` 可以打开插件的输出界面，可以看到最后一条输出 "Hello world"，用这种方式来调试插件
	
 

## 一、模板文件解析

1. 主文件 plugin.py

        import sublime, sublime_plugin
        class TemplatePluginCommand(sublime_plugin.TextCommand):
            def run(self, edit):
                print("Hello world")
                self.view.insert(edit, 0, "Hello, World!")
2. 快捷键配置文件
	
	* Default (Linux).sublime-keymap
	* Default (Linux).sublime-keymap
	* Default (OSX).sublime-keymap
	
			[
    			{"keys": ["shift+f2"], "command": "template_plugin" }
			]
			
		是一个 json 数组，定义快捷键和命令的对应关系
		
## 二、调用命令，打开调试视图

1. 主文件代码和注释

		import sublime, sublime_plugin

        # 定义全局的process，用于保持唯一一个
        process = None
        class TemplatePluginCommand(sublime_plugin.TextCommand):

            def set_debug_view(self):
                self.debug_layout = {
                    "cols": [0.0, 0.5, 1.0],
                    "rows": [0.0, 0.7, 1.0],
                    "cells": [[0, 0, 2, 1], [0, 1, 1, 2], [1, 1, 2, 2]]
                }
                
                self.add_view("Lua Context", 1, 0)
                self.add_view("Lua Expression", 2, 0)

            def add_view(self,name,pos,index):
                # 设定 sublime 分栏布局
                window = sublime.active_window()
                window.set_layout(self.debug_layout)

                # 打开调试文件，用来显示 Context，即当前上下文环境的值
                view = window.new_file()
                view.set_scratch(True)
                view.set_read_only(True)
                view.set_name(name)
                view.settings().set('word_wrap', False)

                # 设置文件在第二个分栏，第一个文件
                window.set_view_index(view, pos, index)

                # 设置文件的语法
                view.run_command("set_file_type",{'syntax': 'Packages/Lua/Lua.tmLanguage'})

            def start_quick(self):
                
                # 启动
                args = ["python -m SimpleHTTPServer"]

                if process:
                    try:
                        process.terminate()
                    except Exception:
                        pass

                process = subprocess.Popen(args)

            def run(self, edit):
                self.start_quick()
                self.set_debug_view()
                
2. 调用外部命令是纯 Python API，需要提的一点是，若使用 ```os.system("cmd")```会非常卡，使用 ```subprocess```会好很多
3. 设定布局的参数说明

		"cols": [0.0, 0.5, 1.0],# 表示，等分两列
		"rows": [0.0, 0.7, 1.0],# 表示，分两行，比例分别是7和3
		# 分成三个窗口，根据上面两个参数分的网格，下面一个cell的4各值，表示一个矩形的左上和右下
		"cells": [[0, 0, 2, 1], [0, 1, 1, 2], [1, 1, 2, 2]]

## 三、Sublime Text 插件基本概念
1. 命令的约定

	在主 py 文件中定义类 TemplatePluginCommand,则调用命令 "template_plugin"会执行此类的 ```run```方法
		
2. 插件生命周期

	* 插件被调用时会执行插件里的方法 ```plugin_loaded()```(若有定义)，插件被卸载时，会调用 ```plugin_unloaded()```(若有定义)
	* 可以通过输入命令，右键，快捷键等方式调用命令
	* 可以通过监听文件创建，文件修改，等事件来作出反应
	
2. 各个关键模块和 Sublime Text 中的对应关系
	
	| 类名/模块名 | Sublime Text编辑器中概念 |
	| ----------- | ------------ |
	| sublime | 全局模块，绝大多数 API 都来自 sublime |
	| sublime.Window | sublime 是可以打开多个窗口的，此模块表示窗口，管理界面，文件，命令行窗口 | 
	| sublime.View | 文本的一个视图，即文件的一个打开窗口，一个 tab 页 |
	| sublime.Region | 一个选区，有开始值和结束值 |	
	| sublime.Selection | Sublime Text 可以选择多个不连续的地方，由此类表示，持有 Region 的引用 |
	| sublime_plugin.EventListener | 在此类中定义 View 事件的回调，如新建文件，文件被修改等 |
	

## 四、完整api
1. 官方 API 在[这里][1]
2. 中文的 API 参考[这里][2],翻译的版本并不完全匹配，请作为参考，但是以官方文档为准

[1]: http://www.sublimetext.com/docs/3/api_reference.html
[2]: http://www.oschina.net/translate/sublime-text-plugin-api-reference
