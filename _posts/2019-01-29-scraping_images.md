---
layout: post
title:  "Scraping Images to Create Labeled Data for CNN Model"
categories: [python]
tags: [scraping,python]
---


This code is a collection of code that I used for scraping images from a variety of locations (Google images, Flickr, Tumblr, Twitter, and Instagram).

I used these platforms to collect images for training a Convolutional Neural Network model.  

In my case I was building a CNN to detect internet memes, and used the platforms above to get ~20K examples of memes (and ~20K non-meme images).

So let's dive in and walk through how to scrape images on each of these platforms:

### Google

I used the code below in conjunction with Selenium (with Chrome Driver) to scrape Google Images.  I did this with manual scroll since I only had a handful of queries, but it would probably be trivial to automate the scrolling.

First, load the packages and function



```python
import os
import re
import time
import requests
import io
import hashlib
import itertools
import base64
from PIL import Image
from multiprocessing import Pool
from selenium import webdriver

#%%
def persist_image(dir_image_src):
    label_directory = dir_image_src[0]
    image_src = dir_image_src[1]

    size = (200, 200)
    try:
        image_content = requests.get(image_src).content
    except requests.exceptions.InvalidSchema:
        # image is probably base64 encoded
        image_data = re.sub('^data:image/.+;base64,', '', image_src)
        image_content = base64.b64decode(image_data)
    except Exception as e:
        print("could not read", e, image_src)
        return False

    image_file = io.BytesIO(image_content)
    image = Image.open(image_file).convert('RGB')
    with open('graphs/' + hashlib.sha1(image_content).hexdigest() + ".jpg", 'wb')  as h:
        image.save(h, "JPEG", quality=85)
    resized = image.resize(size)
    with open(label_directory + hashlib.sha1(image_content).hexdigest() + ".jpg", 'wb')  as f:
        resized.save(f, "JPEG", quality=85)

    return True
```

Run the query and then scroll down to load all of the pictures


```python
#%%
# The method below is a bit manual (meaning it requires the user to scroll down), but was adequate
# for my needs since I only had a few queries.  It wouldn't be hard to automate the scrolling if you
# have hundreds of queries
query = 'memes'

image_urls = set()

search_url = "https://www.google.com/search?safe=off&site=&tbm=isch&source=hp&q={q}&oq={q}&gs_l=img"

browser = webdriver.Chrome('C:/Users/dmbes/Downloads/chromedriver_win32/chromedriver.exe')

browser.get(search_url.format(q=query))
```

Get all of the URLs:


```python
## Now scroll down to bottom of file

images = browser.find_elements_by_css_selector("img.rg_ic")
for img in images:
    image_urls.add(img.get_attribute('src'))
```

Then download the images.  

Note that images will be saved with their file name as their SHA1 hashtag.  This will also automatically deduplicate your images.  


```python
#%%
query_directory = './img/'

values = [item for item in zip(itertools.cycle([query_directory]), image_urls)]

print("image count", len(image_urls))

for image in values:
    persist_image(image)
```

### Twitter

This code assumes you have a basic Twitter JSON file or directory, and will
get all links and then images share in these tweets.  

This code will skip Tweets that Twitter flags as *sensitive*.  


```python

import json
import io, gzip, os
import urllib.request
import random
from random import shuffle
import codecs
import progressbar

import os, gzip, io, json
files = os.listdir()
shuffle(files)
files = os.listdir('timeline')
files = ['timeline/' + x for x in files]
links = []
for file in files:
    with io.TextIOWrapper(gzip.open(file, 'r')) as infile:
        for line in infile:
            if line != '\n':
                tweet = json.loads(line)
                if 'possibly_sensitive' in tweet.keys():
                    if tweet['possibly_sensitive']:
                        continue
                if 'media' in tweet['entities'].keys():
                    for pic in tweet['entities']['media']:
                        if pic['type'] == 'photo':
                            u = pic['media_url']
                            links.append((u,tweet['id_str']))
print('number of links',len(links))
```

Now we will download these files in parallel using all of the cores on your machine.  Files are saved with the Twitter ID number in their file name.


```python
'''
This will multiprocess the download (much faster)
'''

import multiprocessing
import random

def download_link(link, ID):
    u = link
    Type = u.split('.')
    Type = Type[-1]
    Name = 'img/'+ str(ID) + '_' + str(random.randint(1000,9999))  + '.' + Type
    try:
        urllib.request.urlretrieve(u, Name)
    except:
        print('error')

cores =multiprocessing.cpu_count()
pool = multiprocessing.Pool(processes = cores)
output = pool.starmap(download_link, links)
```

### Scrape Flickr

After getting an API key with Yahoo, I used the code below to scrape Flickr images:

First authenticate and run query



```python
#%%
import flickrapi
import urllib.request
api_key = u'XXXXXXXXXXXXXXXXXXXXXXXXX'
api_secret = u'XXXXXXXXXXXXXXXXXX'

flickr = flickrapi.FlickrAPI(api_key, api_secret)
photos = flickr.photos.search(tags = 'funny meme', format = 'json')
```

Then walk the API results to get URLs


```python
#%%
keyword = 'meme'

photos = flickr.walk(text=keyword,
                     tag_mode='all',
                     tags=keyword,
                     extras='url_c',
                     per_page=100,           # may be you can try different numbers..
                     sort='relevance')

urls = []
for i, photo in enumerate(photos):
    print (i)

    url = photo.get('url_c')
    urls.append(url)

    # get 1000 urls
    if i > 1000:
        break

print (urls)
```

Finally download the pictures


```python
for file in urls:
    if file is not None:
        new_name = file.split('/')[-1]
        urllib.request.urlretrieve(file, new_name)
```

### Tumblr

For Tumblr, I used an R package that I developed in the past.  

If interested, reach out to see if I can share.  

### Instagram

For Instagram pictures, I used the Python package found [here](https://github.com/althonos/InstaLooter).  

My command was:

```
instalooter hashtag meme img -n 1000
```
