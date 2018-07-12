# Configuring Our Prompt
# ======================

  # if you install git via homebrew, or install the bash autocompletion via homebrew, you get __git_ps1 which you can use in the PS1
  # to display the git branch.  it's supposedly a bit faster and cleaner than manually parsing through sed. i dont' know if you care
  # enough to change it

  # This function is called in your prompt to output your active git branch.
  # function parse_git_branch {
  #   git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
  # }

#GARETT CHANGES
# Lets define some pretty colors
BLACK='\033[30m'
RED='\033[31m'
GREEN='\033[32m'
YELLOW='\033[33m'
BLUE='\033[34m'
MAGENTA='\033[35m'
CYAN='\033[36m'
WHITE='\033[37m'

BGBLACK='\033[40m'
BGRED='\033[41m'
BGGREEN='\033[42m'
BGYELLOW='\033[43m'
BGBLUE='\033[44m'
BGMAGENTA='\033[45m'
BGCYAN='\033[46m'
BGWHITE='\033[47m'

BRIGHT='\033[01m'
NORMAL='\033[00m'

ITALIC='\033[03m'
UNDERSCORE='\033[04m'  # only works in xterms
BLINK='\033[05m'     # doesn't work in xterms
REVERSE='\033[07m'
INVISIBLE='\033[08m'
# \033[x;yH Moves cursor to x,y
# \033[yA Moves cursor up y lines
# \033[yB Moves cursor down y lines
# \033[xC moves cursor right x spaces
# \033[xD moves cursor left x spaces
CLEAR='\033[2J'

###
# ls colors
###
# The colors defined below should map as:
# directories: bright white over blue
# symlinks: bright cyan
# sockets: bright magenta
# pipes: bright brown (yellow)
# executables: bright green
# block specials: bright brown (yellow)
# char specials: bright brown (yellow)
# BSD ONLY: SETUID: bright white over red
# BSD ONLY: SETGID: bright white over brown (orange)
# BSD ONLY: "tmp" dirs (world writeable + sticky): black over grey
# BSD ONLY: world writeable dirs: red over grey

# Set the colors used by ls.  Useful for dark displays like putty.

LS_COLORS='no=00:fi=00:di=37;44:ln=01;36:pi=40;33:so=01;35:bd=40;33;01:cd=40;33;01:or=01;05;37;41:mi=01;05;37;41:ex=01;32:*.cmd=01;32:*.exe=01;32:*.com=01;32:*.btm=01;32:*.bat=01;32:*.sh=01;32:*.csh=01;32:*.tar=01;31:*.tgz=01;31:*.arj=01;31:*.taz=01;31:*.lzh=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.gz=01;31:*.bz2=01;31:*.bz=01;31:*.tz=01;31:*.rpm=01;31:*.cpio=01;31:*.jpg=01;35:*.gif=01;35:*.bmp=01;35:*.xbm=01;35:*.xpm=01;35:*.png=01;35:*.tif=01;35:';

# For BSD ls
LSCOLORS="HeGxFxDxCxDxDxHBHdehBh"

export LS_COLORS LSCOLORS

# Create the shell prompt ($PS1)
###

# Color used by all hosts on this network
NETCOLOR=$BRIGHT$WHITE
[ -r "$HOME/.network" ] && eval NETCOLOR=\$NET_`cat "$HOME/.network"`
[ -r "$HOME/.network" ] && eval NETDESC=\$DESC_`cat "$HOME/.network"`

# Find out which terminal device we are on
TERMDEV=`tty | cut -c6-`
RVMSTRING=\$\("rvm current 2>/dev/null"\)
RBENVSTRING=\$\("rbenv version 2>/dev/null |awk '{print \$1}'"\)
GITBRANCH=\$\("git status -bs 2>/dev/null | head -n 1 | cut -d' ' -f2-"\)

# Source global definitions
if [ -f /etc/bashrc ]; then
  . /etc/bashrc
fi

# Solaris version of Linux compatible id
if [ -x /usr/xpg4/bin/id ]; then
  ID="/usr/xpg4/bin/id"
  alias id=$ID
else
  ID="id"
fi

