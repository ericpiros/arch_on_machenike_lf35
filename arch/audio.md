# Audio  

According to the Arch Wiki, ALSA should be available in the stock kernel. I will just have to unmute the audio. To unmute the master volume I will need to use amixer which are included in the alsa-utils package.  

> sudo pacman -S alsa-utils  

Now I can issue the following commands to unmute the volume.  

> amixer sset Master unmute  
> amixer sset Speaker unmute  
> amixer sset Headphone unmute  

Alternatively you can issue the command 'alsamixer' and unmute it through a TUI.  

You can also run a test after unmuting your sound by running 'speaker-test -c <number of channels>'.
