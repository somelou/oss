# VSCode自定义用户代码片段

# 目标

输入变量名可以生成双向绑定该变量所需的代码。这个双向绑定是Angular的实现方式。见例如下：

**输入变量名visible:**

```typescript
visible
```

**期望得到代码：**

```typescript
private _visible: boolean;
@Input()
get visible() {
	return this._visible;
}
set visible($value: boolean) {
	this._visible = $value;
	this.visibleChange.emit($value);
}
@Output()
visibleChange = new EventEmitter<boolean>();
```

# 提出设想

获取输入的内容（即变量名），然后和固定代码进行拼接。注意缩进和换行！

# snippet示例

在  `File > Preferences (Code > Preferences on macOS)` 中选择 `User Snippets` 在弹出框里选择对应的代码片段语言，我这里使用的是`TypeScript`

打开的`typescript.json` 中代码如下：

```json
{
	// Place your snippets for typescript here. Each snippet is defined under a snippet name and has a prefix, body and 
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	// same ids are connected.
	// Example:
	"Print to console": {
		"prefix": "log",
		"body": [
			"console.log('$1');",
			"$2"
		],
		"description": "Log output to console"
	}
}
```

- `Print to console` 代码片段名称
- `prefix` 触发前缀
- `body` 插件内容可以是字符串，也可以为数组，若为数组每个元素都做为单独的一行插入
- `description` 插件描述

# Snippet语法

## **制表位(Tabstops)**

使用制表位(Tabstops)可是在代码片段中移动光标位置，使用`$1`,`$2`来指定光标的位置,数字代表光标的移动的顺序，值得注意的时`$0`代表光标的最后位置。如果有多个相同的制表位(Tabstops)会在编译器里同时出现多个光标（类似编译器的块编辑模式）。

## **占位符(Placeholders)**

占位符(Placeholders) 是带默认值的制表位(Tabstops),占位符(Placeholders)的文本会被插入到制表位(Tabstops)所在位置并且全选以方便修改,占位符(Placeholders)可以嵌套使用，比如`${1:another ${2:placeholder}}`。

## **选择项(Choice)**

选择项(Choice)可以有多选值，每个选项的值用 `,` 分隔，选项的开始和结束用管道符号(`|`)将选项包含，例如: `${1|one,two,three|}`，当插入代码片段，选择制制表位(Tabstops)的时候，会列出选项供用户选择。

## **变量(Variables)**

使用 `$name` 或者 `${name|default}` 可以插入变量的值，如果变量未被赋值则插入 `default` 的值或者空值 。当变量未被定义，则将变量名插入，变量(Variables)将被转换为占位符(Placeholders)系统变量如下

- `TM_SELECTED_TEXT` 当前选定的文本或空字符串
- `TM_CURRENT_LINE` 当前行的内容
- `TM_CURRENT_WORD` 光标下的单词的内容或空字符串
- `TM_LINE_INDEX` 基于零索引的行号
- `TM_LINE_NUMBER` 基于一索引的行号
- `TM_FILENAME` 当前文档的文件名
- `TM_FILENAME_BASE` 当前文档的文件名（不含后缀名)
- `TM_DIRECTORY` 当前文档的目录
- `TM_FILEPATH` 当前文档的完整文件路径
- `CLIPBOARD` 剪切板里的内容

插入当前日期或时间：

- `CURRENT_YEAR` 当前年(四位数)
- `CURRENT_YEAR_SHORT` 当前年(两位数)
- `CURRENT_MONTH` 当前月
- `CURRENT_MONTH_NAME` 本月的全名（’七月’）
- `CURRENT_MONTH_NAME_SHORT` 月份的简称（’Jul’）
- `CURRENT_DATE` 当前日
- `CURRENT_DAY_NAME` 当天的名称（’星期一’）
- `CURRENT_DAY_NAME_SHORT` 当天的短名称（’Mon’）
- `CURRENT_HOUR` 当前小时
- `CURRENT_MINUTE` 当前分钟
- `CURRENT_SECOND` 当前秒

当前语言的行注释或块注释:

- `BLOCK_COMMENT_START` 块注释开始标识,如 PHP `/*` 或 HTML `<!--`
- `BLOCK_COMMENT_END` 块注释结束标识,如 PHP `*/` 或 HTML `-->`
- `LINE_COMMENT` 行注释，如： PHP `//` 或 HTML `<!-- -->`

## **变量转换(Variable transforms)**

变量转换(Variable transforms) 允许变量在插入前改变变量的值，变量转换(Variable transforms)由三部分组成

1. 正则匹配：使用正则表达式匹配变量值，若变量无法解析则值为空。
2. 格式串：允许引用正则表达式匹配组。格式串允许条件插入和做简单的修改。
3. 正则表达式匹配选项

下面例子是使用变量转换(Variable transforms)将带后缀的文件名转换为不带后缀的文件名

    ${TM_FILENAME/(.*)\\..+$/$1/}
      |           |         |  |
      |           |         |  |-> 无选项设置
      |           |         |
      |           |         |-> 引用捕获组的第一个分组内容
      |           |             
      |           |
      |           |-> 匹配后缀前的所有字符串
      |               
      |
      |-> 文件名（带后缀）

# **需求实现**

最终未能把输入变量加到输出里（因为prefix需要确定），改为通过批量替换placeholder的方式自己替换

```json
"Angular bind 2-way": {
		"prefix": "ng-tw",
		"body": [
			"$BLOCK_COMMENT_START $3 $BLOCK_COMMENT_END",
			"private _${1:variable}: ${2:any};",
			"@Input()",
			"get ${1:variable}(): ${2:any} {",
			"\treturn this._${1:variable};",
			"}",
			"set ${1:variable}(\\$value: ${2:any}) {",
			"\tthis._${1:variable} = \\$value;",
			"\tthis.${1:variable}Change.emit(\\$value);",
			"}",
			"@Output()", ;
			"${1:variable}Change = new EventEmitter<${2:any}>();",
			"$0"
		],
		"description": "A two-way binding for Angular"
	}
```

# 参考文献

[vscode 自定义代码片段](https://segmentfault.com/a/1190000018457312)

[Snippets in Visual Studio Code](https://code.visualstudio.com/docs/editor/userdefinedsnippets)