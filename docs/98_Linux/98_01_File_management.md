# File management tips and tricks

-   How to quickly check the number of files?

    -   Solution with `rsync`:

        ```
        rsync --stats --dry-run
        ```

    -   Solution with `lfs find` (but will count directories also):
  
        ```
        lfs find . | wc -l
        ```

-   Erasing a lot of files at once (Pawsey tip):
    ["Deleting Large Number ]of Files onm Lustre Filesystems](https://pawsey.atlassian.net/wiki/spaces/US/pages/51925900/Deleting+Large+Numbers+of+Files+on+Lustre+Filesystems).

    Basically, run

    ```
    find -P ./processor0 -type f -print0 -o -type l -print0 | xargs -0 munlink
    find -P ./processor0 -type d -empty -delete
    ```

    -   `-P`: This option restricts the search within the indicated directory tree and forces NO dereference of symbolic links. This warranties that the find command will not look for files within the links.

    -   `./processor0`: This argument is the directory on which the search (and deletion) will be performed.

    -   `-type f -o -type l`: These options indicate that the find command will search for anything that is a file (-type f) or (-o) a soft link (-type l, this is the lower letter l). As indicated with the -P option above, the links are not followed, so only the links will be removed but the object to where they linked are not.

    -   `-print0`: This option indicates the format of the result of the "find" command. This particular format is able to catch strange file names, and ensures that they are readable for the following command  (xargs) which has been concatenated with the pipe. Note that two -print0 indications are needed, one per "side" of the "or" option indicated above.

    -   The pipe command ( represented by a single pipe line: |)
        This command concatenates two commands. This makes the output of the previous command (find) to serve as input to the following command (xargs).

    -   `xargs -0`: xargs will then convert the received list of files, line by line, into an argument 
        for whatever command is specified to it (munlink in this case). The -0 flag is related to the 
        format of the listed files; if you use -print0 in the find command you must use -0 in the xargs command.

    -   `munlink`: This command deletes each file and soft link in the list without overloading the metadata server. In this case, the list is the one received by xargs.

    -   The second find command will search the directory processor0 and all subdirectories for any empty directories (-type d -empty) and delete them. 

