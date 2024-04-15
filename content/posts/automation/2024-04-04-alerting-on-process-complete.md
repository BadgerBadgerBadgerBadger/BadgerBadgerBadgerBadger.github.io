---
layout: post
title: Alerting on Process Completion
excerpt: "A story of me figuring out how to alert myself when a long-running process has completed."
modified: 2024-04-04T00:00:00-00:00
image: https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/6ce2cd86-7228-4180-bbb6-36ab539bd69a
categories: [automation]
tags: [webhooks, home-assistant, shell-scripting]
comments: true
share: true
---


Most of my technical endeavours have something to do with enabling me to work as little as possible for as much reward as possible. And yet, paradoxically, in a pursuit of doing less I often end up doing more. I have this idea that in the long term this will eventually lead me to doing _much_ less. I'm not sure if that's true, but it's a nice thought and keeps me going. Plus, I learn a lot of fun things along the way.

Whenever I have a long-running process on my home-lab I start it in a [screen session](https://linux.die.net/man/1/screen). I check back on it occasionally to see if it has completed. The same could be true of something I'm running on my local machine, though, in that case, it is in a live terminal session. The latter is usually during the course of my day-job and is somewhat of an easier problem to solve since I am at my machine for the duration of the process.

For my local machine (which is a MacBook), I use the [say](https://ss64.com/mac/say.html) command extensively. It's trivial to add a `say "Done"` at the end of a command chain to aurally inform me of a task's completion. But that wasn't quite spicy enough for me, so I came up with something more interesting.

```sh
➜  ~ which sdd
sdd () {
	if [ $# -eq 0 ]
	then
		title="Done"
		text="Done"
	else
		title=$1
		shift
		text="$@"
	fi
	osascript -e "display notification \"$text\" with title \"$title\"" && afplay ~/Documents/star-wars-b1-battle-droid_kampfdroide-roger-roger-sound.mp3
}
```

Behold the `sdd` function I add to the end of every script and command chain. I can invoke it with a title and an optional message, or invoke it with nothing and let it default to "Done" and "Done". When invoked, I will be shown a [notification](https://developer.apple.com/library/archive/documentation/LanguagesUtilities/Conceptual/MacAutomationScriptingGuide/DisplayNotifications.html) on my screen along with an audible "Roger! Roger!"

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/630f82c6-758c-4b62-8869-863a457c26c4)

> **Disclaimer**\
> I have no idea if this is required. I find copyright laws almost maliciously confusing. Anyway, just to cover my backside:
>
> _**I do not own any rights to the "Roger, Roger" voice line, which is associated with the Star Wars franchise created by George Lucas and owned by Lucasfilm Ltd. and The Walt Disney Company. This phrase is used here under the principles of fair use for educational or informational purposes only.**_

This is all fine and dandy for my local machine, but what about my home-lab? I don't want to keep a live SSH session running all the time, nor do I want to have to check back on it at odd intervals. Nor am I _at_ a terminal all the time!

The solution to that comes from my Home Assistant setup (look out for another post that talks about my Home Assistant and dryer smartification journey). 

I have created an [Automation](https://www.home-assistant.io/docs/automation/) that listens to a webhook and sends a notification to my phone whenever that webhook fires.

```yaml
alias: Notify Sharpie
description: ""
trigger:
  - platform: webhook
    allowed_methods:
      - POST
    local_only: false
    webhook_id: "<webhook-id>"
condition: []
action:
  - service: notify.mobile_app_sharpie
    data_template:
      title: Notification
      message: "{{ trigger.json.message }}"
mode: single
```

> Sharpie is the name of my phone.

Now I can curl from any device anywhere and receive a notification on my phone!

```sh
curl -i -H "Content-Type: application/json" -d "{\"message\": \"Sup?\"}" https://<home-assistant-server>/api/webhook/<wehook-id>
```

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/d4e33a9d-bc6c-499b-92fb-156874c155df)

I can plop this down at the end of any command chain or script and be notified when it completes.

But I figured out all this after the fact. During the fact (is that even a phrase?), I had already started a long-running process that I wanted to monitor and be informed of when it terminated. So there was no way for me to append a notification curl at the end of the chain without stopping the process.

Enter the [ps](https://man7.org/linux/man-pages/man1/ps.1.html) command with the [-p](https://medium.com/@linuxschooltech/what-is-ps-p-command-in-linux-aede7e5f0751) flag. This can be passed a process ID and gives you information about the process.

```sh
➜  ~ echo $$
266386
➜  ~ ps -p 266386
    PID TTY          TIME CMD
 266386 pts/1    00:00:00 zsh
➜  ~ echo $?
0
```

> `$$` is the PID of the current shell process.

Note that the exit code returned by the `ps -p` command is `0`. If we were to try a PID that does not exist, we will get a `1`.

```sh
➜  ~ ps -p 42069
    PID TTY          TIME CMD
➜  ~ echo $?
1
```

Thus, without knowing the exact details or output of using `ps -p` on our target process, we can check the exit code and find out if the process is running. This leads us to this helper function.

```sh
# Check if the process with PID exists
process_exists() {
    ps -p $1 > /dev/null 2>&1
}
```

Checking if our target process is still running, now becomes a matter of a sleepy loop (to avoid spamming the command).

```sh
# Loop until the process no longer exists
while process_exists $PID; do
    sleep 1
done

echo "Process $PID has stopped."

# Send the notification
send_message $MESSAGE
```

The `send_message` function can take advantage of our webhook and notify us when the process completes!

```sh
send_message() {
  curl -i -H "Content-Type: application/json" -d "{\"message\": \"$1\"}" https://<home-assistant-server>/api/webhook/<wehook-id>
}
```

And with that we have a script that can monitor a process and let us know when it completes.

# Wrapping it Up

I hope you enjoyed this brief exploration of figuring out how to monitor a live process. If you were to try something like this yourself, you could, no doubt, switch out the notification part with any number of other services. Perhaps [send yourself an SMS](https://www.twilio.com/en-us/messaging/channels/sms) or blink the lights in your house on and off 21 times.
