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

6. Sometimes need to forward local port to the remote port.
  - add local public key to remote host authorized_keys in .ssh 
  - ssh -L 10000(local):localhost:13445(remote) user@remote_machine
  - Now, instead VNC to remote_machine and open localhost:13445, you can go to localhost:10000 on your current machine.


