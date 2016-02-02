---
layout: post
title:  "Syncying two waves and low pass filtering a wave file with python"
date:   2015-11-17 14:43:18
categories: python scipy audio wave
---

I looked for this topic on the webz and only came up with this incompleete [stackoverflow question].

So I thought I might as well write a blog post about it while I'm going at it.

#Scipy

The [scipy] package offers some implementations of filters that seem like they would be useful:
Dependencies:

     $ apt-get install liblapack-dev libblas-dev gfortran

     pip install scipy numpy

# Syncing two wave files

I have two files that are very similar to each other but one of them has a slight delay before starting
The length of the delay varies from case to case.

The idea here is going to be that we're going to calculate the cross correlation between
the two signals in the time domain and therefore obtain the offset needed to synchronise them:

    time             |013456789ABCDEF......
    reference signal |12345543212345654321234554321
    processed signal |0000000012345543212345654321234554321
    
## Cross Correlation

### Theory

The cross correlation of the signals might look like:

                     |1234567898765432100000000000000000

Which indicates that if we shift the signal by 9 frames they are the most similar to each other.

Our goal will be to read the two signals, compute the correlation and then to trim the delay of the
second signal so our output and the reference signal are in sync.

### Implementation

Rather than using the standard library `wave` module, `scipy` offers a special `io` function: [scipy.io.wavfile.read]

   scipy.io.wavfile.read(filename, mmap=False) 


[stackoverflow question] :http://stackoverflow.com/questions/17676882/interpreting-a-wav-file-python
[scipy]: http://www.scipy.org/
[scipy.io.wavfile.read]:http://docs.scipy.org/doc/scipy-0.15.1/reference/generated/scipy.io.wavfile.read.html#scipy-io-wavfile-read
