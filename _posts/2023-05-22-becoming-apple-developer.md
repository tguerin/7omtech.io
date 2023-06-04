---
title:  "Becoming Apple Developer: The Hard Way"
subtitle: "When activating an Apple developer account is not as easy as it should be"
date:  2023-05-22 15:04:23
categories: [Release Process]
image: /images/apple-logo.jpeg
tags:  
- Apple Development
- App Store Process
- Release
---

I‘ve been an Android Developer for almost 10 years now, recently I was playing around with Flutter
and was struck by a personal fact: "I’ve never published a personal mobile application on the stores".

Well… in my previous jobs I was responsible for publishing releases, but I had never built and released
a personal project on stores. This year, I decided things were going to change and built an app about fun/interesting facts.
Factify Facts was born and it was time to release it on the stores ([iOS](https://apps.apple.com/us/app/factify-facts/id6448727476) and [Android](https://play.google.com/store/apps/details?id=com.factify.app)).
I already own a PlayStore developer account but no Apple Developer Account. Spoiler alert: becoming an Apple Developer
was not a ”Piece of Cake“.

## Updating my Apple Id

I got the first macbook during my studies thanks to the Apple student program and it was looking like this:

![](/images/2023-05-22-becoming-apple-developer/mcbook.jpg)

At that time I used a _hotmail.com_ mail address to register. For the youngest one it was the ancestor of outlook (damn I feel old). 
This hotmail mail address became by default my Apple Id. For my developer account, I wanted to update this Apple Id to use my _gmail_ account instead.
Once updated on [appleid.apple.com](https://appleid.apple.com/), I logged in again on my mac and everything was working well.

So far so good, I downloaded the Developer app from AppStore to start enrollment. Upon logging in, the app was requested the password 
for my _hotmail_ account and not my _gmail_ account. Long story short I ended in state where I could enter my new Apple Id and password but a dialog was popping later requesting my password for my old account.
None of the password was working, so I was totally stuck.

![](/images/2023-05-22-becoming-apple-developer/begins.gif)

I reverted my change on [appleid.apple.com](https://appleid.apple.com/) back to my _hotmail_ account and everything went back to normal.

Ok Apple, I will use my _hotmail_ account!

## Enrolling with Developer App on Mac

I started the enrollment within the Developer app. The first step was to take a picture of an Id card with the front 
camera of the mac. I used my driving licence, took a sharp picture and my first name ended up being Amomas and not Thomas.
For some reason you can't edit the fields to correct them and you can't go back… 

I contacted Apple Support, two days later the support had reset my personal information. In the mail, support confirmed 
that It was not possible to continue the enrollment in the Developer app. I had to finish it on the web.

## Dead ending

I started over the enrollment on the web, completed all the step. On the last page, I was excited to click on the submit button
and become an Apple Developer. The excitement became incomprehension as soon as I got this error message:
```Your enrollment in the Apple Developer Program could not be completed at this time.```
I love this kind of error message cause I have no idea at all what to do: should I try again? is it a temporary error? a server issue?

After having read some Apple community threads, I realized I was not the only one. I checked everything: my credit card, my birthdate, apple agreements, everything…

I ended sending an mail to Apple Support again and 2 days later I got the most epic mail:

```
Thank you for contacting Apple Developer Program Support.

My name is xxxxx and I am delighted to assist you today.

From your message, I understand that you are still unable to register for the Apple Developer program.

We cannot verify your identity through the Apple Developer app or provide additional support regarding this Apple ID for Apple developer programs.

You can still enjoy great content by using your Apple ID in Xcode to develop and test apps on your own device.
```

This was the most polite way I had ever seen to tell me: “Your Apple Id is fucked up, we can't do anything but go play around”

“I WANT TO SUBMIT AN APP” I repeat “I WANT TO SUBMIT AN APP”.

![](/images/2023-05-22-becoming-apple-developer/everybodystaycalm-staycalm.gif)

## New Apple Id

As I was not using iCloud services, I was ok to create a new Apple Id. Finally, I could use my _gmail_ account!
This time I took time to ensure that:
* All my personal information was filled on Apple Id website with a valid birthdate (18+)
* 2FA was enabled and working properly
* My credit card information was setup in Apple Id and AppStore app
* iCloud services were enabled

Once everything was double (triple) checked, I started over on the web this time. I filled all the information
paid 99$ and 2 hours later I got this mail:

```
Hello Thomas,
Thank you for joining the Apple Developer Program. We’re thrilled to support your work as you bring your ideas to life! As a developer program member, you have access to a comprehensive set of resources to build, test, and distribute innovative apps for Apple platforms.
```

You would think that the story is over, but noooo…

## From Personal to Company Account

Few days later, I was finishing the in-app purchase implementation for my app when I started to wonder: “Wait a minute, how am I going to declare this revenue?”
As I became a freelancer earlier this year, there is a rule in France that is:
```If you own a company in a specific field, you need to declare all revenue from that field```

In fact, I must declare my revenue on my freelance company. I contacted Apple Support to request a migration to company account.

Few days later, I got the answer and had to provide following information:
* Your organization’s legal entity name in Roman characters
* Your organization's [D-U-N-S Number](https://developer.apple.com/support/D-U-N-S)
* A valid organization website is required

Thanks to Apple you can enjoy this new website cause at the time of the request it didn't exist. Regarding
DUNS Number, Apple provide a tool to search your company. Don't pay for your DUNS number!

After everything was setup by mail, I got a call to ensure that I was actually me. Once confirmed that me was me,
the migration process started. Two days later I finally got my company developer account.

![](/images/2023-05-22-becoming-apple-developer/champagne.gif)

## Summary

As this article started to be a bit longish, I decided to elude the French specific part where I had to fill
the W8ben then W8ben-e form for U.S. tax. 

It took more time to have my developer account up and ready
than developing Factify Facts. The main point to keep in mind when creating your developer account is to check
that everything (birthdate, credit card, etc…) is correct. Don't hesitate to contact support, they honestly did a good job and helped me a lot. Expect a 2 days delay for your answer though.

Finally, for anyone wishing to be an Apple Developer, I wish you good luck!
