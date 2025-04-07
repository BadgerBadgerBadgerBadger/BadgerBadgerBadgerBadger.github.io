
People have complicated feelings about AI Coding assistants. I won't wade into that debate. For me, personally, while I enjoy learning about and playing with software, I also enjoy achieving end-goals. For this particular project, [Claude](https://claude.ai/) has been a boon. The reality of being the father to a nearly one-year old is that I no longer get a lot of time for my hobbies. But with Claude, I was able to do in days what might have taken me weeks.

When we first found out we were having our baby boy, I was overjoyed. The terrifying reality of parenthood did not set in until I held my child in my arms for the first time.

Those first few days were the hardest. The first few weeks were all but a blur, and the first few months had us sleep deprived and somewhat cranky. All normal things, I am told by many a parent.

It was not until recently that I have become comfortable enough looking after my child and family that I am able to let my brain wander into its more hobbyish moods. And when it did, it decided to start solving some real world problems, though I leave it up to you to decide if the returns are worth the trouble.

We have been using one of the many baby tracking apps on the market to keep track of when we feed our baby, when he sleeps, how often he poops, and other beautiful bodily functions. It's a simple app in its engineering, but exquisite in its interface and UX.

It maintains a "baby" as an entity for which it records various activities: food, bottle, sleep, etc. Some activities like "bottle" or "diaper" which are points in time have a time field for when they occurred. Activities like "sleep" have a start and end time. Activities can also have groups (such as "fruit" or "vegetables" for the "food" activity), and all activities can have a "notes" field. I and my partner's user accounts are added to the baby as "caregivers." When we update an activity on our phones, it syncs to both devices with detaials on who added the record.

It's a system that has worked well for us and one we've used for months now. Recently, I wanted it to work even better (or I got lazier?).

We have Google nest devices scattered around the house. We use them to play music, play white noise for when baby sleeps, control smart lights, learn about the weather, all the usual smart home stuff. So I asked myself: wouldn't it be nice if it could answer questions like "when did baby last have a bottle?", "how long was baby's last nap?", "how much did baby drink last week?".

And thus started my journey.
# Syncamajiggy

Since the baby tracking app did not have an API I could access, step one was to figure out how to sync that data to a server I controlled. I don't have any mobile dev experience, but with Claude's help I figured out how to get a rooted Android emulator running on my Mac.

```sh
➜  ~ sdkmanager "system-images;android-30;google_apis;arm64-v8a"
```

I had downloaded an APK of the tracker app from https://www.apkmirror.com/ and based on its metadata, Claude informed me this would be a good system image. The command above downloads it to my system.

```sh
➜  ~ avdmanager create avd -n rooted_emulator -k "system-images;android-30;google_apis;arm64-v8a" -d pixel_4
```

