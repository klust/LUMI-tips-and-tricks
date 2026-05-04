# Getting job notifications on your phone

## Procedure one: With pushover

[Pushover](https://pushover.net/) is a simple-to-use notification service that is almost free
for individual use, basically only requiring the purchase of their app.
It also seems to be relatively safe.

Setting up:

-   Create an account with them, which will give you a user key.

    It is dangerous to put that user key in a script, especially if that script is also stored
    in a place where others can read it. 

    For our example, the key is stored in the file `~/.pushover_user_key` which has been made 
    only user-readable.

-   On their website, in your profile page that you get after logging in, create an application
    which will give you an API token. 

    That one should also be stored safely. In our example, it is stored in the file
    `~/.pushover_LUMI_app`.

-   The following bash function can then be used to send a message:

    ```bash
    function pushover { # Send a message to the LUMI app in pushover.
   
       curl -s \
            --form-string "token=$(cat ~/.pushover_LUMI_app)" \
            --form-string "user=$(cat ~/.pushover_user_key)" \
            --form-string "message=$1" \
            https://api.pushover.net/1/messages.json >& /dev/null
   
    }
    declare -fx pushover
    ```

-   And in your job script, you can simply put at the start:

    ```bash
    pushover "Job $SLURM_JOBID on LUMI has started."
    ```


## Procedure two: With Telegram

Telegram offers an entirely free method, but it does seem a bit more risky as it involves
starting a bot that others may also find and use, though with some care they cannot easily 
use it to send messages to you.

-   Create a Telegram bot: Open a chat with "@BotFather" with the message "/newbot" and
    answer the questions.

    You will get a bot token which needs to be stored safely. For our example code, it is 
    stored in `~/.telegrambot` with read rights only for the owner.

-   Start a chat with your bot in Telegram. And now we need to figure out the chatID of that
    chat. One way is to use the cURL tool to request information from the bot:

    ```bash
    curl https://api.telegram.org/bot$(cat ~/.telegrambot)/getUpdates
    ```

    or, to get more readable output if the `jq` command is installed,

    ```bash
    curl https://api.telegram.org/bot$(cat ~/.telegrambot)/getUpdates | jq
    ```

    Look for the "chat" object and note its "id". This number again has to be stored 
    safely. For our example script, it was put in the file `~/.telegramchatid`.

-   Now create a bash function to send a message:

    ```bash
    function TelegramMessage { # Send a message to the LUMI job notification bot.

       curl -X POST \
            -d chat_id=$(cat ~/.telegramchatid) \
            -d text="$1" \
            "https://api.telegram.org/bot$(cat ~/.telegrambot)/sendMessage" >& /dev/null

    }
    declare -fx TelegramMessage
    ```

-   And in your job script, you can simply put at the start:

    ```bash
    TelegramMessage "Job $SLURM_JOBID on LUMI has started."
    ```

    to get a message send to Telegram which can then produce a notification.


## Procedure 3: With Slack

It should be possible with Slack also, as I got a user who was asking if this is
allowed.

Steps I took:

-   Created a personal workspace with no other members in there at the moment via
    the Slack web app [app.slack.com](https://app.slack.com)

-   I also created an app for that workspace:

    -   Create a Slack App: Go to the Slack API dashboard and create 
        a new app from scratch.

    -   Enable Webhooks: In the app settings, click Incoming Webhooks and 
        toggle it to "On."

    -   Add Webhook to Workspace: Click "Add New Webhook" at the bottom of the screen.
        When prompted to choose a channel, select your own name (the "Direct Messages" section).

    -   Copy the URL: It will look like 
        [https://hooks.slack.com/services/T.../B.../XXXX](https://hooks.slack.com/services/T.../B.../XXXX).

-   Try now:

    ``` bash
    curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' https://hooks.slack.com/services/T.../B.../XXXX
    ```

    This is enough to send a message. A message sent this way, will appear under the
    "Direct messages" with your name as sender.

-   But we can make this a bit more versatile and use a different technique for sending
    messages:

    -   In the app, under "OAuth and Permissions", scroll down the page to "Scopes" 
        and add the "Bot Token Scope" "chat:write".

    -   Now you have to install or re-install the app in the workspace you created
        which can also be done from the "OAuth and Permissions" screen, a little higher
        up, and in "Channel for webhook", select your app under "Direct Messages".

    -   From the same "OAuth and Permissions" screen, also copy your "Bot User OAth Token"
        (which should start with `xoxp-`).

    -   Now go find you slack member ID: Click on your profile picture in 
        Slack. Select "Profile" in the menu. Then in the right screen with your profile, click "More" 
        (the three vertical dots) and finally "Copy member ID". 

        Copy it to a safe place. It will likely start with U.

-   Try now:

    ``` bash
    curl -X POST -H "Authorization: Bearer xoxb-your-token-here" \
    -H "Content-type: application/json" \
    --data '{"channel":"YOUR_MEMBER_ID", "text":"Testing API message"}' \
    https://slack.com/api/chat.postMessage
    ```

    This message will arrive under "Apps", in the name of your app, so that it does not mix with
    other stuff you may want to do in the channel where you keep messages to yourself, which is kind
    of a personal notebook. 


## RocketChat

Unfortunately, it looks like you cannot send messages to people on the CSC RocketChat
that way. The "@username" trick for the channel does not seem to work.

Procedure:

-   Use the client to get an access token and userid via the "Profile" submenu "Personal Access Tokens".
    Store safely. 

    For the purpose of this example, the token was stored in `~/.RocketChat_token` and the userid in
    `~/.RocketChat_userid`.

-   Sending a message to a channel:

    ``` bash
    curl -H "X-Auth-Token: $(cat ~/.RocketChat_token)" \
         -H "X-User-Id: $(cat ~/.RocketChat_userid)" \
         -H "Content-Type: application/json" \
         -d '{ "channel": "CHANNEL_NAME", "text": "Hello from the command line" }' \
         https://chat.csc.fi/api/v1/chat.postMessage
    ```

-   According to the documentation, you can use "@username" for the channel to send to a user,
    but that doesn't seem to work on LUMI.
