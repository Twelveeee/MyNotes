C不记简单的

# 基础

#### 头文件

```c
#include<stdio.h>
```

#### 放在屁股暂停

```c
system("pause");
```

# 结构体

## 定义

```c
//第一种
Struct Product//结构体名
{
 char cname// 成员列表
}product1,product2 ;

//第二种
struct Product product1;
struct Product product2;
```

## 引用

```c
//引用
product1.cname="icebox"
//或者
Struct Produc
{
 char cnam
}product1={“icebox"} ;
strcpy (product1->cname,"icebox");


stuct Product *pproduct;
pproduct=& product 1;
pproduct->product1;

```



# 链表

malloc函数原型

void *malloc(unsigned int size);

//在内存中动态的分配一块SIZE大小的内存空间。



calloc 函数原型

void *calloc(unsigned n,unsigned size);

//在内存中动态分配N个长度为SIZE的连续内存空间数组



free函数原型

void free(void *ptr);

//使用由指针ptr指向的内存区，使部分内存区能被其他变量使用。



```c
struct Student//创建节点结构
{
	char cName[20];
	int iNumber;
	struct student* pNext;  //指向下一个节点的指针
};
int iCount;				//全局变量显示链表长度
struct Student *Create()
{
	struct Student* pHead=NULL; 		//初始化链表头指针为空
	struct Student* pEnd,*pNew;
	iCount=0;
	pEnd=pNew=(struct Student*)malloc(sizeof(struct Student));	//初始化链表长度
	printf(“please first enter Name,then Number\n”);
	scanf(“%s”,&pNew-> cName);
	scanf(“%d”,&pNew->iNumber);
while(pNew->iNumber!=0)
{
	iCount++;
	if(iCount==1)
	{
		pNew->pNext=pHead;		//使得指向为空
		pEnd=pNew;				//跟踪新加入的节点
		pHead=pNew;				//头指针指向首节点
}
else
{
pNew->pNext=NULL; 		//新节点的指针为空
		pEnd->pNext =pNew;		//原来的尾节点指向新节点
		pEnd =pNew;				//*pEnd指向新节点
}
pNew=(struct Student*)malloc(sizeof(struct Student));//再次分配节点内存空间
scanf(“%s”,&pNew-> cName);
	scanf(“%d”,&pNew->iNumber);
}
free(pNew); //释放没有用到的空间
return pHead;
}
//输出链表
void Print(struct Student *pHead)
{
	struct Student *pTemp;			//循环所用的临时指针
	int iIndex=1;					//表明链表中节点的序号
	printf(“ –the list has %d members: ---\n\n”,iCount);
	pTemp=pHead;				//得到首节点的地址

	while(pTemp!=NULL)
	{
		printf(“the No.%d stuednt is :\n”,iIndex);
		printf(“the name is %s\n”,pTemp->cName);
		printf(“the number is :%d\n\n”,pTemp->iNumber);
		pTemp =pTemp->pNext;		//临时指针移动到下一个节点
		iIndex++;
}
}
//主函数
int main()
{
	struct Student *pHead; 	//定义头节点
	pHead =Create();		//创建结点
Print(pHead);			//输入链表
return 0;				
}

```

