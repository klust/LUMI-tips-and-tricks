# Using screen

1.  Start a Screen session

    Simply type `screen` and press Enter. This will start a new session.

    To create a named session instead, use `screen -S mysession`.

2.  Detach from a session

    Press `Ctrl-A` followed by `D`. The session will continue running in the background.

3.  List active sessions

    Run `screen -ls` 

4.  Re-attach to a session

    1.  Use `screen -r` to attach to resume the last session.

    2.  Use `screen -r mysession` to attach to a named session.

    3.  If you got kicked out of the system, the session may still be attached. Use
        `screen -D -r mysession` to force-detach, then re-attach to the session.
        (No name is needed if there is only one session.)