This command creates an [Android Virtual Device](https://developer.android.com/studio/run/managing-avds)(AVD) based on a Pixel 4 device definition and the system image I had downloaded earlier. I named it `rooted_emulator`.

Finally...

```sh
➜  ~ emulator -avd rooted_emulator -writable-system -no-snapshot -selinux permissive
```

This command launches the virtual device in an emulator making the system partition writable (`-writable-system`, we'll soon see why that's needed), foregoes saving any state (`-no-snapshort`) and does not block security policy violations (`-selinux permissive`).

When it came to installing the APK into the emulator, I had my first surprise. My last brush with anything Android was back in 2011 when life was _much_ simpler. You got a slightly shady APK from somewhere and installed it on your rooted device and enjoyed some fleeting pleasures not readily available on the Play Store.

These days APKs can come in multiple parts! Apparently Google Play can use [Android App Bundles](https://developer.android.com/guide/app-bundle) to bundle together parts specific to the device its being downloaded to. The split nature of the APK could account for resources for varying screen sizes and densities, languages, and even dynamic features! Very cool stuff.

I installed the APK using...

```sh
➜  ~ adb install-multiple base.apk split_config.*.apk
```

Once installed I could launch the app, log into my account and view my data. Now it's time to figure out how this app is talking to the backend.

I used the free version of [HTTP Toolkit](https://httptoolkit.com/) a brilliant piece of software that automates the process of intercepting HTTP calls. 

![image](https://github.com/user-attachments/assets/f0997774-7c22-4a90-9116-1783c99af646)

It supports many applications and for Android in particular it has a plethora of options.

![image](https://github.com/user-attachments/assets/4533cd61-fccb-4e14-8af2-168b690506d8)

For me, it was as easy as choosing `Android Device via ADB` (which becomes ungrayed when it detects an active device via [`adb`](https://developer.android.com/tools/adb)). The program requires a rooted device with a writable system. This is because in order to successfully decrypt HTTPS traffic it needs to inject an SSL certificate as a [system level certificate](https://medium.com/hackers-secrets/adding-a-certificate-to-android-system-trust-store-ae8ca3519a85).

Let me explain.

Android phones (devices?) support configuring a proxy server. While that allows us to intercept the traffic from the apps, it's not much use since pretty much all traffic these days is encrypted via [SSL certificates](https://www.cloudflare.com/en-gb/learning/ssl/what-is-ssl/).

HTTP Toolkit solves this by injecting its own certificate into the system certificate store of the device. It has to be on the system level since most apps don't support the [user certificate store](https://support.google.com/pixelphone/answer/2844832?hl=en). And for this we need a rooted, writable system.

A rooted emulator with a writable system partition solves this. So with that...

![image](https://github.com/user-attachments/assets/57406026-b424-48b1-b04d-764f7ec85ddd)

... we have ~~inception~~ interception!

It took me a while to figure out which requests were from the baby tracker app. HTTP Toolkit intercepts all traffic from the device and I don't know if there's a way to filter down to a single app.

With trial and error I found the requests relevant to my quest. After some false starts I discovered the tracker app was talking to a [Firebase Cloud Storage](https://firebase.google.com/docs/storage) system. The crucial request is `https://firestore.googleapis.com/google.firestore.v1.Firestore/Listen` which subscribes to updates from collections (which are analogous to SQL tables). This was what kept the app in sync with event updates and what I had to replicate on the server side.

![image](https://github.com/user-attachments/assets/901235aa-6655-4da7-bdc0-cc9463bb1eb1)

Using the [cloud.google.com/go/firestore](https://pkg.go.dev/cloud.google.com/go/firestore) Golang module, and a generous amount of guidance from Claude, I was able to build a simple backend server that keeps listening to the specific collection that records baby's daily events and syncs them to a locally hosted postgres server.

The next step was to hook this up to a voice assistant.
# Dialog what?

Trying to figure out how do this with a Google Nest device was frustrating. Their [DialogFlow](https://dialogflow.cloud.google.com) thing seems to be deprecated (or not well supported?), their Conversational Agents thing does not seem to work with Nest devices, and navigating their myriad of products and technical jargon was giving me a headache.

It's a problem I bashed my skull against for a while. I even considered somehow sending the commands from the Nest to my HomeAssistant instance and using HA's voice tech. But that just felt _convoluted_. I kept having this feeling that this shouldn't be _this_ hard.

And then I remembered why I was having this feeling. I had once built a very simple [Skill](https://www.amazon.com/alexa-skills/b?ie=UTF8&node=13727921011) for an old Amazon Echo device. Although I remembered nothing about the process, I remembered it being relatively painless. I also remembered that the Amazon Echo ecosystem supports a Skills marketplace, something the Google Home ecosystem doesn't seem to. At least not without pain.

It led me to unearthing my old Amazon Echo (1st or 2nd gen, I believe). Building a basic skill that sent a request to my server and replied with a nicely formatted string took less than 2 hours. The ease of use compared to Google's nightmare was so stark that I nearly wept.

After that, there was no looking back.
# Be Intentional

Alexa skills are built around the concept of ["intents"](https://developer.amazon.com/en-US/docs/alexa/custom-skills/create-intents-utterances-and-slots.html) - essentially things users want to accomplish. When we create a skill, we define these intents and provide sample utterances that might trigger them. The model reminded me of pattern matching in a language like Rust or Elixir.

The lifecycle of an Alexa request looks something like this:

1. User says: "Alexa, ask hey baby when was baby's last bottle?".
2. Alexa identifies "hey baby" as my skill's invocation name.
3. It matches "when was baby's last bottle" to my `LastActionIntent` with the slot `actionType` set to "bottle" (more on Slots below).
4. My backend receives this intent and slot data as a JSON payload.
5. My code queries the database and returns a response.
6. Alexa speaks the response to the user.

> Since I am not planning on publishing this Skill to the Alexa marketplace (or whatever it's called, I can't say: "Alexa, when was baby's last bottle?". That's something that can be achieved with [Name-free interactions])(https://developer.amazon.com/en-US/docs/alexa/custom-skills/understand-name-free-interaction-for-custom-skills.html), which are only available for published skills. Name-free interactions trigger Alexa to look for the skill that best matches the utterance and use that to process the request.

Looking at the JSON I've defined in my interaction model (which is just a fancy way of saying "the configuration of my skill"), you can see I've focused on a few key use cases.

For example, one of the most common questions we have is about the last time our baby had a bottle:

```json
{
    "name": "LastActionIntent",
    "slots": [
        {
            "name": "actionType",
            "type": "actionType"
        }
    ],
    "samples": [
        "last {actionType}",
        "when did baby last have {actionType}",
        "when was baby's last {actionType}",
        "when was the last {actionType}"
    ]
}
```

This intent captures various ways we might ask about the last occurrence of an action. The `actionType` slot can be filled with values like "bottle", "diaper", "nap", etc. I've defined these in a custom slot type:

> Slots in Alexa skills function like variables in programming - they're placeholders for dynamic values that users provide when they speak. We can define our own slot types (like `actionType` above) with a specific set of valid values and synonyms, similar to enums in programming. Amazon also provides built-in slot types like `AMAZON.DATE`, `AMAZON.NUMBER`, and `AMAZON.TIME` that handle common data formats and natural language variations without us having to implement them ourselves.

```json
{
    "name": "actionType",
    "values": [
        {
            "name": {
                "value": "bottle",
                "synonyms": [
                    "drink",
                    "formula"
                ]
            }
        }
    ]
}
```

Beyond just knowing when things happened, we also wanted to track quantities like how much formula was consumed:

```json
{
    "name": "LastBottleVolumeIntent",
    "slots": [],
    "samples": [
        "how many milliliters was baby's last bottle",
        "what was the volume of baby's last bottle",
        "how much was baby's last bottle",
        "last bottle volume",
        "how big was the last bottle"
    ]
}
```

And we can get quite specific with stats over time periods:

```json
{
    "name": "BottleStatsIntent",
    "slots": [
        {
            "name": "bottleMetric",
            "type": "bottleMetricType"
        },
        {
            "name": "datePeriod",
            "type": "AMAZON.DATE"
        }
    ],
    "samples": [
        "how {bottleMetric} did baby drink {datePeriod}",
        "how {bottleMetric} did baby have in bottles {datePeriod}",
        "how {bottleMetric} formula did baby have {datePeriod}",
        "how {bottleMetric} did baby drink",
        "how {bottleMetric} from bottles"
    ]
}
```

The `bottleMetricType` slot configuration.

```json
{  
    "name": "bottleMetricType",  
    "values": [  
        {  
            "name": {  
                "value": "bottles",  
                "synonyms": [  
                    "many bottles",  
                    "number of bottles",  
                    "times"  
                ]  
            }  
        },  
        {  
            "name": {  
                "value": "ml",  
                "synonyms": [  
                    "milliliters",  
                    "volume",  
                    "much",  
                    "quantity"  
                ]  
            }  
        },  
        {  
            "name": {  
                "value": "total",  
                "synonyms": [  
                    "all"  
                ]  
            }  
        }  
    ]  
}
```

I found myself being quite thorough with the sample utterances. Unlike traditional UI design where you know exactly what controls are available to users, voice interfaces are much more open-ended. I don't always know how I am going to phrase my question when I get around to asking.

On the backend side, handling these intents is straightforward. I built a small Go server that receives requests from Alexa, maps the intents to database queries, and returns formatted responses. The handler for the `LastActionIntent` looks something like this:

```go
func handleLastActionIntent(ctx context.Context, request alexa.Request) (alexa.Response, error) {
	actionType, err := request.GetSlotValue("actionType")
	if err != nil {
		return nil, errors.Wrap(err, "failed to get action type")
	}

	// Query the database for the last entry of the given action type
	entry, err := db.GetLastEntry(ctx, actionType)
	if err != nil {
		return nil, errors.Wrap(err, "failed to get last entry")
	}

	// Format the response
	timeAgo := humanize.Time(entry.Timestamp)
	text := fmt.Sprintf("Baby's last %s was %s", actionType, timeAgo)
	
	return alexa.NewSimpleResponse(
		fmt.Sprintf("Last %s", actionType),
		text,
	), nil
}
```

When the request comes in, I extract the slot value, query my database, and format a human-friendly response like "Baby's last bottle was 2 hours ago." The Alexa SDK handles converting this text response to speech.

You might have noticed that slot values can have synonyms. In my code I map these to their canonical value before converting them into a query. This is made easy because for any slot synonym the Alexa request includes the canonical value.

Alexa's built-in slot types impressed me. The `AMAZON.DATE` type, for example, handles natural language date expressions like "yesterday" or "last week" without me having to implement date parsing myself. There are similar built-ins for numbers, durations, and other common data types.

Testing the skill was also fairly painless. The Alexa Developer Console has a simulator that lets you type or speak test commands and see exactly how your skill would respond, including the full JSON of the request and response.

![image](https://github.com/user-attachments/assets/c23e6634-1bfd-4510-a14d-1758be16ad32)

# Trust but Verify

Since this backend exposes the data about my dear baby boy, security wasn't optional. Thankfully, Amazon provides mechanisms to verify that requests are genuinely coming from Alexa and not someone particularly interested in how many diapers I'm going through.

Each request sent to the skill's endpoint includes two crucial security elements:

1. A URL pointing to an Amazon certificate.
2. A cryptographic signature of the request body.

The verification process is more complex than I initially expected.

First, we need to validate the certificate URL itself (found in the `SignatureCertChainUrl` header). Amazon requires it to follow a specific format - it must be HTTPS, hosted on Amazon's S3 servers, and follow a particular path structure. This prevents attackers from pointing to their own certificates.

```go
// Simplified example of URL validation
if !strings.HasPrefix(parsedURL.Path, "/echo.api/") {
    return errors.New("certificate URL path does not start with /echo.api/")
}
```

Next comes timestamp validation. Each Alexa request includes a timestamp, and we need to verify it's recent (within 150 seconds). This prevents replay attacks where someone could capture a valid request and resend it later.

The real cryptographic magic happens in the signature verification. We download Amazon's certificate, verify it's valid and specifically issued for Alexa services, and then use it to check the signature against the request body.

For performance reasons, we also cache the certificate. The certificate doesn't change frequently, so there's no need to download it for every request.

Finally, we verify that the skill ID in the request matches the specific skill's ID, ensuring that only that skill can access the backend endpoint.

```go
// Verify the application ID
if requestAppID != s.config.AlexaSkillID {
    return errors.Errorf("Expected skill ID: %s, got: %s", 
                        s.config.AlexaSkillID, requestAppID)
}
```

And with that...

<audio controls preload="auto"><source src="https://all-i-see-is-static.netlify.app/audio/babys-last-nap.mp4"></audio>
