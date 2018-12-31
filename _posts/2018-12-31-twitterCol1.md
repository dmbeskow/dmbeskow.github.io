---
layout: post
title:  "Introducing twitter_col package"
categories: [python]
tags: [twitter,python]
---


I've had an opportunity to work on a variety of open source social media projects.  A number of these have involved Twitter Data.  Having worked extensively with Twitter Data in R, I migrated to Python in 2017.  This migration was done for several reasons.  First, I discovered that if you don't extract the raw JSON data for Twitter (which many R packages don't), then you end up dropping alot of valuable data on the floor.  Additionally, most third party applications require the full Twitter JSON payload.  Once you do decide to collect raw JSON, it makes sense to transition to Python because of the ease of working with JSON data in Python (JSON data easily transitions to a Python dictionary).  You can work with JSON data in R (usually in a list of lists), but (in my opinion) it is not as easy or straightforward.  

Since I currently have to work extensively with Twitter data, I decided to create a Python package tailored for the types of collection and data munging that my current work flows require.  This package, `twitter_col`,  is easy for me to install (from Github) on any machine or cloud environment, allowing easy access to the functions that I need most.  

Having created this package for my own use, I've had a number of people request access and instructions on how to use it.  

This short tutorial is the first of several tutorials on how to use the `twitter_col` package.  This tutorial will focus on installing the package and using two Python command line programs to stream Twitter data.  In addition to demonstrating this specific package, I will also discuss a few lessons learned regarding how to build a Python package.

# Why Build a Python Package

If you ever think you're going to reuse code more than a couple times, you should consider placing it into a package.  Building a package for R or Python is extremely easy.  The process of packaging it usually forces you to improve your code, properly document it for yourself and others, and then store it in a place and way that you and others can reuse it now or six months from now.

I've found that if I don't properly document and package my code, by the time I come back to it 18 months later, it will take almost as long to understand my old code as it did to write it in the first place.  

The meme below sums up why you should consider packaging your code for reuse:

![meme1](https://dmbeskow.github.io/images/2018-12-31-twitterCol1/meme1.png)

Having built a number of packages in R, and now three packages in Python, I've found that it is fairly easy in either language.  Both allow you to write and document functions and/or data, and both can be easily staged and installed from Github or Gitlab.  Additionally, Python allow you to write command line interface tools, which I'll dive into below.

# Building a Python Package

I don't intend to walk through a step by step tutorial of how to build a Python package...there are too many that already exist.  If you want to learn how to build one, then I recommend looking at the tutorial at [python packaging](https://python-packaging.readthedocs.io/en/latest/index.html).  This will start with minimal structure, and then address how to include dependencies as well as add command line interfaces and/or data.  

# Command Line Interfaces (CLI)

Before I launch into using the `stream_content` command line interface, I want to briefly address command line interface (CLI) tools.  The CLI allows the user to easily call functions and adjust parameters from the command line.  

Note that both Python and R allow a user to call a script or module from the command line (`python3 module.py` in Python or `Rscript script.r` in R), but by default you have to open the file to change parameters.  A CLI tool simply allows you to change variables and parameters from the command line.  

Additionally, launching a script from the command line allows you to move the process to the background, allowing for it to run for days, weeks, or months, which may be necessary when streaming data.

I have used CLI's for streaming data (as in this case) as well as training ML algorithms where I want to tinker with parameters or hyperparameters.

In Python there are several ways to load the command line interfaces.  I use the `argparse` package, though I am familiar with other ways using the `sys` package.  I find the `argparse` package easier to document and control certain behaviors.

Note that you do not have to create a Python package to create a CLI...you can create a module that is a CLI, place it in your working directory, and run `python3 myModule.py parameter1 parameter2`.

# Installing `twitter_col`

I've built these CLI's to work in a virtual environment.  While all the rest of the normal functions are available even if you aren't in a virtual environment, the streaming command line interfaces will only work with a virtual environment.  

We will create the virtual environment from the terminal in Mac or Linux or using the Windows Linux Subsystem (WSL) in Windows:

```
virtualenv -p python3 twitter-env
```

Then we activate the environment with the command

```
source twitter-env/bin/activate
```

Once this is done, the packages and command line utilities we load will be contained in this specific environment.

The basic installation command is given below.  This command will install `twitter_col` and its dependencies: `pandas`, `tweepy`, `textblob`, and `progressbar2`.

```
pip3 install --upgrade git+git://github.com/dmbeskow/twitter_col.git
```

If at any time you require help, you can use the help(twitter_col) or help(twitter_col.function_name).  The documentation is present (but probably isn't going to win any awards).  

# Twitter API

Before you can munge, model, and visualize...you have to get data.  With Twitter Data, you can use either the Streaming API (collect data going forward) based on geography or content, or use the REST API (collect data going backwards).  I use both for different purposes (usually initial collection on the Streaming API with follow-up and enrichment collection on the REST API).  Note that if you're collecting data on an event that is in the past, your only option is the REST API.  

In this tutorial I will introduce the Streaming API using two Python command line interfaces that I've packaged in the `twitter_col` package.

Both of the command line interfaces require the user to provide the path to a JSON file with their Twitter credentials.  Having created your Twitter credentials, place them in a json file with the format below:

```json
{
  "consumer_key": "XXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "consumer_secret": "XXXXXXXXXXXXXXXXXXXXXXXXXXXX",
   "access_token": "XXXXXXXXXXXXXXXXXXXXXXXXXXXX",
   "access_secret": "XXXXXXXXXXXXXXXXXXXXXXXXXXXX"
 }

```

# Streaming based on content

In this section I will introduce the `stream_content` command line interface (CLI) that facilitates easy access to the streaming API with content filtering.  This allows you to filter the Streaming API by any token (hashtag, screen name, text, etc).

Let's say I want to stream content during the Worldcup in regards to Germany, France, and Spain.  I could use their country hashtags with the CLI command:


```python
stream_content key.json '#GER,#FRA,#ESP'
```

This CLI tool will create a new file every 20K tweets.  

In this case, the resulting file will be named '#GER_#FRA_#ESP_YYMMDD-hhmmss.json.gz'.  In general I find it is helpful to keep your search terms in the name of the file so you can remember how you obtained the data (remember, on the command line your parameters aren't nicely stored in a file).  

However, there are some times when you have a very messy list of search terms and you don't want them concatenated together to create your file name.  In this case you can invoke the -tag optional parameter to create your own file name prefix:


```python
stream_content key.json '#GER,#FRA,#ESP' -tag worldcup
```

This will create the filename 'worldcup.YYMMDD-hhmmss.json.gz'  

# Streaming based on geographic bounding box

In this section I will introduce the `stream_geo` command line interface (CLI) that facilitates easy access to the streaming API with geographic bounding box filtering.  This allows you to filter the Streaming API by a rectangular bounding box (city, state, country, region).  If you need to find bounding boxes for specific countries, I recommend [country bounding boxes](https://gist.github.com/graydon/11198540).  

Let's say we want to stream data for New York City.  We could do this with the following command


```python
stream_content key.json -74 40 -73 41
```

This CLI tool will create a new file every 20K tweets.  

This will create a file with the filename 'geo_-74.0_40.0_-73.0_41.0.YYMMDD-hhmmss.json.gz'.  In general it is nice to keep the bounding box in the file name for future reference.  If this is cumbersome, you can once again overwite this with the optional -tag flag:


```python
stream_content key.json -74 40 -73 41 -tag nyc
```

which produces a file named 'nyc.YYMMDD-hhmmss.json.gz'

# Streaming in the background

If you are going to stream Twitter data (or any data) for more than a couple hours, I recommend running the command in the background (and making sure your computer/server doesn't go to sleep).  

There are several ways to do this.  Some people use the `screen` command to run shell windows in the background.  I've used this before, but in general prefer the `nohup` command.  

`nohup` will run any command on the command line, and then push this process into the background, returning the terminal to the user.  To see if your command is still running, you just check the running processes.  It will also write out a message/error log to your working directory.  

For example, to run the `stream_geo` command with `nohup`, use the following command:

```bash
nohup stream_content key.json -74 40 -73 41 -tag nyc > nyc_log.out &!
```

This will immediately start the streaming process and return the terminal to the user.  All logs will be written to nyc_log.out.  
To view all running processes, use the command

```bash
ps -x
```

To only print out the Python processes, use the following command

```bash
ps -x | grep 'python'
```

When you want to stop a streaming process, just kill the process

```bash
kill process_ID
```

Note that nohup can be used with any terminal command.  I often use it for streaming data, training ML algorithms, processing large networks/text, and some general data management (zipping/unzipping big data, moving big data, etc).  

## Next tutorial

In my next tutorial I will cover how to use `twitter_col` to simplify Twitter REST API scraping.  To view the `twitter_col` source code, visit https://github.com/dmbeskow/twitter_col
