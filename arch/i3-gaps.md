# i3-Gaps Configuration  

We log into our i3 session for the first time and go through the wizard to create our first i3 config. This will be created under ~/.config/i3/config.  

First thing we change is our terminal emulator. By default i3 expects urxvt, which we did not install. We installed tilix earlier and that is what we will be using.

> sudo pacaman -S tilix  

Once tilix is installed, next would be setting it in our i3 config file.  

> vim .config/i3/config  

We will be modifying the line with the comment \# start a terminal. We change it from 'bindsym $mod+Return exec i3-sensible-terminal' to 'bindsym $mod+Return exec tilix'.  

Once this is modified we save it and reload the i3 settings by pressing 'Super + Shift + r'.  

Now that we have a terminal, we can go ahead and start making changes to the config file. First I want to modify the keyboard shortcuts. I have been using Regolith Linux for quite some time now and have gottent used to their shortcuts. So I will be basing my shortcuts on that. I have uploaded the default i3 config for Regolith [here][1].

Now that we have navigation set up, there are a few more applications we need to install to get a fully functional desktop. In particular I will be using the following:  
- Rofi - this wil be my launcher. I like this more than dmenu.
- Polybar - this will replace the i3-status bar.
- Picom - the compositor.  

We will tackle this one at a time. First let is focus on Polybar. The polybar is available in the AUR. So before anything else I will be installing 'yay' to make AUR management easier. So we first need to clone the 'yay' repo.  

> git clone https://aur.archlinux.org/yay.git

Next we will build the package. Once you have cloned the repository change into the directory and run the following:  

> cd yay
> makepkg -si

After yay has been installed, we can proceed to install polybar.  

> yay -S polybar  

Once installed, we will need to create a config file. There is a sample config file found in/usr/share/doc/polybar/config. You can copy this to your .config/polbar directory and use it.  

I have modified the default config by removing everything that refers to bspwm. I also used the following Polybar modules:  
- modiles-left:
  - i3
  - xwindow
- module-center
  - date
- module-right
  - filesystem
  - alsa
  - xkeyboard
  - memory
  - cpu
  - wlan
  - battery
  - temperature
  - powermenu  

Now that we have Polybar set up, we will install and configure Rofi.  

> sudo paman -S rofi  



[1]: ./files/regolith_i3_default_config
