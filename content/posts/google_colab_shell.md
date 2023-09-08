+++
title = 'Launching a Shell/Terminal for Google Colab'
date = 2023-09-05T14:16:51-07:00
draft = false
+++

[Google Colab](https://colab.research.google.com/) is basically a web GUI in front of a full-container on Linux. While you can run shell command one by one, it is not easy to run commands interactively. We can use [tmate](https://tmate.io/) to help get a full shell. 

You can get the shell access if you pay for higher level Google Colab plans. With this tip, you can get that for free. 

### How

Add a code block with following

```
!wget -q https://github.com/tmate-io/tmate/releases/download/2.4.0/tmate-2.4.0-static-linux-i386.tar.xz 
!tar -xvf tmate-2.4.0-static-linux-i386.tar.xz &> /dev/null
!./tmate-2.4.0-static-linux-i386/tmate -S /tmp/tmate.sock new-session -d  &> /dev/null
! sleep 5 # allowing time for the new session to start
!./tmate-2.4.0-static-linux-i386/tmate -S /tmp/tmate.sock display -p 'Connect to this: #{tmate_web} or #{tmate_ssh}'
```

You can either click the url to open the shell in a browser to use your terminal and launch ssh directly.

Full notebook [here](https://colab.research.google.com/drive/1iu1FSVCb2s4uf_7zhqkB1VdwS2lXx4LP?usp=sharing).

### Note

Google might get mad you.