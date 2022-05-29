
## Preface
A few years ago when I first got my very own Pi 3B+ and 
booted Raspbian (now called Raspberry Pi OS) off of it.
Besides commenting on the hideous desktop environment, 
one of the few things I first did was to check out the 
system environment.

Unfortunately, I was greeted by something resembling this:
```
pi@raspberrypi:~$ uname -a
Linux raspberrypi 4.1.19-v7+ #858 SMP Tue Mar 15 15:56:00 GMT 2016 armv7l GNU/Linux
```

Hold up... `armv7l`?
What? Why? Did I do something wrong?

No, all the boxes were ticked—I flashed the correct system 
on the SD card and confirmed the CPU was indeed `aarch64`.
And yet, what the hell is this? Am I getting scammed here?

After searching, it turned out Raspbian didn't even bother 
supporting `aarch64`, and they decided to just ship 
`armv7l` software on `aarch64`-capable hardware.
It wasn't until eons later they finally released something 
called "Raspberry Pi OS 64-bit".

In my opinion this should not even be acceptable to begin 
with: it _should_ have been shipped with `aarch64` by 
default. Rage burnt gold in my veins as I demanded my 
money's worth of an `arrch64` system back at whatever cost.

I continued searching and stumbled upon an article written 
by a great developer, who also ported the UEFI firmware, 
with detailed instructions on how to install UEFI and boot 
an `aarch64` Debian install on the Raspberry Pi 3B+.

So naturally down the rabbit hole I went. 
And the initial trials were definitely not happy memories. 
Days of failing to get even a HDMI output at all, 
let alone getting all three parts to work (namely the card 
partitioning, the UEFI firmware and the Linux system).

At the time I had no idea what I was doing; the logs 
contained no useful information so I could only do blind 
trials and guess what went wrong; for every hour of the 
living day to rinse and repeat the tedious process of 
removing the card, blind-debugging it, inserting it back 
in, plugging in power and hoping for dear life that it 
booted past UEFI with HDMI output. 

This week after repeating most of the same frustrating 
process again, clever me finally decided to document this 
process (and I wrote that with utmost sarcasm), best as I 
could with withering wisdom, to hopefully avoid such 
tragedies from happening again.
