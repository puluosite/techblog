# TMUX
keep track of my tech issues and how to resolve them

1. setup oh my zsh on mac
2. install iterm2 on mac
3. set the tmux
  - #bashrc tmux setting
      
      alias tmux="TERM=screen-256color-bce tmux"
  - #tmux.config
      
      set -g default-terminal "screen-256color"
4. in the iterm2, ssh to remote machine with -X, 
  - tmux new-session -d -s session_name
  - tmux -CC a

5. Sometimes need to mount remote disk locally
  - sudo sshfs -o follow_symlinks -o allow_other user@remote-host:/home/user ./current_dir


