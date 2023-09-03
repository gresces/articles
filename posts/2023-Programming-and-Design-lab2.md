---
title: "计算机程序设计实践lab2-文件读写"
date: 2023-09-03T19:55:55+08:00
draft: false
tags: ["计算机", "作业"]
description: 计算机程序设计实践lab2
katex: false
toc: true
summary: 
---

## 前言
这个实验要求做的内容较多，并且给的时间比较少，所以在这里给出一些自己的代码，从而节省时间。

另外对于一些读者来说，其中接触到的一些技术是新的，这在无形之中增加了时间使用量，这里也会大幅度提高难度。所以本文章也会将这部分发出来供大家参考，同时也供作者回顾之用。

## 文本信息读写

### 将文本信息读入并进行处理

文件结构

```bash {linenos=table}
❯ tree
.
├── datatype.h
├── main
├── main.c
├── Makefile
├── testfile.txt
├── txtop.c
├── txtop.h
├── txtwr.c
└── txtwr.h

1 directory, 9 files
```
其中`main.c`为主要的程序入口，提供`txt -> bin`与`bin ->
txt`两种文件转换方式;`datatype.h`提供了本程序所需的结构体数据类型;`txtop.*`与`txtwr.*`分别保存了所需的函数。

#### 文件读入

文件读入和处理的主要函数声明都包含于`txtop.h`中。

```C {linenos=table, hl_lines=[27, 41, 52, 62, 73, 85]}
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

#include "datatype.h"
#define MAXFILE 1024	/*max file bytes*/
#define MAXLINE	128


/*
Input: /path/to/file, Struct Array
Output: -1	failed
		N	linenum = N
		1	parse error

Function: 	read txt file by line
			parse every line
			save info into Struct
Dependency:
	parse_line
	
*/
int file_parse_dat(char * file_dir, Student stus[]);
/*
Input: /path/to/file, Struct Array
Output: -1	failed
		N	linenum = N
		1	parse error

Function: 	read txt file by line
			parse every line
			save info into Struct
Dependency:
	parse_line
	
*/
int file_parse(char * file_dir, Student stus[]);

/*
Input: stdin buf, struct Student pointer
Output: -1	error
		0	success

Function: 	parse line to struct
Dependency:
	delete_space
 */
int parse_line(char * buf, Student * stu);

/*
Input: stdin buf
Output: none

Function:	delete multiple spaces to one
Dependency:
	NO
 */
void delete_space(char * buf);

/*
Input: /path/to/save/file, info Studdent struct array
Output: -1	WRITE file failed
		0	success
		1	error

Function:	print struct data
			save information
 */
int file_save(char * file_dir, Student stus[]);

/*
Input: /path/to/file, Struct Array
Output: -1	failed
		0 	success
		1	parse error

Function: 	read txt file from struct
			ptint
	
*/
int file_read_array(char * file_dir, Student stus[]);
```

相关函数的主要定义在`txtop.c`中

```C {linenos=table}
#include "txtop.h"
#include "datatype.h"

void delete_space(char * buf){
	int i = 0;
    int j = 0;

    int is_first_space = 1;

    while(buf[i] != '\0'){
        if(buf[i] != ' '){
            buf[j] = buf[i];
            j++;
            is_first_space = 1;
        }
        else{
            if(is_first_space == 1){
                buf[j] = buf[i];
                j++;
                is_first_space = 0;
            }
        }
        i++;
    }
	buf[j] = '\0';
//    if((buf[j-2] == ' ') || (buf[j-2] == '\n')){
//        buf[j-2] == '\0';
//    }
}

int parse_line(char * buf, Student * stu){
	
	/*delete space*/
	delete_space(buf);

    unsigned long id = 0;
    char name[32] = "";
    float score[3] = {0};
    int weight = 0;
    char waste[32] = "";

    sscanf(buf, "%lu %s %f %f %f %d", &id, name, &score[0], &score[1],
                    &score[2], &weight);
    stu->id = id;
    strcpy(stu->name, name);
    stu->score[0] = score[0];
    stu->score[1] = score[1];
    stu->score[2] = score[2];
    stu->weight = weight;

	printf("%lu\n", stu->id);

    return 0;	
}


int file_parse_dat(char *file_dir, Student stus[]){
	int fd;			/* file descriptor */
	int nbytes;		/* number of bytes write */

	/*Open file(write)*/
	if((fd = open(file_dir, O_RDONLY)) < 0) {
		perror("Error: Open file error");
		exit(1);
	}

	/*READ file*/
	if((nbytes = read(fd, stus, sizeof(Student) * 10)) < 0){
		perror("Error: READ file error");
	}
	close(fd);

	int l = 0;
	while(stus[l].id != 0){
		l++;
	}
	l--;
	return l;
}

int file_parse(char * file_dir, Student stus[]){

	int 	fd;			/*file descriptor*/
	int		nbytes;		/*number of bytes*/
	int		linenum = 0;	/*line number*/
	char	buf[MAXFILE] = "";
	char	readbuf[MAXLINE] = "";


	/*OPEN file*/
	if((fd = open(file_dir, O_RDONLY)) < 0){
		perror("Error: OPEN file error");
		return -1;
	}

	/*READ file*/
	if((nbytes = read(fd, buf, sizeof(buf)-1)) < 0){
		perror("Error: READ file error");
	}

	/*READ line*/
	for(int i = 0, j = 0; i < MAXFILE; i++){
		if(buf[i] == '\0'){
			break;
		}
		else{
			if(buf[i] == '\n'){
				readbuf[j] = '\0';
				parse_line(readbuf, &stus[linenum]);
				linenum++;
				j = 0;
			}
			else{
				readbuf[j] = buf[i];
				j++;
			}
		}
	}

	linenum--;
	/*CLOSE file*/
	int retval = 0;
	if((retval = close(fd)) < 0){
		perror("Error: Close file error");
		exit(1);
	}

	return linenum;
}

int file_read_array(char * file_dir, Student stus[]){
	int 	fd;			/*file descriptor*/
	int		nbytes;		/*number of bytes*/
	int		line = 0;	/*line number*/
	char	buf[MAXFILE] = "";
	char	readbuf[MAXLINE] = "";


	/*OPEN file*/
	if((fd = open(file_dir, O_RDONLY)) < 0){
		perror("Error: OPEN file error");
		return -1;
	}

	/*READ file*/
	if((nbytes = read(fd, stus, sizeof(Student) * 10)) < 0){
		perror("Error: READ file error");
	}

	/*PRINT*/
	while(stus[line].id != 0){
		char buf[MAXLINE] = {'\0'};
		sprintf(buf, "%lu, %s, %.1f, %.1f, %.1f, %d;\n" ,
						stus[line].id,
						stus[line].name,
						stus[line].score[0],
						stus[line].score[1],
						stus[line].score[2],
						stus[line].weight
						);
		printf("%s", buf);
		line++;
	}

	return 0;
}
```

