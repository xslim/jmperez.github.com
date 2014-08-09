---
layout: post
title: Detecting tempo of a song using brower's Audio API
date: 2014-08-09 8:40:00+02:00
image: /assets/images/posts/bpm.png
description: Article about project for detecting BPM of a track using the Audio API, in combination with Spotify Web API and Echo Nest API.
tags:
  - spotify
  - echo nest
  - bpm
---

This article explains some ideas behind a small project to detect the tempo of a song using the Audio API. I recommend you to have a look at these links before reading the rest of the article: [Demo](http://jmperezperez.com/beats-audio-api/) and [Code on GitHub](https://github.com/JMPerez/beats-audio-api).

## Beat detection using Audio API

A couple of days ago I came across [Beat Detection Using Web Audio](http://tech.beatport.com/2014/web-audio/beat-detection-using-web-audio/), a blog post by **Joe Sullivan** where he explained a simple algorithm to calculate the tempo of a song using the Audio API. After reading it, I was curious about how well the algorithm would work for other tracks.

So I sat down and coded a small project using his algorithms and adding a search box to search for songs, and display the result for them.

The algorithm is supposed to work better with the central fragment of a song. This avoids the parts with lower volume and fewer signals we can take to detect the beats.

## Searching songs and obtaining a preview MP3
The APIs of some music streaming services provide a sample of 30 seconds of every song in their catalog. We can't choose what 30 seconds of a track we want (i.e. what the starting point of the chunk is), but they will usually be a fragment of the most representative part of the song, and this is a good candidate for the algorithm. 

The bitrate of the samples is rather low, around 96 kb/s according to my tests. I have tried to use them previously for my [karaoke project](http://jmperezperez.com/karaoke/), but the result was very noisy. 

Since I'm more than familiar with the [Spotify Web API](https://developer.spotify.com/web-api/), I have chosen it for [searching tracks](https://developer.spotify.com/web-api/search-item/). The search is also quite flexible and normally is enough with the track name (without artist name) to find the song we want.

The Search endpoint returns a list of tracks, and their preview_url property points to an MP3 file that we can plug in straight to an AudioContext to process it.

## Calculating the tempo

I am following the algorithm described (in great detail) by Joe Sullivan. I have tweaked it a bit to:

1. **Dynamically adjust the threshold to identify peaks**: In some cases a threshold of 0.8 was simply too much and the amount of representative peaks returned too low for doing a proper guess. Thus, I lower the threshold until I have a few more peaks for the given sample. 

2. **Round the theoretical tempo to the closest integer**. Otherwise it is rather impossible to get multiple intervals with the same value. At first sight we lose precision, but tempos are usually integers anyway,

To check how well the algorithm works I wanted to know the BPM of the song, calculated by some trustable source. Luckily enough, the [Echo Nest API](http://developer.echonest.com/docs/v4/track.html) provides the BPM value for a song.

### Matching Spotify tracks with Echo Nest songs

The process of matching is simple. Given a track in Spotify, find the same one in Echo Nest.

However, note that any of these music APIs returned multiple results for what we would think is the same track. And they can have different BPM themselves. So you may obtain a BPM calculated using this algorithm that doesn't match the first result from a track searched on the Echo Nest, using the track name and artist name. But it may match the second or third result.

The Echo Nest API works with identifiers from a wide variety of music APIs. This is called the **Rosetta Stone** project. This makes it easier to return the data for THE specific track we are interested in. One of those IDs is the Spotify URI of a track.

Firstly, we try to obtain the `audio_summary` bucket for the song's profile using the Spotify URI. If this fails because there is no match in Echo Nest's API, we perform a search in EN, and obtain the song's profile for the first of the results.

## Rendering the peaks and playing the track

From Joe's post, I really liked the simple graphs showing the peaks of a song as a way of representing what the algorithm was doing. I decided to do the same, and also add a button to play the track along with an indicator that goes through the graph and helps us see what peaks were detected by the algorithm.

## Enhancements?

I'm sure there must be better algorithms to find out the tempo of a song, but this one has proven to be very simple and effective for most of the "danceable" tracks, between 90 and 180 BPM. And I think it's a fun way to use the modern browser's recent APIs.