## some useful tool
1. ncdu: find the disk usage and delete big files
2. tldr: human readable example for command
3. imagemagick/convert: image conversion tool, generating gif
4. ffmpeg: video conversion
5. broot: nagivate folders
6. paste: join lines

## some useful bash scripting
1. $? -> previous command execution result; $0(current_script_name), $#(#args), $$(PID)
2. !! -> execute last command
3. cat <(ls) <(ls ..) #each <(xxx) generates a tmp file that can pass to another program
4. curly brace to extend command: 
    + e.g. cp /path/to/project/{foo,bar,baz}.sh /newpath; 
    + mv *{.py,.sh} folder; 
    + touch {foo,bar}/{a..h}
5. find.
    + find . -name '*.tmp' -exec rm {} \;
    + find . -name '*.png' -exec convert {} {}.jpg \;
    
6. xargs:
    + xargs echo rm # to echo the command before really executing them
7. sed:
    + sed -E 's/^.*cte_to_implement\(\"(.*)\"\);$/\1/' // \1 refers to the 1st () group
    
8. cronjob
    + create a bash wrapper instead running python directly
    ```bash
    #!/bin/csh
    source /home/xxx/.cshrc
    python /home/yyy.py
    ```
