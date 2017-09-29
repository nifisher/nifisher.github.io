---
layout: code
title: "dotfiles: bash promt"
description: "Console prompt color coded based on user and hostname"  
category: project.notes
published: true
featured: false
date: 2017-09-27
---

Pieced together from notes I found around the web, the idea behind this is that each user and host automatically
get a color assigned to them.  In theory it should allow for quick notice if you aren't on the machine/user you think
you are. In reality, I don't tend to pay enough attention to notice, but I still like the script.


Explained in piecemeil because why not?



First, let us make the colors easier to read...  

```
R="\[\e[m\]"                                                                                                              
RESET="\[\e[m\]"                                                                                                          
BOLD="\[\e[1m\]"                                                                                                          
FAINT="\[\e[2m\]"                                                                                                         
UNDERLINE="\[\e[4m\]"                                                                                                     
REVERSE="\[\e[7m\]"                                                                                                       
STRIKEOUT="\[\e[9m\]"                                                                                                     
LO_BLACK="\[\e[30m\]"                                                                                                     
LO_RED="\[\e[31m\]"                                                                                                       
LO_GREEN="\[\e[32m\]"                                                                                                     
LO_YELLOW="\[\e[33m\]"                                                                                                    
LO_BLUE="\[\e[34m\]"                                                                                                      
LO_MAGENTA="\[\e[35m\]"                                                                                                   
LO_CYAN="\[\e[36m\]"                                                                                                      
LO_WHITE="\[\e[37m\]"                                                                                                     
HI_BLACK="\[\e[30;90m\]"                                                                                                  
HI_RED="\[\e[31;91m\]"                                                                                                    
HI_GREEN="\[\e[32;92m\]"                                                                                                  
HI_YELLOW="\[\e[33;93m\]"                                                                                                 
HI_BLUE="\[\e[34;94m\]"                                                                                                   
HI_MAGENTA="\[\e[35;95m\]"                                                                                                
HI_CYAN="\[\e[36;96m\]"                                                                                                   
HI_WHITE="\[\e[37;97m\]"                                                                                                  
BG_BLACK="\[\e[40m\]"                                                                                                     BG_RED="\[\e[41m\]"                                                                                                       
BG_GREEN="\[\e[42m\]"                                                                                                     
BG_YELLOW="\[\e[43m\]"                                                                                                    
BG_BLUE="\[\e[44m\]"                                                                                                      
BG_MAGENTA="\[\e[45m\]"                                                                                                   
BG_CYAN="\[\e[46m\]"                                                                                                      
BG_WHITE="\[\e[47m\]"   
```

That would be 6 colors (in light and dark), 6 background colors, and a few character styles.  Now just take a hash of the username/hostname, divide by 6, and use the remainder to pick a color...  

```
HOSTCOLOR=$HI_BLUE
USER_COLOR=$LO_GREEN

{
local HOSTHASH, COLORCODE
HOSTHASH="$(printf "%d" 0x$(echo $HOSTNAME | cksum | cut -d ' ' -f 1))"
COLORCODE=$(echo "(${HOSTHASH} % 6) + 31" | bc)
HOSTCOLOR="\[\e[0;${COLORCODE}m\]"
} 2> /dev/null || HOSTCOLOR=$HI_BLUE

{
local USERHASH, COLORCODE
USERHASH="$(printf "%d" 0x$(echo $USER | cksum | cut -d ' ' -f 1))"
COLORCODE=$(echo "(${USERHASH} % 6) + 31" | bc)
USERCOLOR="\[\e[0;${COLORCODE}m\]"
} 2> /dev/null || USERCOLOR=$LO_GREEN
```

Bash can use more than 6 colors, and someday I might update this to use more, but this has been fine so far.

After that, it's just a matter of setting PS1...  

```
# clear PROMPT_COMMAND as I'm not using it
export PROMPT_COMMAND=

if [ `whoami` = "root" ]
then
        export PS1="$BG_RED$HI_YELLOW\u${HI_WHITE}@${HI_YELLOW}\h$HI_BLACK:$HI_GREEN\w $HI_WHITE\$$R "
else
        #export PS1="$BG_BLUE$LO_CYAN\u${HI_CYAN}@${LO_CYAN}\h$HI_BLACK:$HI_GREEN\w $HI_WHITE\$$R "
        #export PS1="$BG_BLUE$LO_MAGENTA\u${HI_MAGENTA}@${LO_MAGENTA}\h$HI_BLACK:$HI_GREEN\w $HI_WHITE\$$R "
        #export PS1="$BG_BLUE$LO_GREEN\u${HI_GREEN}@${LO_GREEN}\h$HI_BLACK:$HI_GREEN\w $HI_WHITE\$$R "
	#export PS1="\n$LO_BLUE[$HI_BLUE\u${LO_GREEN}@${HI_GREEN}\w$LO_BLUE]\n$LO_BLUE[$LO_CYAN\h$LO_BLUE]::$R "
	export PS1="\n$LO_WHITE[${USERCOLOR}\u${LO_GREEN}@${HI_GREEN}\w$LO_WHITE]\n$LO_WHITE[${HOSTCOLOR}\h${LO_WHITE} ($SHLVL.\j)]::$R "
fi
 
```

And clean up our valiables before leaving...  

```
unset R RESET BOLD FAINT UNDERLINE REVERSE STRIKEOUT LO_BLACK LO_RED LO_GREEN LO_YELLOW LO_BLUE LO_MAGENTA LO_CY
unset HI_BLACK HI_RED HI_GREEN HI_YELLOW HI_BLUE HI_MAGENTA HI_CYAN HI_WHITE BG_BLACK BG_RED BG_GREEN BG_YELLOW
```

