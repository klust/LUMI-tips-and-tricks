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
