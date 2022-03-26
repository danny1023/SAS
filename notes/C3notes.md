# NOTEs of *Prepering for the SAS Programming Certification Exam*

>相当于前面C1和C2的整理复习

<!-- TOC -->

- [1. base必备知识](#1-base必备知识)
- [2. Week1-*Review of Getting Started with SAS Programming, Part 1*](#2-week1-review-of-getting-started-with-sas-programming-part-1)
    - [2.1. 获取-accessing](#21-获取-accessing)
    - [2.2. 挖掘-exploring](#22-挖掘-exploring)
- [3. Week2-*Review of Getting Started with SAS Programming, Part 2*](#3-week2-review-of-getting-started-with-sas-programming-part-2)
    - [3.1. 预处理-preparing](#31-预处理-preparing)
    - [3.2. 分析-analysing](#32-分析-analysing)
    - [3.3. 导出-exporting](#33-导出-exporting)
- [4. Week3-*Review of Doing More with SAS Programming, Part 1*](#4-week3-review-of-doing-more-with-sas-programming-part-1)
    - [4.1. 数据步-data steps](#41-数据步-data-steps)
    - [4.2. 排序总结-summarizing data](#42-排序总结-summarizing-data)
    - [4.3. 基础函数-functions](#43-基础函数-functions)
    - [4.4. 自定义格式-custom formats](#44-自定义格式-custom-formats)
- [5. Week4-*Review of Doing More with SAS Programming, Part 2*](#5-week4-review-of-doing-more-with-sas-programming-part-2)
    - [5.1. 合并表格-Combining Tables Review](#51-合并表格-combining-tables-review)
    - [5.2. 循环-Processing Repetitive Code Review](#52-循环-processing-repetitive-code-review)
    - [5.3. 报告输出控制-Restructuring Tables Review](#53-报告输出控制-restructuring-tables-review)

<!-- /TOC -->

## 1. base必备知识

1. Read SAS, Excel, and text files in a SAS program
2. Write DATA step code to subset rows and columns, compute new columns, and process data conditionally
3. Sort tables and remove duplicates using the SORT procedure
4. Create and apply SAS formats
5. Summarize data using the MEANS and FREQ procedures
6. Create listing reports using the PRINT procedure
7. Use SAS macro variables to dynamically replace text in a SAS program
8. Export data and results
9. Understand how SAS compiles and executes DATA step code, including how rows are processed in the Program Data Vector
10. Direct DATA step output with the OUTPUT statement
11. Summarize data and process data in groups in the DATA step
12. Use SAS functions to manipulate data values
13. Concatenate and merge tables using the DATA step
14. Process code iteratively in a DATA step using the DO loop.
15. Restructure SAS tables using the DATA step and TRANSPOSE procedure
16. Identify and resolve syntax and logic errors

## 2. Week1-*Review of Getting Started with SAS Programming, Part 1*

### 2.1. 获取-accessing

* 读取`csv`文件：
```sas
proc import dbms=csv datafile="&path/payroll.csv" out=payroll replace;
	guessingrows=max;
run;
/* 查看library中所有的table */
proc contents data=payroll;
run;
```


* 读取`xlsx`文件：
```sas
options validvarname=v7;
libname employee xlsx "&path/employee.xlsx";
/* 查看library中所有的table */
proc contents data=employee._all_;
run;
```

### 2.2. 挖掘-exploring

* print
* freq
* means
* univarite
    - 可以展示极值
* where
* sort

```sas
proc print data=emp_sort;
    where JobTitle like '%Logistics%';
    format salary dollar10. TermDate HireDate BirthDate date9.;
run;

proc freq data=cr.employee_raw order=freq nlevels ;
    tables EmpID Country Department;	
run;

proc means data=emp_sort n mean;
	where HireDate>="01Jan2010"d and TermDate is missing;
	var salary;
run;

proc univariate data=emp_sort;
	var salary;
run;

proc sort data=cr.employee_raw out=emp_sort noduprecs;
    by _all_;
run;
```

## 3. Week2-*Review of Getting Started with SAS Programming, Part 2*

### 3.1. 预处理-preparing

* data

* proc sql
    *考试中不会考核这个部分，所以暂不提供示例*

### 3.2. 分析-analysing

* freq

```sas
proc freq data=cr.profit;
	table Order_Date*Order_Source / nocol norow;
	format Order_Date MONNAME.;
run;
```

* means

```sas
proc means data=cr.employee sum mean min max maxdec=0;
	where Department="Sales";
	class JobTitle;
	var Salary;
	ways 0 1;
run;
```

```sas
proc means data=salary_sorted noprint;
	var Salary;
	class Department City;
	output out=salary_summary sum=TotalSalary mean=AvgSalary; /*重命名变量*/
	ways 2;
run;
```

### 3.3. 导出-exporting

* proc export

```sas
proc export data=cr.employee_current file="&outpath/employee_current.csv" dbms=csv replace;
run;
```

* ODS(output delivery system)

```sas
ods graphics on; /* 绘图需要 */
ods noproctitle;

ods excel file="&outpath/heart.xlsx";
title "Distribution of Patient Status";
proc freq data=sashelp.heart order=freq;
	tables DeathCause Chol_Status BP_Status / nocum plots=freqplot;
run;

title "Summary of Measures for Patients";
proc means data=sashelp.heart mean;
	var AgeAtDeath Cholesterol Weight Smoking;
	class Sex;
run;
ods excel close;
ods proctitle;
```

```sas
ods noproctitle;
ods pdf file="&outpath/truck.pdf" style=Journal startpage=no;/* 不分页 */
title "Truck Summary";
title2 "SASHELP.CARS Table";

proc freq data=sashelp.cars;
	where Type="Truck";
	tables Make / nocum;
run;

proc print data=sashelp.cars;
	where Type="Truck";
	id Make;
	var Model MSRP MPG_City MPG_Highway;
run;

ods pdf close;
ods proctitle;
```

## 4. Week3-*Review of Doing More with SAS Programming, Part 1*

**PDV(program data vector)**

### 4.1. 数据步-data steps

```sas
proc means data=cr.employee_current noprint maxdec=0;
	var Salary;
	class Department;
	ways 1;
	output out=salary sum=TotalSalary;
run;

data salaryforecast;
	set salary;
	Year=1;
	TotalSalary=TotalSalary*1.03;
	output;
	Year=2;
	TotalSalary=TotalSalary*1.03;
	output;
	Year=3;
	TotalSalary=TotalSalary*1.03;
	output;
	format TotalSalary dollar16.;
run;
```

### 4.2. 排序总结-summarizing data

* putlog

```sas
proc sort data=cr.employee_current out=emp_sort;
	by Department Salary;/* 必须先做排序 */
run;

data dept_salary;
	set emp_sort;
	by Department;
	retain LowSalaryJob;
	if first.Department then do;
		TotalDeptSalary=0;
		LowSalaryJob=JobTitle;
	end;
	TotalDeptSalary+Salary;
	if last.department then do;
		HighSalaryJob=JobTitle;
		output;/* 这里输出即可 */
	end;
	keep Department TotalDeptSalary HighSalaryJob LowSalaryJob;
	format TotalDeptSalary dollar12.;
run;
```

### 4.3. 基础函数-functions

* Numeric, Character, Date Functions
* intnx, intck

```sas
data outfield;
	set sashelp.baseball;
	where substr(Position, 2, 1) = "F";
    /* substr字符串截取 */
	Player=catx(" ",scan(Name, 2, ","), scan(Name, 1, ","));
    /* catx字符串拼接 */
	BatAvg=round(nHits/nAtBat, .001);
run;

proc sort data=outfield;
	by descending BatAvg;
run;
```

```sas
data emp_new;
	  set cr.employee_new(rename=(HireDate=HireDateC));
      /* 注意这里的rename部分，构建中间变量 */
	  EmpID=substr(EmpID,4);
	  HireDate=input(HireDateC, anydtdte10.);
      /* anydtdte可以用来自动识别日期，不用人工解析 */
	  Salary=input(AnnualSalary, dollar10.);
	  drop HireDateC;
run;
```

### 4.4. 自定义格式-custom formats

* 直接编写映射关系；
* 从表格中获取关系；

```sas
proc format;
	value $statfmt "S"="Single"
	              "M"="Married"
	              "O"="Other";
	value salrange low-<50000="Under $50K"
	               50000-100000="50K-100K"
	               100000<-high="Over 100K";
run;

proc freq data=cr.employee;
	tables Status;
	tables City*Salary / nopercent nocol;
	format Status $statfmt. Salary salrange.;
run;
```

```sas
data myfmt;
	set cr.continent_codes;
	FmtName="CONTFMT";
    /* 原始数据为数值型，不需要带$ */
	Start=Code;
	Label=Continent;
	keep FmtName Start Label;
run;

proc format cntlin=myfmt;
run;

proc means data=cr.demographics sum maxdec=0;
	class Cont;
	var Pop;
	format Cont CONTFMT.;
run;
```

## 5. Week4-*Review of Doing More with SAS Programming, Part 2*

### 5.1. 合并表格-Combining Tables Review

* concatenating
* merging

>纵向合并前，必须先排序。

```sas
data q3_sales;
	set cr.m7_sales cr.m8_sales cr.m9_sales(rename=(EmpID=Employee_ID));
run;

proc freq data=q3_sales;
	tables Order_Type;
run;
```

```sas
/* 先排序!!! */
proc sort data=cr.employee_addresses(rename=(Employee_ID=EmpID))   
          out=address_sort;
    by EmpID;
run;

data emp_full;
	merge cr.employee(in=inemp) address_sort;
	by EmpID;
	if inemp=1;
run;
```

```sas
/* 先排序 */
proc sort data=cr.employee(keep=EmpID Name Department) out=emp_sort;
	by EmpID;
run;

proc sort data=cr.employee_donations out=donate_sort;
	by EmpID;
run;

data donation nodonation;
	merge emp_sort(in=in_emp) donate_sort(in=in_don);
	by EmpID;
	if in_don=1 and in_emp=1 then do;
		TotalDonation=sum(of Qtr1-Qtr4);
		output donation;
	end;
	else if in_don=0 and in_emp=1 then output nodonation;
run;
```

### 5.2. 循环-Processing Repetitive Code Review

* 循环和条件循环-Iterative and Conditional DO Loops
* 嵌套循环-Nested Do Loops



### 5.3. 报告输出控制-Restructuring Tables Review