---
layout: post
title: SSH tunnel protocol
date: '2021-07-29'
categories: Bioinformatics
tags: [DNA]
---


_**Objective:**_ Goal is to set up local and server machines to run an ssh tunnel to enable anvio. 

Protocol follows from anvio blog post [here](https://merenlab.org/2018/03/07/working-with-remote-interative/) and [here](https://merenlab.org/2015/11/28/visualizing-from-a-server/)

Its a bit more complicated then in the blog post since in cologn I need to tunnel into a server machine and then tunnal from there.  

### On Server

*  I created this file on the sever side to tell python how to connect to the web 

```
cat << EOF > $(python -c "import os; print(os.path.join(os.path.dirname(os.path.abspath(os.__file__)), 'webbrowser.py'))")
def open_new(url):
    print("OPEN_ON_LOCAL[" + url + "]")

def register(*args, **kwargs):
    pass

def BackgroundBrowser(*args, **kwargs):
    pass
EOF
```
* Test if it worked: 

```
python -c 'import webbrowser; print(webbrowser.__file__.rstrip("c"))'
```

This should print something like: 
```
/path/to/anvio/env/lib/python3.6/webbrowser.py
```
To test of the module runs right: 
```
python -c "import webbrowser; webbrowser.open_new('https://www.google.com/')"
```

This should print back: 
```
OPEN_ON_LOCAL[https://www.google.com/]
```
-> Which is does successfully, we are on the right track!

* I needed to add a file to the following directory
`~/miniconda3/envs/anvio-7/etc/conda/activate.d`

File name: activate-interactive-ports.sh

`bash
#!/usr/bin/env bash
[[ "$(whoami)" = "sogin" ]] && export ANVIO_PORT=8100
`
Restart anvio and test if it worked: 

`echo $ANVIO_PORT`

-> it worked

### Locally 

* I created an alias to logging into cologn and pasted it into my bash_profile or similar file (in this case .zshrc)

```
alias cologn="ssh -L 8100:localhost: 8100 sogin@your-uni-server | tee /dev/tty | python3 ~/.ssh/run_webbrowser.py"
```
Note: the number after the L must be the same as your port you enntered above in teh file activate-itneractive-ports.sh. You also need to modify the server address and alias name as needed

* I then needed to create the run_webbrowser.py file on my end in the .ssh folder:

~/.ssh/run_webbrowser.py
```
import sys
import webbrowser

for line in sys.stdin:
    if "OPEN_ON_LOCAL[" in line:
        line = line.replace("0.0.0.0", "127.0.0.1")
        webbrowser.open(line.split("OPEN_ON_LOCAL[")[1].split("]")[0])
```
* To log into cologn, just type cologn!
* Cologn requires one more additional step, run the tunnel again to a gc-node

```ssh 8100:localhost:8100 gc-node-1```

Tada! 

note: if you want to log into an interactive session on the server you will need to do something extra that I'm not sure about yet. 