这两个文件中冗余代码极多，甚至无用代码会超过有用代码，这主要是因为在阅读文档时没有理解意思，多做了一些无用的工作。

#### 文件写入

文件写入的主要函数包含于`txtwr.h`与`txtwr.c`中。

```C {linenos=table}
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

#include "datatype.h"
#define MAXFILE 1024	/*max file bytes*/
#define MAXLINE	128


/*
Input: /path/to/writted/file, struct Student array, linenum
Output:	-1	Write file error
		0	success

Function:	Open file
			Write data
			Close file
 */
int file_write(char * file_dir, Student stus[], int number);

/*
Input: /path/to/writted/file, struct Student array, linenum
Output:	-1	Write file error
		0	success

Function:	Open file
			Write data
			Close file
 */
int file_write_array(char * file_dir, Student stus[], int number);
```

```C {linenos=table}
#include "txtwr.h"
#include "datatype.h"

int file_write_array(char * file_dir, Student stus[], int number){
	int fd;			/* file descriptor */
	int nbytes;		/* number of bytes write */

	/*Open file(write)*/
	if((fd = open(file_dir, O_WRONLY|O_CREAT|O_TRUNC, 0777)) < 0) {
		perror("Error: Open file error");
		exit(1);
	}
	
	/*Write file*/
	if(nbytes = write(fd, stus, sizeof(Student) * (number + 1)) < 0){
		perror("Error: Write file error");
		exit(1);
	}

	/*Close file*/
	int retval = 0;
	if((retval = close(fd)) < 0){
		perror("Error: Close file error");
		exit(1);
	}
	return 0;
}

int file_write(char * file_dir, Student stus[], int number){
	
	int fd;			/* file descriptor */
	int nbytes;		/* number of bytes write */

	/*Open file(write)*/
	if((fd = open(file_dir, O_WRONLY|O_CREAT|O_TRUNC, 0777)) < 0) {
		perror("Error: Open file error");
		exit(1);
	}

	int line = 0;
	while(line <= number){
		char buf[MAXLINE] = {'\0'};
		sprintf(buf, "%lu, %s, %.1f, %.1f, %.1f, %d;\n" ,
						stus[line].id,
						stus[line].name,
						stus[line].score[0],
						stus[line].score[1],
						stus[line].score[2],
						stus[line].weight
						);
		printf("%s", buf);
	/*Write file*/
		if(nbytes = write(fd, buf, strlen(buf)) < 0){
			perror("Error: Write file error");
			exit(1);
		}
		line++;
	}

	/*Close file*/
	int retval = 0;
	if((retval = close(fd)) < 0){
		perror("Error: Close file error");
		exit(1);
	}
}
```

这些操作读者可以自行阅读。显而易见的是，这写代码可拓展性并不好，读者在使用时请进行删改。

#### Makefile

```Makefile {linenos=table, hl_lines=["3-4"]}
CC = gcc
CFLAGS = -I.
DEPS = txtop.h txtwr.h datatype.h
OBJ = main.o txtop.o txtwr.o

DIR = .

%.o: %.c $(DEPS)
	$(CC) -g -c -o $@ $< $(CFLAGS)


main: $(OBJ)
	$(CC) -g -o $@ $^ $(CFLAGS)


.PHONY: clean

clean:
	rm -f $(DIR)/*.o
```

这个`Makefile`文件还有很大的优化的余地，但是在这个小工程里面是足够用的了。高亮的两行说明这个文件还没有实现完全的编译自动化。