# Prepare the titlebar string if we happen to be on an xterm (or a derivative).
case $TERM in
  xterm*|screen|cygwin)
    TITLEBAR='\[\033]0;\u@\h:\w\007\]'

    # If we're in an xterm, we might be in Konsole or iTerm
    # Renames the Konsole/iTerm tab to the current hostname
    # followed by '#' if root
    TABNAME="\[\033]30;\h\$([ 0 == \$EUID ] && echo '#')\007\]"

    # Set the screen window title if we are in screen
    if [ "$TERM" == "screen" ]; then
      tmp="${USER}@$(echo ${HOSTNAME}|cut -d. -f1)"
      tmp="${tmp}\$([ 0 == \$EUID ] && echo '#')"
      TABNAME="${TABNAME}"'\[\033k'"${tmp}"'\033\\\]'
      unset tmp
    fi
                # Additionally rename the screen window, if applicable

    # Colorizes the Konsole tab to red EUID=0
    TABCOLOR="\[\$([ 0 == \$EUID ] && printf '\033[28;16711680t' || printf '\033[28;0t')\]"
    ;;
  *)
    TITLEBAR=''
    TABNAME=''
    TABCOLOR=''
    ;;
esac

# Prints "[Last command returned error X]" where X is the return code of the
# last executed program when not 0
PRINTErrCode="\$(returnval=\$?
  if [ \$returnval -ne 0 ]; then
    echo \"\\n\[${BRIGHT}${WHITE}[${RED}\]Last command returned error \$returnval\[${WHITE}\]]\"
  fi)"

# Prints "host: /path/to/cwd (terminal device)" properly colorized for the
# current network. "user" is printed as red if EUID=0
TOPLINE="\[${NORMAL}\]\n\$([ 0 == \$EUID ] && echo \[${BRIGHT}${RED}\] || echo \[${NETCOLOR}\])\u\[${NORMAL}${NETCOLOR}\]: \w\[${NORMAL}\] (${NORMAL}${RED}${RVMSTRING:-null}${RBENVSTRING:-null}${NORMAL}) ${BRIGHT}${BLUE}${GITBRANCH}${NORMAL}\n"

# Prints "[date time]$ " substituting the current date and time.  "$" will print
# as a red "#" when EUID=0
BOTTOMLINE="[\d \t]\$([ 0 == \$EUID ] && echo \[${BRIGHT}${RED}\] || echo \[${NETCOLOR}\])\\\$\[${NORMAL}\] "

# Colorize the prompt and set the xterm titlebar too
PS1="${PRINTErrCode}${TABCOLOR}${TABNAME}${TITLEBAR}${TOPLINE}${BOTTOMLINE}"

# Attempt to locate GNU versions of common utilities
# Do this first, before any of the below utilities are checked for GNU-ness
[ `type -P gls` ] && LS=gls || LS=ls
[ `type -P ggrep` ] && GREP=ggrep || GREP=grep
[ `type -P gmake` ] && alias make=gmake
[ `type -P gtar` ] && alias tar=gtar
[ `type -P gmv` ] && alias mv='gmv -iv'
[ `type -P gcp` ] && alias cp='gcp -iv'
[ `type -P grm` ] && alias rm='grm -iv'

        #FLATIRON STYLING
  # This function builds your prompt. It is called below
  # function prompt {
  #   # Define the prompt character
  #   local   CHAR="♥"

  #   # Define some local colors
  #   local   RED="\[\e[0;31m\]"
  #   local   BLUE="\[\e[0;34m\]"
  #   local   GREEN="\[\e[0;32m\]"
  #   local   GRAY_TEXT_BLUE_BACKGROUND="\[\e[37;44;1m\]"

  #   # Define a variable to reset the text color
  #   local   RESET="\[\e[0m\]"

  #   # ♥ ☆ - Keeping some cool ASCII Characters for reference

  #   # Here is where we actually export the PS1 Variable which stores the text for your prompt
  #   export PS1="\[\e]2;\u@\h\a[$GRAY_TEXT_BLUE_BACKGROUND\t$RESET]$RED\$(parse_git_branch) $GREEN\W\n$BLUE//$RED $CHAR $RESET"
  #     PS2='> '
  #     PS4='+ '
  #   }

  # # Finally call the function and our prompt is all pretty
  # prompt

  # For more prompt coolness, check out Halloween Bash:
  # http://xta.github.io/HalloweenBash/

  # If you break your prompt, just delete the last thing you did.
  # And that's why it's good to keep your dotfiles in git too.

  # A handy function to open your bash profile from any directory
  function bp {
    $EDITOR ~/.bash_profile
  }

 # A function to git clone and open that repository
  function gcd {
  REPO=$1
  CLONEPATH=$2

  if [ -z $CLONEPATH ]; then
      CLONEPATH=${$(basename $1)/.git/}
  fi

  git clone $REPO $CLONEPATH
  cd $CLONEPATH
}

