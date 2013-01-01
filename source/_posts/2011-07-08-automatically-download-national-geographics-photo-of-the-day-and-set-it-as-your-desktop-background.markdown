---
comments: true
date: 2011-07-08 05:14:04
layout: post
slug: automatically-download-national-geographics-photo-of-the-day-and-set-it-as-your-desktop-background
title: Automatically download National Geographic's Photo of the Day and set it as
  your desktop background
wordpress_id: 152
categories:
- Photography
- Programming
tags:
- national geographic
- photo of the day
- python
- wallpaper
---

National Geographic is an exceptional source of photos and imagery. I wrote a short Python script that's been tested on Linux and Mac OS X to read the [@NatGeo](http://twitter.com/#!/NatGeo) Twitter stream and automatically download their Photo of the Day featured image. Occasionally the photo of the day is provided by a National Geographic reader and these don't provide high-res download links (likely due to licensing and permissions) so the script ignores these.

 If you're running Linux, it will attempt to set the Gnome background wallpaper to the downloaded image, and by default it downloads to ~/Dropbox/Wallpapers. This can be changed by modifying the DOWNLOAD_FOLDER variable. The bonus of it downloading to Dropbox is that I can run the script in a cron job on my Linux desktop and have the wallpapers appear on all my other machines --- especially nice as my laptop displays wallpapers randomly from the Wallpapers folder.

[code lang="python"]
#!/usr/bin/env python

"""
natgeowp.py

Fetches the National Geographic Photo of the Day from the @NatGeo Twitter
stream, downloads it, and tries to set it as the background wallpaper.

Can be scheduled to run in a cron job with crontab.

Martin Foot <martin@mfoot.com>

This is a cleaned up version of my original script after seeing Christian
Stefanescu's NASA image of the day wallpaper script -
http://0chris.com/nasa-image-day-script-python.html
"""

try:
    from urllib.request import urlopen
except ImportError:
    # We're using Python 2.x, not Python 3.x
    from urllib import urlopen

import json
import re
import os
import commands
import platform

# Configurable settings
FEED_URL = 'http://twitter.com/statuses/user_timeline.json?id=NatGeo'
DOWNLOAD_FOLDER = os.path.join(os.getenv('HOME'), 'Dropbox', 'Wallpapers')

def fetch_tweets(feed_url):
    """
    Return JSON object of the Twitter stream.
    """
    # Don't edit below
    twitter = urlopen(feed_url)
    tweets = twitter.read()
    twitter.close()
    tweets = json.loads(tweets)
    return tweets

def filter_tweets(haystack):
    """
    Filter out any tweets that don't contain the words "Photo of the Day"
    """
    needle = 'Photo of the Day'

    found = []

    # Loop through tweets, newest last.
    for tweet in haystack:
        if tweet['text'].startswith(needle):
            # We've found a photo, extract the URL (tweet seems to be
            # automated, we can assume there will be one).
            pic_url = re.search("(http://\S+)", tweet['text'])#.groups()[0]

            if pic_url != None:
                pic_url = pic_url.groups()[0]
                found.append(pic_url)

    return found

def check_downloadable(urls):
    """
    Loop through a list of National Geographic Photo of the Day URLs, and if
    they provide a download link, record it. Some images such as
    reader-submitted images don't provide a link, likely due to licensing
    issues. We won't download these.
    """
    found = []

    for pic_url in urls:
        text = str(urlopen(pic_url).read())
        pic_url = re.search('a href="(\S+)">Download Wallpaper', text)

        # Not all photos have a download link
        if pic_url != None:
            pic_url = pic_url.groups()[0]

            found.append(pic_url)

    return found

def download_photos(urls):
    """
    Download the list of files to the DOWNLOAD_FOLDER directory.
    """
    for url in urls:
        print 'Downloading ' + url
        filename = re.search(".*/(.+)", url).groups()[0]
        path = os.path.join(DOWNLOAD_FOLDER, filename)

        imageFile = open(path, "wb")
        imageFile.write(urlopen(url).read())
        imageFile.close()

    return path

def set_gnome_wallpaper(file_path):
    command = "gconftool-2 --set \
            /desktop/gnome/background/picture_filename \
            --type string '%s'" % file_path
    status, output = commands.getstatusoutput(command)
    return status

if __name__ == '__main__':
    tweets = fetch_tweets(FEED_URL)
    image_urls = filter_tweets(tweets)
    downloadable_urls = check_downloadable(image_urls)

    if len(downloadable_urls) > 0:
        most_recent = download_photos(downloadable_urls)

        osplatform = platform.system()

        # If we're on Linux, try and set the desktop wallpaper with gconftool
        # (TODO: Non-Gnome desktop environments).
        if osplatform == "Linux":
            set_gnome_wallpaper(most_recent)
[/code]
