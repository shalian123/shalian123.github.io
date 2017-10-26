---
title: c++动态申请一维、二维数组并释放
date: 2017-05-15 08:59:59
tags: [动态二维数组,c++]
categories: c/c++

---
在平时写代码的过程中，常常会需要动态开辟一个二维数组，具体的行和列不再是常量，之前遇到了总是容易忘记，现在特地整理一下。同时将一维数组也整理了一下。
c++中有三种方法来动态申请多维数组

**(1)C中的malloc/free**

**(2)C++中的new/delete**

**(3)STL容器中的vector**

**第一种malloc/free**
动态开辟一维数组
```
动态开辟一维数组
void dynamicCreate1Array()
{
	int m;
	int i;
	int *p;
    
	printf("请输入开辟的数组长度：");
	scanf("%d",&m);
	p = (int*)malloc(sizeof(int)*m);//动态开辟

	printf("请输入数据：");
	for(i = 0; i < m ; i++)
		scanf("%d",&p[i]);

    printf("输出数据：\n");
	for(i = 0; i < m; i++)
		printf("%d ",p[i]);
	free(p);
}
```
动态开辟二维数组
```
void dynamicCreate2Array()
{
	int m,n;
	int i,j;
	int **p;

    printf("请输入数组行和列：");
	scanf("%d%d",&m,&n);

	p = (int**)malloc(sizeof(int*)*m); //开辟行

	for(i = 0; i < m; i++)
	{
		*(p+i) = (int*)malloc(sizeof(int)*n);//开辟列
	}
	//输入数据
	printf("请输入数：");
    for(i = 0 ; i < m;i++)
		for(j = 0; j < n;j++)
			scanf("%d",&p[i][j]);
    
	//输出数据
	for(i = 0 ; i < m;i++)
	{
		for(j = 0; j < n;j++)
		{
		   printf("%3d ",p[i][j]);
		}
		printf("\n");
	}
	//释放开辟的二维空间
	for(i = 0; i < m;i++)
		free(*(p+i));
}
```
**第二种：new/delete**
1.动态开辟一维数组
```
void DynamicCreate1Array()
{
	int len;
	
	cout<<"请输入长度：";
	cin>>len;
	
	int *p = new int[len];
	
	cout<<"请输入数据：";
    for(int i = 0; i < len; i++)
		cin>>p[i];
	
	cout<<"输出数据："<<endl;
	for(i = 0; i < len; i++)
		cout<<setw(4)<<p[i];
	
	delete[] p;
}
```
2.动态开辟二维数组
```
void DynamicCreate2Array()
{
	int m, n;
	cout << "请输入行和列：";
	cin >> m >> n;

	//动态开辟空间
	int **p = new int*[m]; //开辟行
	for (int i = 0; i < m; i++)
		p[i] = new int[n]; //开辟列

	cout << "请输入数据：";
	for (int i = 0; i < m; i++)
	for (int j = 0; j < n; j++)
		cin >> p[i][j];

	cout << "输出数据：" << endl;
	for (int i = 0; i < m; i++)
	{
		for (int j = 0; j < n; j++)
			cout << setw(3) << p[i][j];
		cout << endl;
	}

	//释放开辟的资源
	for (int i = 0; i < m; i++)
		delete[] p[i];
	delete[] p;

}
```
**第三种：STL中的vector**
动态开辟二维数组
```
void VectorCreate()
{
	int m,n;
	cout<<"请输入行和列：";
	cin>>m>>n;
	
	//注意下面这一行：vector <int后两个 "> "之间要有空格！否则会被认为是重载 "> > "。 
    vector<vector<int> > p(m,vector<int>(n));
	
	cout<<"请输入数据：";
	for(int i = 0 ; i < m ; i++)
		for(int j = 0; j < n; j++)
			cin>>p[i][j];
		
	cout<<"输出数据："<<endl;
	for(int i = 0; i < m; i++)
	{
    	for(int j = 0; j < n; j++)
			cout<<setw(3)<<p[i][j];
		cout<<endl;
	}
			
}
```