# Environment Variables
# =====================
  # Library Paths
  # These variables tell your shell where they can find certain
  # required libraries so other programs can reliably call the variable name
  # instead of a hardcoded path.

    # NODE_PATH
    # Node Path from Homebrew I believe
    export NODE_PATH="/usr/local/lib/node_modules:$NODE_PATH"

    # Those NODE & Python Paths won't break anything even if you
    # don't have NODE or Python installed. Eventually you will and
    # then you don't have to update your bash_profile

  # Configurations

    # GIT_MERGE_AUTO_EDIT
    # This variable configures git to not require a message when you merge.
    export GIT_MERGE_AUTOEDIT='no'

    # Editors
    # Tells your shell that when a program requires various editors, use sublime.
    # The -w flag tells your shell to wait until sublime exits
    export VISUAL="atom"
    export SVN_EDITOR="atom"
    export GIT_EDITOR="atom"
    export EDITOR="atom"

    # Version
    # What version of the Flatiron School bash profile this is
    export FLATIRON_VERSION='1.1.1'
  # Paths

    # The USR_PATHS variable will just store all relevant /usr paths for easier usage
    # Each path is seperate via a : and we always use absolute paths.

    # A bit about the /usr directory
    # The /usr directory is a convention from linux that creates a common place to put
    # files and executables that the entire system needs access too. It tries to be user
    # independent, so whichever user is logged in should have permissions to the /usr directory.
    # We call that /usr/local. Within /usr/local, there is a bin directory for actually
    # storing the binaries (programs) that our system would want.
    # Also, Homebrew adopts this convetion so things installed via Homebrew
    # get symlinked into /usr/local
    export USR_PATHS="/usr/local:/usr/local/bin:/usr/local/sbin:/usr/bin"

    # Hint: You can interpolate a variable into a string by using the $VARIABLE notation as below.

    # We build our final PATH by combining the variables defined above
    # along with any previous values in the PATH variable.

    # Our PATH variable is special and very important. Whenever we type a command into our shell,
    # it will try to find that command within a directory that is defined in our PATH.
    # Read http://blog.seldomatt.com/blog/2012/10/08/bash-and-the-one-true-path/ for more on that.
    export PATH="$USR_PATHS:$PATH"

    # If you go into your shell and type: echo $PATH you will see the output of your current path.
    # For example, mine is:
    # /Users/avi/.rvm/gems/ruby-1.9.3-p392/bin:/Users/avi/.rvm/gems/ruby-1.9.3-p392@global/bin:/Users/avi/.rvm/rubies/ruby-1.9.3-p392/bin:/Users/avi/.rvm/bin:/usr/local:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/local/mysql/bin:/usr/local/share/python:/bin:/usr/sbin:/sbin:

# Helpful Functions
# =====================

# A function to CD into the desktop from anywhere
# so you just type desktop.
# HINT: It uses the built in USER variable to know your OS X username

# USE: desktop
#      desktop subfolder
function desktop {
  cd /Users/$USER/Desktop/$@
}

# A function to easily grep for a matching process
# USE: psg postgres
function psg {
  FIRST=`echo $1 | sed -e 's/^\(.\).*/\1/'`
  REST=`echo $1 | sed -e 's/^.\(.*\)/\1/'`
  ps aux | grep "[$FIRST]$REST"
}

# A function to extract correctly any archive based on extension
# USE: extract imazip.zip
#      extract imatar.tar
function extract () {
    if [ -f $1 ] ; then
        case $1 in
            *.tar.bz2)  tar xjf $1      ;;
            *.tar.gz)   tar xzf $1      ;;
            *.bz2)      bunzip2 $1      ;;
            *.rar)      rar x $1        ;;
            *.gz)       gunzip $1       ;;
            *.tar)      tar xf $1       ;;
            *.tbz2)     tar xjf $1      ;;
            *.tgz)      tar xzf $1      ;;
            *.zip)      unzip $1        ;;
            *.Z)        uncompress $1   ;;
            *)          echo "'$1' cannot be extracted via extract()" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}

# Aliases
# =====================
  # LS
  alias ls='ls -G'
  alias l='ls -laG'
  alias co='checkout'
  alias be='bundle exec'
  alias subl='sublime'

# Case-Insensitive Auto Completion
  bind "set completion-ignore-case on"

# Postgres
export PATH=/Applications/Postgres.app/Contents/Versions/latest/bin:$PATH

# Final Configurations and Plugins
# =====================
  # Git Bash Completion
  # Will activate bash git completion if installed
  # via homebrew
  if [ -f `brew --prefix`/etc/bash_completion ]; then
    . `brew --prefix`/etc/bash_completion
  fi

  # RVM
  # Mandatory loading of RVM into the shell
  # This must be the last line of your bash_profile always
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

#Default Editor

export EDITOR='subl -n -w'

[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*
