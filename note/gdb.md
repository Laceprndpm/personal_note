# Portable

| 命令                         | 说明              | 示例                       |
| -------------------------- | --------------- | ------------------------ |
| `gdb <程序>`                 | 启动 GDB 并加载可执行文件 | `gdb ./a.out`            |
| `run` / `r`                | 运行程序            | `r arg1 arg2`            |
| `quit` / `q`               | 退出 GDB          | `q`                      |
| `start`                    | 启动程序并停在 `main`  | `start`                  |
| `break <位置>` / `b`         | 设置断点            | `b main`，`b file.cpp:25` |
| `tbreak <位置>`              | 设置一次性断点         | `tbreak func`            |
| `delete <编号>`              | 删除断点            | `delete 2`               |
| `disable <编号>`             | 禁用断点            | `disable 1`              |
| `enable <编号>`              | 启用断点            | `enable 1`               |
| `info breakpoints` / `i b` | 查看断点列表          | `i b`                    |
| `watch <表达式>`              | 监视变量变化          | `watch x`                |
| `rwatch <表达式>`             | 监视只读访问          | `rwatch x`               |
| `awatch <表达式>`             | 监视任何访问          | `awatch x`               |
| `continue` / `c`           | 继续运行到下一个断点      | `c`                      |
| `step` / `s`               | 单步进入函数          | `s`                      |
| `next` / `n`               | 单步跳过函数          | `n`                      |
| `finish`                   | 执行到当前函数返回       | `finish`                 |
| `until <行号>`               | 运行到指定行          | `until 50`               |
| `print <表达式>` / `p`        | 打印变量/表达式        | `p x`，`p *ptr`           |
| `display <表达式>`            | 每次停下自动显示        | `display i`              |
| `undisplay <编号>`           | 取消自动显示          | `undisplay 1`            |
| `info locals`              | 查看当前函数局部变量      | `info locals`            |
| `info args`                | 查看当前函数参数        | `info args`              |
| `backtrace` / `bt`         | 打印函数调用栈         | `bt`                     |
| `frame <编号>`               | 切换到指定栈帧         | `frame 1`                |
| `x /<数量><格式><单位> <地址或表达式>` | 查看内存            | `x/10xw &arr[0]`         |
| `set <变量>=<值>`             | 修改变量值           | `set x=100`              |
| `info symbol <地址>`         | 查看地址对应的符号       | `info symbol 0x4005d4`   |
| `p/x <指针>`                 | 打印指针地址          | `p/x ptr`                |
| `p *<指针>`                  | 打印指针指向内容        | `p *ptr`                 |
| `list` / `l`               | 查看源代码           | `l`，`l 20,40`            |
| `list <函数名>`               | 查看函数定义          | `l main`                 |
| `file <程序>`                | 切换调试程序          | `file ./b.out`           |
| `info variables`           | 查看全局变量          | `info variables`         |
| `set pagination off`       | 关闭分页，方便脚本操作     | `set pagination off`     |
| `set logging on`           | 打印日志到文件         | `set logging on`         |
| `source <文件>`              | 执行 GDB 脚本       | `source gdb_script.gdb`  |
| `record`                   | 启用指令级回溯         | `record`                 |
| `reverse-continue`         | 反向运行            | `reverse-continue`       |
| `Ctrl + C`                 | 中断运行            | -                        |
| `Enter`                    | 重复上次命令          | -                        |
| `Ctrl + L`                 | 清屏              | -                        |
| 内存显示格式                     | 说明              | -                        |
| `x`                        | 十六进制            | -                        |
| `d`                        | 十进制             | -                        |
| `u`                        | 无符号十进制          | -                        |
| `o`                        | 八进制             | -                        |
| `t`                        | 二进制             | -                        |
| `f`                        | 浮点数             | -                        |
| `s`                        | 字符串             | -                        |
| `c`                        | 单字符             | -                        |
# GDB 使用手册（Full Command Version）

