### Web Sources Framework

Mp3tag 有一个内嵌的网络源框架，它是通过网络源结构化文件定义抓取。使用这些结构化文件，理论上你可以从所有HTML（不使用JavaScript，ActiveX）的网站通过抓取获得艺术家/专辑信息导入标签数据。也可以在 Mp3tag 论坛的 [Web Sources Archive](http://forums.mp3tag.de/index.php?showtopic=1794) 中找到许多例子。

#### 网络源结构化文件的格式定义

**[Name]**
网络源名称，例如 Discogs.com

**[BasedOn]**
网络源的基础 URL, 例如 http://www.discogs.com

**[IndexUrl]**
搜索URL，例如 http://www.discogs.com/artist/%s。

**[AlbumUrl]**
搜索结果的基本URL，例如 http://www.discogs.com

**[WordSeparator]**
在用户输入的搜索 Query 中，用分隔符代替空白，例如 %20

**[IndexFormat]**
用于将搜索结果分割成不同的标签字段的格式化字符串。%url% 必须有，例如 %url%|%album%|%type%|%label%

**[SearchBy]**
由网络源定义的作为搜索 Query 的字段，你最多可以使用三个不同的字段定义方式，Field Name||%field%||&query=%s. Mp3tag 会为同一类字段定义显示成一个搜索项。这个功能的例子可以在标准的Discogs网站资源中找到。

**[Encoding]**
用于所有URL的编码。可以是utf-8、iso-8859-1、url、url-utf-8或ansi（将使用系统代码页）。

[**ParserScriptIndex**]
这个Key包含一个抓取的多行脚本（以=...开头），它可以解析多个专辑的搜索结果页面。

**[ParserScriptAlbum]**
这个Key包含一个抓取的多行脚本（以=...开头），它可以解析搜索结果中的专辑页面。

**[Include]**
这个key嵌套了多个相同网络源结构化文件相同的部分。被引用的key会覆盖掉引用文件的key。这个功能的例子可以在标准的Discogs.src中找到，它引用了Discogs.inc以减少代码重复。

### Parser Scripts

#### How to Start

首先你需要找到一种方法来识别包含特征信息的行，例如发行年份。Mp3tag 解析器将输出视为行和字符，你告诉解析器如何从起点移动到你想抓取的地方。

Mp3tag使用一个指针，它被定位在文件的开头，并且可以通过几个命令来移动它。那么我们如何将这个指针移动到网络源上的发行年份呢？

We can either move it down N lines or we can tell it to move down until it finds the text "Year:". To do the first we would either use the command `MoveLine N` (where N is the number of lines) or `GotoLine N` -- this either means go down seven lines from where you are or go to the Nth line from the top of the text. To do the search, the command would be `FindLine "Year:"`, which - well - finds the next line from where you are that contains the text "Year:". In all cases the pointer would be moved to the first character of the target line. From there we could tell the pointer to move N steps to the right (`MoveChar N`) or to move to the Nth character in the line (`GotoChar N`) or to position the pointer after the text "Year:" (`FindInLine "Year:"`). All of these commands would result in the pointer being moved to the first digit of the number of hidden files. Please note the difference between the `FindLine` and `FindInLine` commands: The earlier goes through lines from where you are to find a text and places the pointer at the beginning of the line, while the latter looks within the line where the pointer is and positions it after the found text.
Now that we are where we want, we need tell Mp3tag to store the data. To do this, a `Say` command is used, but we have to find a way to tell it what to say and what not. In this case we want to output the rest of the line and so we use the `SayRest` command. An alternative would be the `SayUntil` " " command, which would output everything until a space character is found.

所以，一个抓取发行年份的脚本会是这样的：

```
FindLine "Year:"
FindInLine "Year:"
SayNextNumber
```

你会注意到，这个例子使用的是 `Find-Commands`，而不是 `Move-` 或 `Goto-Commands`。只要你有机会使用查找命令就尽量使用，因为互联网页面往往很快就会发生变化。使用依赖 `Find-Commands` 的脚本比依赖绝对位置的脚本更有可能在原始数据的变化中存活下来。

#### Debugging

仅仅从理论上讲，做这些事情可能会有点棘手，如果你在计算行数或字符时出错，你可能会得到很意外的结果。要检查解析器正在做什么，你可以在你的脚本顶部添加一个Debug "on""debug.out "命令。这将给你一个输出文件，它将一步步地告诉你解析器在做什么，以及为什么你最终会得到一个给定的输出。

很多时候，你甚至想用Debug命令来启动你的脚本，以便在你一步步建立你的脚本之前，看看你实际得到了什么数据来进行解析。

### List of Parser Commands

| Command              | Parameter(s) | Description                                                  |
| -------------------- | ------------ | ------------------------------------------------------------ |
| FindLine             | S n          | 寻找S字符串的第1次或第n次出现的行数（从当前位置开始）        |
| FindLineNoCase       | S n          | 寻找S字符串的第1次或第n次出现的行数（忽略大小写，从当前位置开始） |
| FindInLine           | S n n        | 在当前行中查找S的下1次/第n次出现。如果第3个参数设置为1，没有找到S也不会产生错误 |
| GotoChar             | N            | Skip to the Nth character in the current line                |
| GotoLine             | N            | Go to Nth line (counting from top)                           |
| MoveChar             | N            | Move right/left N characters                                 |
| MoveLine             | N n          | Move down/up N lines (starting from current position). If the second parameter is set to 1, possible errors are ignored. |
| Say                  | S            | 发送S到输出                                                  |
| SayUntil             | S            | 发送S前的everything到输出                                    |
| SayUntilML           | S            | 发送S前的everything到输出，支持跨行                          |
| SayRest              |              | 发送当前行当前位置到行尾到输出                               |
| SayNChars            | N            | 发送后续n个字符到输出                                        |
| SayNextNumber        |              | Outputs the next numeric value from the input.               |
| SayNextWord          |              | Outputs the next word from the input.                        |
| SayOutput            | S            | Send the content of output S to the current output. The output CurrentUrl is always generated at runtime. |
| SayNewline           |              | 输出回车和换行（CR LF）                                      |
| SayRegexp            | S s s        | 输出第一个参数中正则表达式的所有匹配结果，并由第二个参数中的字符串分隔。如果提供了第三个参数，则只执行匹配，直到达到第三个参数的内容。如果找不到第三个参数内容，则忽略该行。 |
| Set                  | S s          | Sets the content of output S to the value s. Resets the content if s is omitted. |
| SkipChars            | S            | Skip any characters contained in S                           |
| If                   | S            | Check for occurrence of S on current position.               |
| IfNot                | S            | Check for absence of S on current position.                  |
| IfOutput             | S            | Check for content in output buffer named S.                  |
| IfNotOutput          | S            | Check for absence of content in output buffer named S.       |
| Else                 |              | Else branch of an If operation.                              |
| Endif                |              | End of an If/IfNot/IfOutput/IfNotOutput operation.           |
| OutputTo             | S            | Sets the name of the output buffer of the Say commands to S. |
| Do ... While         | S n          | Execute the command surrounded by the two commands while S occurs on current position. The optional second parameter limits the execution of the loop to maximal n times.<br />`Do`<br />`MoveLine 1`<br/>`While "<td>"`<br />Nesting of Do commands is not allowed. |
| Replace              | S S          | 用第二个参数代替第一个参数的所有出现。                       |
| RegexpReplace        | S S          | 将第一个参数中正则表达式匹配的所有内容替换为第二个参数中的字符串。 |
| JoinUntil            | S            | Joins the current line to the next occurrence of S.          |
| JoinLines            | N            | Joins the current line with the next N lines.                |
| KillTag              | S s          | Replaces tag S with s in current line (or blank if omitted). |
| Unspace              |              | Removes leading and trailing spaces from the current line.   |
| Debug                | S s n        | 调试输出，S设置为"on"或"off"，s是可选的文件名，n是可选的调试文件的最大文件尺寸，单位为MB。 |
| json                 | S s          | JSON输入模式，S设置为 "on"或 "off"，如果在JSON输入模式下，输入会被解析为JSON数据结构，并可以使用以下JSON相关函数进行访问。可选的第二个参数可以设置为 "current"，以将当前的、可能经过转换的输入当作JSON输入。 |
| json_foreach         | S            | 在JSON输入模式下，开始对S的JSON数组进行遍历，This scripting function emits the size of the accessed array on the input which can be then used inside the web source script. 遍历必须以json_foreach_end结束。 |
| json_foreach_end     |              | If in JSON input mode, ends iteration of the last iteration started by json_foreach. |
| json_select_object   | S            | 在JSON输入模式下，选择用S表示的JSON对象，并使其字段可供json_select使用。如果成功，则在当前位置可以使用对象名S。 |
| json_unselect_object |              | 在JSON输入模式下，离开当前JSON对象，返回到上一级对象         |
| json_select          | S            | 在JSON输入模式下，选择S所表示的JSON元素，并将其内容发送到输入端。 |
| json_select_array    | Sns          | If in JSON input mode, selects the nth element of the JSON array denoted by S to the input. If n is -1, all elements are emitted delimited by s. |
| json_select_many     | SSs          | If in JSON input mode, selects all objects from the JSON array denoted by the first parameter and emits their corresponding elements denoted by the second parameter to the input, delimited by s. |

N 大写N是必需的数字参数
S 大写S是所需的字符串参数（使用双引号）
n 小写n是可选数字参数
s 小写s是可选字符串参数（使用双引号）