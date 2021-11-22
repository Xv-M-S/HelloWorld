# 操作系统实验

## **一、进程相关编程实验**

### 实验内容：

![image-20211023173219995](https://raw.githubusercontent.com/Xv-M-S/Typero-images/main/typora-user-images/image-20211023173219995.png)

### 实验过程（a）：

1. **通过notepad++编写教材P103页c程序**

   ```C
   #include<sys/types.h>
   #include<stdio.h>
   #include<sys/wait.h>
   #include<unistd.h>
   
   int main(){
   	pid_t pid,pid1;
   	
   	//创建一个子进程
   	pid=fork();
   	
   	if(pid<0){
   		//出现错误-创建子进程失败
   		fprintf(stderr,"Fork Failed");
   		return 1;
   	}else if(pid==0){
   		//子进程
   		pid1=getpid();
   		printf("child:pid=%d\n",pid);/*A*/
   		printf("child:pid1=%d\n",pid1);/*B*/
   	}else{
   		//父进程
   		pid1=getpid();
   		printf("parent:pid=%d\n",pid);/*C*/
   		printf("parent:pid1=%d\n",pid1);/*D*/
   		wait(NULL);
   	}
   	return 0;
   }
   ```

2. **通过openEuler操作系统编译并运行该C程序**

   ![image-20211023220638672](https://raw.githubusercontent.com/Xv-M-S/Typero-images/main/typora-user-images/image-20211023220638672.png)

3. **去掉wiat()后的输出**

   ![image-20211023220924347](https://raw.githubusercontent.com/Xv-M-S/Typero-images/main/typora-user-images/image-20211023220924347.png)

### 实验过程（a）理论分析：

- **fork（）函数创建一个子进程：fock函数调用一次却返回两次；向父进程返回子进程的ID，向子进程中返回0。**
- **getpid()函数返回当前进程的ID**
- **于是在子进程中：pid=0,pid1=子进程ID；而在父进程中，pid=子进程ID，pid1=父进程ID**
- **wait(NULL)的功能：父进程一旦调用了wait就立即阻塞自己，由wait自动分析是否当前进程的某个子进程已经退出，如果让它找到了这样一个已经变成僵尸的子进程，wait就会收集这个子进程的信息，并把它彻底销毁后返回；如果没有找到这样一个子进程，wait就会一直阻塞在这里，直到有一个出现为止。**
- **去掉wait(NULL)后，如果先终止父进程,子进程将继续正常进行，只是它将由init进程(PID 1)继承,当子进程终止时,init进程捕获这个状态.于是输出没有变化。**

### 实验过程（b）：

1. **修改源文件，增加一个全局变量**

   ```C
   #include<sys/types.h>
   #include<stdio.h>
   #include<sys/wait.h>
   #include<unistd.h>
   
   int main(){
   	pid_t pid,pid1;
   	
   	//创建一个子进程
   	pid=fork();
   
   	int count=0;//增加的全局变量
   
   	if(pid<0){
   		//出现错误-创建子进程失败
   		fprintf(stderr,"Fork Failed");
   		return 1;
   	}else if(pid==0){
   		//子进程
   		pid1=getpid();
   		printf("child:pid=%d\n",pid);/*A*/
   		printf("child:pid1=%d\n",pid1);/*B*/
   		++count;//子进程对全局变量进行加一操作
   		printf("子进程count:%d\n",count);
   	}else{
   		//父进程
   		pid1=getpid();
   		printf("parent:pid=%d\n",pid);/*C*/
   		printf("parent:pid1=%d\n",pid1);/*D*/
   		--count;//父进程对全局变量进行减一操作
   		printf("父进程count:%d\n",count);
   		wait(NULL);
   	}
   	
   	return 0;
   }
   ```

2. **程序运行结果**

   ![image-20211023221237752](https://raw.githubusercontent.com/Xv-M-S/Typero-images/main/typora-user-images/image-20211023221237752.png)

### 实验过程（b)理论分析：

- **一个程序一旦调用fork函数，系统就为一个新的进程准备了前述三个段，首先，系统让新的进程与旧的进程使用同一个代码段，因为它们的程序还是相同的，对于数据段和堆栈段，系统则复制一份给新的进程，这样，父进程的所有数据都可以留给子进程，但是，子进程一旦开始运行，虽然它继承了父进程的一切数据，但实际上数据却已经分开，相互之间不再有影响了，也就是说，它们之间不再共享任何数据了。**
- **所以在调用fork()之后，父进程和子进程都各自有一个独立的数据段存储着一个初始值为0的count变量。父进程对其数据段的count变量进行减一操作，故输出的count为-1；子进程对其数据段的count变量进行加一操作，故输出的count为1.**

### 实验过程（c）:

1. **在子进程中增加system（）函数**

   ```C
   #include<sys/types.h>
   #include<stdio.h>
   #include<sys/wait.h>
   #include<stdlib.h>
   #include<unistd.h>
   
   int main(){
   	pid_t pid,pid1;
   	
   	//创建一个子进程
   	pid=fork();
   
   	int count=0;//增加的全局变量
   
   	if(pid<0){
   		//出现错误-创建子进程失败
   		fprintf(stderr,"Fork Failed");
   		return 1;
   	}else if(pid==0){
   		//子进程
   		//调用system函数
   		system(NULL);
   		pid1=getpid();
   		printf("child:pid=%d\n",pid);/*A*/
   		printf("child:pid1=%d\n",pid1);/*B*/
   		++count;//子进程对全局变量进行加一操作
   		printf("子进程count:%d\n",count);
   	}else{
   		//父进程
   		pid1=getpid();
   		printf("parent:pid=%d\n",pid);/*C*/
   		printf("parent:pid1=%d\n",pid1);/*D*/
   		--count;//父进程对全局变量进行减一操作
   		printf("父进程count:%d\n",count);
   		wait(NULL);
   	}
   	
   	return 0;
   }
   ```

2. **程序运行结果**

   ![image-20211023224558890](https://raw.githubusercontent.com/Xv-M-S/Typero-images/main/typora-user-images/image-20211023224558890.png)

### 实验过程 （c）分析：

- **system()函数执行了三步操作：**
  - **fork（）一个子进程；**
  - **在子进程中调用exec函数去执行command；**
  - **在父进程中调用wait去等待子进程结束。 对于fork失败，system()函数返回-1。 如果exec执行成功，也即command顺利执行完毕，则返回command通过exit或return返回的值。**

### 实验过程（d):

1. **在子进程中添加execlp()函数**

   ```C
   #include<sys/types.h>
   #include<stdio.h>
   #include<sys/wait.h>
   #include<unistd.h>
   
   int main(){
   	pid_t pid,pid1;
   	
   	//创建一个子进程
   	pid=fork();
   
   	int count=0;//增加的全局变量
   
   	if(pid<0){
   		//出现错误-创建子进程失败
   		fprintf(stderr,"Fork Failed");
   		return 1;
   	}else if(pid==0){
   		//子进程
   		//调用execlp函数
   		execlp("bin/ls","ls",NULL);
   		pid1=getpid();
   		printf("child:pid=%d\n",pid);/*A*/
   		printf("child:pid1=%d\n",pid1);/*B*/
   		++count;//子进程对全局变量进行加一操作
   		printf("子进程count:%d\n",count);
   	}else{
   		//父进程
   		pid1=getpid();
   		printf("parent:pid=%d\n",pid);/*C*/
   		printf("parent:pid1=%d\n",pid1);/*D*/
   		--count;//父进程对全局变量进行减一操作
   		printf("父进程count:%d\n",count);
   		wait(NULL);
   	}
   	
   	return 0;
   }
   ```

2. **程序运行结果：**

   ![image-20211023224808692](https://raw.githubusercontent.com/Xv-M-S/Typero-images/main/typora-user-images/image-20211023224808692.png)

### 实验过程（d）分析：

**通过系统调用execlp()，子进程采用UNIX命令/bin/ls()（用来列出目录清单）来覆盖其地址空间。**

## 二、线程相关编程实验

![image-20211023214314383](https://raw.githubusercontent.com/Xv-M-S/Typero-images/main/typora-user-images/image-20211023214314383.png)

### 实验过程：

1. **通过多线程编写一个求和的程序，主函数为一个线程，求和函数为一个线程，两个线程采取分叉-连接策略：即父进程创建一个或多个子线程后，应等待所有子线程终止才恢复执行。**

   ```c
   #include<pthread.h>
   #include<stdlib.h>
   #include<stdio.h>
   
   int sum;//全局声明的数据可为同一进程的所有线程共享
   void* runner(void* param);//特定函数-执行累加求和
   
   int main(int argc, char* argv[]) {
   	pthread_t tid;//声明创建线程的表示符
   	pthread_attr_t attr;//表示线程属性
   	pthread_attr_init(&attr);//设置线程属性
   	if (argc != 2) {
   		fprintf(stderr, "usage:a.out<integer value>\n");
   		return -1;
   	}
   	if (atoi(argv[1]) < 0) {
   		fprintf(stderr, "%d must be>=0\n", atoi(argv[1]));
   		return -1;
   	}
   	pthread_create(&tid, &attr, runner, argv[1]);//创建累加线程
   	pthread_join(tid, NULL);//等待runner线程的完成
   
   	printf("sum=%d\n", sum);//输出求和的结果
   }
   
   void* runner(void* param) {
   	int i, upper = atoi(param);
   	sum = 0;
   	for (int i = 0; i <= upper; i++) {
   		sum += i;
   	}
   	pthread_exit(0);//累加线程终止
   }
   ```

2. **编译的过程中pthread库要手动链接：gcc pthread.c -lpthread -o main**

3. **执行的过程向main函数传递一个参数n，表示求从1到n的累加和,例如：./mian 18**

4. **程序运行结果：**

   ![image-20211023214931453](https://raw.githubusercontent.com/Xv-M-S/Typero-images/main/typora-user-images/image-20211023214931453.png)

### 实验分析：

- **该程序通过调用Pthread库来实现多线程。**
- **首先通过pthread_t tid声明创建线程的标识符、通过pthread_attr_t attr标识线程属性，并通过调用pthread_attr_init(&attr)设置这些属性。**
- **通过调用pthread_create()函数来创建一个单独的线程，此处该线程用于求累加和。**
- **初始父线程main()通过调用pthread_join()函数等待runner（）线程完成。累加和线程通过调用pthread_exit()之后就会终止。一旦累加和线程终止，父线程就会输出累加和的值**
- **该程序中的线程通过在函数外声明的全局变量来进行消息的传递。全局声明（即在函数之外声明）的任何数据，可为同一进程的所有线程所共享。**
- **该程序通过分叉-连接策略（即父进程创建一个或多个子线程后，等待所有子线程终止才恢复执行）来实现同步线程**
