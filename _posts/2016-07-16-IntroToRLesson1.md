---
layout: post
title:  "Learning R: Lesson 1"
categories: [R]
tags: [jekyll]
---



## Introduction

R is an extremely powerful statistical scripting language.  It is open-source and quickly gaining traction across academia, research organizations, and businesses.  It is often the tool of choice for statisticians, data scientists, quantitative financial analysts, and a myriad of other professions.  It is used for research at the vast majority of graduate schools.  It is currently used by companies like Facebook, Google, the NY Times and Wallstreet financial organizations.  

R is open-source and is freely available to download.  You can put base R on any government computer.  You can use base R as-is to write and run R script.  That being said, RStudio has provided a very useful "front-end" for R that is generally easier to use (R is still the "engine"; you can't run RStudio without R).  We will primarily use RStudio in SE350.  Some DoD organizations, however, will not allow you to install RStudio.  Remember though, you can still run everything just like we teach you from base R.

In R you run programs and calculations from a command line.  While you will sometimes write directly in the command line (to explore data or run simple calculator computation), it is usually best to write your code in a \*.R file.  This file serves as "scratch-paper" for you.  It is much easier to edit and adjust your code when it is contained on the "scratch-paper".  To open a \*.R file, select *File -> New File -> R Script*.  Now you have blank "scratch-paper"

---

##Installation

1. Install Base R by going to http://cran.r-project.org/bin/windows/base/ 
2. Install RStudio by going to http://www.rstudio.com/products/rstudio/download/


##R Environment and Workspace

R is always pointing to a certain folder on your computer.  This is called your *working directory*.  R will always directly read files and write files to this directory.  You can see your working directory by typing 


{% highlight r %}
getwd()
{% endhighlight %}



{% highlight text %}
## [1] "/Users/iMac/Documents/gitStuff/dmbeskow.github.io/_source"
{% endhighlight %}

If you want to change where your working directory is, you can do this two ways.  If you are using RStudio, you can go to *Session -> Set Working Directory*.  If you want to change your *working directory* using a command (especially if you're using base R), then you can type


{% highlight r %}
setwd("C:/Users/david.beskow/Google Drive/DataAnalysisLessons")  ###Make sure you use Forward Slashes
{% endhighlight %}

If you want to see the names of files in your working directory without opening Windows Explorer, you can use the command 


{% highlight r %}
dir()
{% endhighlight %}



{% highlight text %}
## [1] "2014-09-28-jekyll-with-knitr.Rmd"
## [2] "2016-07-11-test-rmarkdown.Rmd"   
## [3] "2016-07-16-IntroToRLesson1.Rmd"  
## [4] "Makefile"
{% endhighlight %}
Note that this gives the names of the files in your working directory, which saves you the time of opening up Windows Explorer to remind yourself what you names your data file.  



Types and Shape of Data
----
Before we get into data, I first want to show you that your command line can operate like a calculator


{% highlight r %}
5+4+7*7
{% endhighlight %}



{% highlight text %}
## [1] 58
{% endhighlight %}

or


{% highlight r %}
pi*7.2^2
{% endhighlight %}



{% highlight text %}
## [1] 162.8602
{% endhighlight %}

Note that in both of these examples, the answer is printed to the screen, but not stored in memory.  In other words, I cannot access that answer without redoing the calculation.  If I want to store it in memory, then I assign the answer to a given computation to a name. We use the symbol <- to mean "assign".  In other words, the result of the computation on the right of the symbol is assigned to the name on the left of the symbol. For example:


{% highlight r %}
x<-4*4
{% endhighlight %}

I have now assigned the result of my computation to the name x.  If I want to see this value of x in the future, I can just type it in the console.


{% highlight r %}
x
{% endhighlight %}



{% highlight text %}
## [1] 16
{% endhighlight %}

And I can also use it in future computations:


{% highlight r %}
y<-x/2
{% endhighlight %}
x is now stored in your *Global Environment*.  Think of this as your "workbench" that contains all of the data and values that you are working on.  In RStudio, you can usually see what is in your *Global Environment* in the top right part of the RStudio window.  If you're using base R, you can list the variables that are in your *Global Environment* by typing 


{% highlight r %}
ls()
{% endhighlight %}



{% highlight text %}
## [1] "x" "y"
{% endhighlight %}

When you close either RStudio or base R, it will ask you if you want to save your *work space*.  It is essentially asking you if you want to save what is on your workbench.  If you choose "yes", then it will save an *.RData file of everything that is in your workspace in your working directory.  If you restart R from this working directory, it will load all of these items into your workspace.  Generally it is not a good idea to save your workspace as long as you have all of the code it would take to quickly recreate all of the items in your workspace.  However, if you have some code that takes along time to run, then it is best to save these items in a workspace so that you don't have to wait hours/days a second time to recreate them.  For example, I created some R code to "clean" Afghanistan Blue Force Tracker data.  It took approximately 11 days to clean the data.  In this case, I would want to save my results so I don't have to wait 11 days again for this to run.  In general, however, R takes seconds to run, and it is best to not save your workspace as long as you have clean and easy to run code.

##Input/Output Data

Now that we have all of that done, let's learn how to read and write data.  To do this, we will read in the birth data that was provided to you.  This is a 10% random sample of births in the United States in 2006. 

A key for the variables is given below:

Name    |  Description
--------|-------------
DOB_MM  | Month of date of birth
DOB_WK  | Day of week of birth
MAGER   | Mother's age
TBO_REC | Total birth order
WTGAIN  | Weight gain by mother
SEX     | a factor with levels F M, representing the sex of the child
APGAR5  | APGAR score
DMEDUC  | Mother's education level
UPREVIS | Number of prenatal visits
ESTGEST | Estimated weeks of gestation
DMETH_REC | Delivery Method
DPLURAL | "Plural Births;" levels include 1 Single, 2 Twin, 3 Triplet, 4 Quadruplet, and 5 Quintuplet or higher
DBWT |  Birth weight, in grams

We use the command *read.csv* to read in data.  We also make sure to assign this to a name


{% highlight r %}
births<-read.csv("births.csv",as.is=TRUE)
{% endhighlight %}



{% highlight text %}
## Warning in file(file, "rt"): cannot open file 'births.csv': No such
## file or directory
{% endhighlight %}



{% highlight text %}
## Error in file(file, "rt"): cannot open the connection
{% endhighlight %}

Now that we've read the file in, let's check on its size.  


{% highlight r %}
dim(births)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'births' not found
{% endhighlight %}

This command gives us the number of rows and columns in the data set.  We see that there are 427,323 records of 14 variables.  I usually also use the command *head* to let me see the first 5 rows.  This gives titles of the variables (columns) as well as a feel for the data:


{% highlight r %}
head(births)
{% endhighlight %}



{% highlight text %}
## Error in head(births): object 'births' not found
{% endhighlight %}

Finally, if we want to get a comprehensive summary of all of the variables, we can use the *summary* command


{% highlight r %}
summary(births)
{% endhighlight %}



{% highlight text %}
## Error in summary(births): object 'births' not found
{% endhighlight %}

This give is the min, max, median, mean as well as the 1st and 3rd quantile (remember that the median is the 2nd quantile).  For example, looking at the *MAGER* variable we see that the youngest woman to give birth was 12 years old, the oldest woman to give birth was 50 years old, with the mean age of 27.37.

We will play with the birth data later.  Now that we understand how to get data into R, you may need to push data out of R. We usually do this by writing it to a comma separated value (csv) file.  These are raw data files that Microsoft Excel can read.  You can write to a csv by using the command *write.csv*.  To learn more about this command, see *help(write.csv)*.  Note that you can use this on any command to learn more about it or to remember what the arguments are.



Data can have different classes of data.  The basic building blocks are integer, numeric, character, date, boolean (logical) or factor classes.  The first three should be self explanatory, and examples of all three are below:


{% highlight r %}
x<-4                   #integer
x<-4.56                #numeric
x<-TRUE                #boolean
x<-"Start the Corps!"  #character
{% endhighlight %}

Use the class command to find out what type of data you have. Note that because we were using x for all three, that we were writing over the value of x.  At the end of running these three lines of code, x would equal the last line of code: the character string "Start the Corps!"


{% highlight r %}
class(x)
{% endhighlight %}



{% highlight text %}
## [1] "character"
{% endhighlight %}

R does not automatically recognize *date* data.  When you read *date* data into R, it is initially converted to *character* data.  If you want R to recognize it as a *date*, you need to explicity change it


{% highlight r %}
x<-"2014-01-01"
x<-as.Date(x)
class(x)
{% endhighlight %}



{% highlight text %}
## [1] "Date"
{% endhighlight %}

There is also a type of data called *factor*.  This is character data that has a numeric value tied to it for certain types of models.  Character data is often coerced to the *factor* class when you have nominal data. You can think of a list of data that has either "male" or "female".  If I change this into a factor, it will still be represented as "male" and "female", but it will also be represented numerically.  You need to be very careful when using factors, since many of the functions in R can't handle factor data.  You can see the use of factor data below:


{% highlight r %}
y<-c("male","male","female","male","female")
{% endhighlight %}
This is character data.  If I tried to plot y right now, R would show an error, since you can't print *character* data.  Lets convert this to a factor now:


{% highlight r %}
y<-as.factor(y)
y
{% endhighlight %}



{% highlight text %}
## [1] male   male   female male   female
## Levels: female male
{% endhighlight %}
Now watch when I try to plot this:

{% highlight r %}
plot(y)
{% endhighlight %}

![plot of chunk unnamed-chunk-19](/knitr-jekyll/figure/source/2016-07-16-IntroToRLesson1/unnamed-chunk-19-1.png)

It plots a barchart because R recognizes this as a factor and has a numeric value associated with both of the "levels" in the factor



There are also different dimmensions of data.  So far we've been using *scalars*, in which our variable x is a single value.  Data can have 1, 2, or many dimensions, however.  A one dimensional list of data is known as a *vector*.  An example of a vector is given below:


{% highlight r %}
x<-c(1,6,3,9,8,2)
{% endhighlight %}


If you need to create a vector of sequential integers, you can use a colon:


{% highlight r %}
x<-c(1:10)
x
{% endhighlight %}



{% highlight text %}
##  [1]  1  2  3  4  5  6  7  8  9 10
{% endhighlight %}

If you need to create a vector of the same number, you can use the repeat command:


{% highlight r %}
rep(1,10)  # Repeat 1 ten times
{% endhighlight %}



{% highlight text %}
##  [1] 1 1 1 1 1 1 1 1 1 1
{% endhighlight %}

I can refer to certain elements of a vector using subscripts or a boolean vector.  Using subscripts it looks like


{% highlight r %}
a <- c(1,2,5.3,6,-2,4) # numeric vector
a[c(2,4)] # 2nd and 4th elements of vector
{% endhighlight %}



{% highlight text %}
## [1] 2 6
{% endhighlight %}

To use a boolean vector, it would look like this:

{% highlight r %}
a <- c(1,2,5.3,6,-2,4) # numeric vector
c <- c(TRUE,TRUE,TRUE,FALSE,TRUE,FALSE) #logical vector
a[c]
{% endhighlight %}



{% highlight text %}
## [1]  1.0  2.0  5.3 -2.0
{% endhighlight %}

Notice that this will produce a vector of only those elements of "a" that had TRUE in their respective location in the "c" vector.  

We can also work with two dimensional data.  This is the most common way that data comes.  You can think of an excel sheet that somes with rows and columns.  There are several ways of storing two dimensional data in R; we will primarily focus on data frames.  Data frames are the most common method of pulling data into R.  In a data frame, each column must be the same type of data (all numeric, all character, etc.), but the columns don't have to have the same type of data as other columns.  I can therefore have a column with date data, a column with numeric data, and a column with character data.  All columns in a dataframe must have the same length.  


{% highlight r %}
d <- c(1,2,3,4)
e <- c("red", "white", "red", NA)
f <- c(TRUE,TRUE,TRUE,FALSE)
mydata <- data.frame(ID=d,Color=e,Passed=f)
{% endhighlight %}
Use the *name* command to access and/or change the names of each column in a data.frame


{% highlight r %}
names(mydata)[3]<-"Passed1"      #Changes the name of the third column
{% endhighlight %}



{% highlight r %}
names(mydata)       #Lists the names of the data frame
{% endhighlight %}



{% highlight text %}
## [1] "ID"      "Color"   "Passed1"
{% endhighlight %}

In order to access and use a single column just like we would a vector, we use the "$".  For example, to access the *Color* column, we use


{% highlight r %}
mydata$Color
{% endhighlight %}



{% highlight text %}
## [1] red   white red   <NA> 
## Levels: red white
{% endhighlight %}

Note that when you bring data into R with *write.csv*, it automatically comes in as a data frame.  We can check this by looking at the *class* for the *birth* data.  


{% highlight r %}
class(births)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'births' not found
{% endhighlight %}


Subset Data
---

We will now return to the birth data and learn how to subset data.  Remember that the *birth* data had 14 variables.  Let's pretend that we really don't need all of the variables, but are really only concerned with the Month of Birth, Day of Week, Age of mother, Sex, Apgar Score and Estimated Gestation.  We can use one of the following two commands to subset our data:


{% highlight r %}
births.sub1<-births[,c(2,3,4,7,8,11)]
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'births' not found
{% endhighlight %}
Note in this example that the comma separates numbers relating to rows and numbers relating to columns.  Since the numeric vector is on the right side of the comma, it is referring to columns.  As we will see in a little bit, it it was on the left side of the comma, it would refer to the row numbers.  

Another way to do this is


{% highlight r %}
myvars <- names(births) %in% c("DOB_MM","DOB_WK","MAGER","SEX","APGAR5","ESTGEST")
{% endhighlight %}



{% highlight text %}
## Error in match(x, table, nomatch = 0L): object 'births' not found
{% endhighlight %}



{% highlight r %}
births.sub2 <- births[,myvars]
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'births' not found
{% endhighlight %}
Note that if we produce the top 5 rows of this data set, it is paired down:


{% highlight r %}
head(births.sub1)
{% endhighlight %}



{% highlight text %}
## Error in head(births.sub1): object 'births.sub1' not found
{% endhighlight %}

Now that we've learned to subset the column data, let's subset row data.  The first way is to use numbers, just like we did with columns.  Remember that the numeric vector on the left side of the comma refers to row numbers.  So in the following, we have selected the first 100 rows of data:


{% highlight r %}
births.sub3<-births[1:100,]
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'births' not found
{% endhighlight %}
Now lets use *births.sub1* and select only those children that are female.  To do this we use the following code:


{% highlight r %}
births.F<-subset(births.sub1,SEX=="F")
{% endhighlight %}



{% highlight text %}
## Error in subset(births.sub1, SEX == "F"): object 'births.sub1' not found
{% endhighlight %}
You can look up into your *Global Environment* and see that there are 208,653 observations in this subset (meaning there were 208,653 females in this random sample)

Note that you can use the AND/OR functions (&/|).  Let's say we want all male children born to women who are 18 years or younger.  We can do this with this command:


{% highlight r %}
births.M18<-subset(births.sub1,SEX=="F" & MAGER<=18)
{% endhighlight %}



{% highlight text %}
## Error in subset(births.sub1, SEX == "F" & MAGER <= 18): object 'births.sub1' not found
{% endhighlight %}

Once again looking at our *Global Environment* will show us that there were 13,118 records in this subset.  




##Table Data
---

Another common way to store information is in a table. Here we look at how to define both one way and two way tables. 

The first example is for a one way table. One way tables are not the most interesting example, but it is a good place to start. One way to create a table is using the *table* command. The arguments it takes is a vector of factors, and it calculates the frequency that each factor occurs. Here is an example of how to create a one way table:


{% highlight r %}
a <- factor(c("A","A","B","A","B","B","C","A","C"))
table(a)
{% endhighlight %}



{% highlight text %}
## a
## A B C 
## 4 3 2
{% endhighlight %}
A table is usually the input command for a barplot


{% highlight r %}
barplot(table(a))
{% endhighlight %}

![plot of chunk unnamed-chunk-37](https://dmbeskow.github.io/images/2016-07-16-IntroToRLesson1/unnamed-chunk-37-1.png)

or a pieplot



{% highlight r %}
pie(table(a))
{% endhighlight %}

![plot of chunk unnamed-chunk-38](https://dmbeskow.github.io/images/2016-07-16-IntroToRLesson1/unnamed-chunk-38-1.png)

We will talk more about plots and graphs next lesson.  

If you want to add rows to your table just add another vector to the argument of the table command.   In the table below we see the incidence of two questions in a survey:


{% highlight r %}
a <- c("Sometimes","Sometimes","Never","Always","Always","Sometimes","Sometimes","Never")
b <- c("Maybe","Maybe","Yes","Maybe","Maybe","No","Yes","No")
table(a,b)
{% endhighlight %}



{% highlight text %}
##            b
## a           Maybe No Yes
##   Always        2  0   0
##   Never         0  1   1
##   Sometimes     2  1   1
{% endhighlight %}

The table command allows us to do a very quick calculation, and we can immediately see that two people who said *Maybe* to the first question also said *sometimes*?? to the second question.

A two way table is also the basic input of a stacked barplot:


{% highlight r %}
barplot(table(a,b))
{% endhighlight %}

![plot of chunk unnamed-chunk-40](https://dmbeskow.github.io/images/2016-07-16-IntroToRLesson1/unnamed-chunk-40-1.png)

Note that the table command is very useful in exploring data.  Let's use it to explore the birth data.  If we wanted to quickly see the number of males and females in the data set, we could use the command


{% highlight r %}
table(births$SEX)
{% endhighlight %}



{% highlight text %}
## Error in table(births$SEX): object 'births' not found
{% endhighlight %}
If we wanted to see the education options and numbers for the data set, we could use the following:


{% highlight r %}
table(births$DMEDUC)
{% endhighlight %}



{% highlight text %}
## Error in table(births$DMEDUC): object 'births' not found
{% endhighlight %}
This would quickly show us that close to half of our data has NULL as the value for education.