## 启动与退出
| 命令                 | 说明              | 示例                    |
| ------------------ | --------------- | --------------------- |
| `gdb <程序>`         | 启动 GDB 并加载可执行文件 | `gdb ./a.out`         |
| `run` / `r`        | 运行程序            | `r arg1 arg2`         |
| `start`            | 启动程序并停在 `main`  | `start`               |
| `quit` / `q`       | 退出 GDB          | `q`                   |
| `file <程序>`        | 切换调试程序          | `file ./b.out`        |
| `symbol-file <文件>` | 切换符号文件          | `symbol-file ./a.out` |

## 断点与监视
| 命令 | 说明 | 示例 |
|------|------|------|
| `break <位置>` / `b` | 设置断点 | `b main`，`b file.cpp:25` |
| `tbreak <位置>` | 设置一次性断点 | `tbreak func` |
| `delete <编号>` | 删除断点 | `delete 2` |
| `disable <编号>` | 禁用断点 | `disable 1` |
| `enable <编号>` | 启用断点 | `enable 1` |
| `info breakpoints` / `i b` | 查看断点列表 | `i b` |
| `watch <表达式>` | 监视变量变化 | `watch x` |
| `rwatch <表达式>` | 监视只读访问 | `rwatch x` |
| `awatch <表达式>` | 监视任何访问 | `awatch x` |

## 程序控制
| 命令 | 说明 | 示例 |
|------|------|------|
| `continue` / `c` | 继续运行到下一个断点 | `c` |
| `step` / `s` | 单步进入函数 | `s` |
| `next` / `n` | 单步跳过函数 | `n` |
| `finish` | 执行到当前函数返回 | `finish` |
| `until <行号>` | 运行到指定行 | `until 50` |
| `jump <行号>` | 跳转到指定行执行 | `jump 100` |

## 查看与修改
| 命令 | 说明 | 示例 |
|------|------|------|
| `print <表达式>` / `p` | 打印变量/表达式 | `p x`，`p *ptr` |
| `display <表达式>` | 每次停下自动显示 | `display i` |
| `undisplay <编号>` | 取消自动显示 | `undisplay 1` |
| `set <变量>=<值>` | 修改变量值 | `set x=100` |
| `info locals` | 查看当前函数局部变量 | `info locals` |
| `info args` | 查看当前函数参数 | `info args` |
| `info registers` | 查看寄存器值 | `info registers` |
| `info threads` | 查看线程列表 | `info threads` |
| `thread <编号>` | 切换线程 | `thread 2` |
| `backtrace` / `bt` | 打印函数调用栈 | `bt` |
| `frame <编号>` | 切换到指定栈帧 | `frame 1` |
| `info symbol <地址>` | 查看地址对应的符号 | `info symbol 0x4005d4` |

## 内存与代码查看
| 命令 | 说明 | 示例 |
|------|------|------|
| `x/<格式> <地址>` | 查看内存 | `x/10xw &arr[0]` |
| `list` / `l` | 查看源代码 | `l`，`l 20,40` |
| `list <函数名>` | 查看函数定义 | `l main` |

## 调试脚本与日志
| 命令 | 说明 | 示例 |
|------|------|------|
| `source <文件>` | 执行 GDB 脚本 | `source gdb_script.gdb` |
| `set logging on` | 打印日志到文件 | `set logging on` |
| `set logging off` | 停止日志 | `set logging off` |
| `set logging file <文件名>` | 指定日志文件名 | `set logging file debug.log` |
| `set logging overwrite on/off` | 是否覆盖已有日志文件 | `set logging overwrite on` |
| `set logging redirect on/off` | 是否只写文件不显示终端 | `set logging redirect on` |

## 反向调试
| 命令 | 说明 | 示例 |
|------|------|------|
| `record` | 启用指令级回溯 | `record` |
| `reverse-continue` | 反向运行 | `reverse-continue` |

## 快捷键
| 快捷键 | 说明 |
|--------|------|
| `Ctrl + C` | 中断运行 |
| `Enter` | 重复上次命令 |
| `Tab` | 命令补全 |
| `Ctrl + L` | 清屏 |

## 内存显示格式
| 格式 | 说明 |
|------|------|
| `x` | 十六进制 |
| `d` | 十进制 |
| `u` | 无符号十进制 |
| `o` | 八进制 |
| `t` | 二进制 |
| `f` | 浮点数 |
| `s` | 字符串 |
| `c` | 单字符 |
