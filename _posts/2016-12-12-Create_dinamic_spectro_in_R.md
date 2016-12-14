---
layout: post
title: "Creating dynamic spectrograms"
date: 12-12-2016
categories: rblogging
tags: dynamic spectro
---

This code creates a video with the spectrogram that scrolls from right to left. The spectrogram is synchronized with the audio. This is done by creating single image files for each of the movie frames and then putting them together in .mp4 video format. You will need the ffmpeg UNIX application to be able to run the code.

``` r
# load packages
require("seewave")
require("tuneR")

# download the audio file here (or use your own)
download.file(url = "http://marceloarayasalas.weebly.com/uploads/2/5/5/2/25524573/0.sur.2014.7.3.8.31.wav",
    destfile = "example.wav")

# read wav file
wav1 <- readWave("example.wav", from = 0, to = 19, units = "seconds")

# determine the spectrogram width (in seconds)
tlimsize <- 1.5


# frames per second
fps <- 50

# set a margin of silence to add at the start and end of wav (so the sound starts playing at 0)
marg <- tlimsize / 2
wav <-pastew(wave2 = silence(duration = marg, samp.rate = wav1@samp.rate, xunit = "time"), 
  wave1 = wav1, f = wav@samp.rate, output = "Wave")
wav <-pastew(wave1 = silence(duration = marg, samp.rate = wav@samp.rate, xunit = "time"), 
  wave2 = wav, f = wav@samp.rate, output = "Wave")


#start graphic device to create image files
tiff("fee%04d.tiff",res = 120, width = 1100, height = 700)

x <- 0

#loop to create image files 
repeat{

  tlim <- c(x, x + tlimsize)

  spectro(wave = wav, f = wav@samp.rate, wl = 300, ovlp = 90, flim = c(2, 10.5), tlim = tlim, scale = F, 
  grid = F, palette = gray.colors, norm = F, dBref = 2*10e-5, osc = T, colgrid="white", colwave="chocolate2", 
  colaxis="white", collab="white", colbg="black")
  
  abline(v = tlim[1]+marg, lty = 2, col = "skyblue", lwd = 2)
  
  x <- x + 1/fps
  
  # stop when the end is reached
  if(x >= (length(wav@left)/wav@samp.rate) - tlimsize) break
  
  }

dev.off()

# This is run in UNIX using the system function (need ffmpeg installed)

#Make video
system("ffmpeg -framerate 50 -i fee%04d.tiff -c:v libx264 -profile:v high -crf 2 -pix_fmt yuv420p 
spectro_movie.mp4")

# save audio file
savewav(wave = wav1,filename =  "audio1.wav")

#Add audio
system("ffmpeg -i spectro_movie.mp4 -i audio1.wav -vcodec libx264 -acodec libmp3lame 
-shortest spectro_movie_audio.mp4")
```


At the end you should get something like this:
<br>
<br>
[![Alt text](https://img.youtube.com/vi/McAQaIXeuUQ/0.jpg)](https://www.youtube.com/watch?v=McAQaIXeuUQ)
<iframe  title="YouTube video player" width="1000" height="585" src="https://youtu.be/McAQaIXeuUQ" frameborder="0"></iframe>