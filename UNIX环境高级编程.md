## UNIX环境高级编程

`$ ls > file.text`  将ls打印到文件

`cp aa bb` 拷贝文件aa到bb

UNIX中打开文件都由文件件描述符引, 文件描述符定义在`<unistd.h>`文件中: 

```
#define	 STDIN_FILENO	0	/* standard input file descriptor */
#define	STDOUT_FILENO	1	/* standard output file descriptor */
#define	STDERR_FILENO	2	/* standard error file descriptor */

```

`$ od -c` 查看文件内容, 使用字符形式显示

`ln -s source_file target_file` 创建链接一个符号链接文件

`size` 查看对象大小


