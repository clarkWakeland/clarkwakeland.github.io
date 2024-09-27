---
title: How to avoid a BSOD on your 2 billion dollar spacecraft
layout: post
date: 2024-09-04 18:57:41
tags: aerospace
description: A close call
hidden: false
---
<div class = "container">
    <div class = "row">
    <div class="col"></div>
        <div class = "col-12">
        {% include figure.liquid path="assets/img/satellitesafemode.png" title="" class="img-fluid rounded z-depth-1" %}
        </div>
    <div class="col"></div>
    </div>
</div>

Short answer: turn it off and turn it back on

Long Answer…

### Background

The lifecycle of most spacecraft testing consists of a final phase where all the systems are tested to various levels of synergy. One of the most important and complex set of tests are the **C**losed **L**oop **T**ests (**CLT**s), where the spacecraft is sent simulated orbital data, and then its attitude response is observed. It’s a closed loop because the attitude telemetry is fed back into the simulation while the test is occurring, effectively making the spacecraft and whatever hardware is currently being used part of the simulation. This particular test involved observing the response from control thrusters on the spacecraft when commanded to perform a slew and engine burn to a transfer orbit. 

To get the spacecraft response data back to in the loop, a set of memory addresses mapped to the sim needs to be uploaded onto the spacecraft RAM. These memory addresses were not determined while this test was being developed. Instead, a set of placeholder addresses meant for the previous spacecraft was used in the development environment.

Ok, fair enough. When we’re developing our tests prior to them being run on the spacecraft, it’s useful to have *something* to upload, even if it's not the final product.

## The Vehicle

The placeholder memory addresses were the **actual memory addresses used for a previously developed spacecraft**. As such, the name of the file containing the addresses was something like “XProp_transducer_addr_val.upx”. In all of the review meetings, this was most likely ignored because of how official it looked. All the engineers who worked on the last spacecraft assumed the naming convention was the same on this one, and those like me who were working on their first spacecraft had no other point of reference. 

The day comes and it's finally time to run this test. Naturally, we get an unexplainable error, which is something I wish I could say we weren't used to. The standard procedure is to stop the test and assess the vehicle state before deciding if we can redo the test or continue with other tests on different systems. We decide to run a seperate test but are still getting some strange errors, so we agree to turn the spacecraft off and back on to get it in a nominal configuration. Again, fairly standard procedure.

Except turning the spacecraft off and on isn't as simple as just flipping a switch or unplugging it. There are about a dozen steps that need to happen in a fairly specific order to ensure no hardware gets damaged in the process. We're constantly checking telemetry during this teardown to see that nothing is on when a component further down the process is about to turn off. 

In our teardown script, we get an error we've never seen before. Some of the motors on the vehicle are not turning off. Ok, that's weird. Let's send the off command again, maybe there was a routing failure. Still nothing, but the commands are showing as "received". Hmmmm. Well, unfortunately we can't just continue, but we've got a few ideas as to what's happening.

Some of the telemetry that's being downlinked from the satellite is displaying as active but showing no variation. I.e., a voltage or temperature reading is actively being downlinked as the same value, down to five sig figs, with no change. Further inspection of telemetry shows errors on the connection between the onboard computer and the ERIU.

The **E**nhanced **R**emote **I**nterface **U**nits (**ERIU**s) are how the computer onboard the spacecraft communicates with all the different sensors and systems that read actual orbit and mission data. Think of the ERIUs as the pony express, relaying commands from the onboard computer to the wild west frontier of the satellite sensors. The sensors in turn send their telemetry back through the ERIU to the onboard computer for processing, or anywhere else that was specified in memory, like a CLT sim. 

<div class = "container">
    <div class = "row">
    <div class="col"></div>
        <div class = "col-8">
        {% include figure.liquid path="assets/img/ponyExpress.jpg" title="" class="img-fluid rounded z-depth-1" %}
        </div>
    <div class="col"></div>
    </div>
    <div class = "caption">
    They don't make ERIUs like they used to
    </div>
</div>



I think you can see where this is going. The addresses we loaded at the start of the test are meant for a completely different spacecraft and simulation setup. When the ERIU tried to send telemetry back to the sim, the addresses it was given weren't actually pointing to anything, and it bascially throws a null pointer dereference and crashes. Some bus communication quirk is causing the last valid telemetry broadcast by the ERIU to be continously sent to the onboard computer.

## The Problems

There are two significant problems that are immediately apparent to us:

1. Because the ERIU has crashed, we have no way of commanding different subsystems off, and cannot safely enter a powered off vehicle state.
2.  We are no longer getting active telemetry from the vehicle, which means that if something bad were happening to any system, we would not know about it.

Problem #1 can luckily be put off for the time being while we focus on the more time sensitive problem #2. I should clarify that the vehicle is in a very safe and stable configuration at the moment. There is almost zero chance that something significant would break while we're in this tricky spot of not recieving valid telemetry. That being said, as an engineer, having no way of describing your state is incredibly concering. An apt comparison would be if you were the flying an airplane at cruise altitude and your altimeter, airspeed indicator, and attitude indicator suddenly stopped updating. You're *probably* not going to have anything bad happen to you, and there's not much reason to believe your state is rapidly changing, but I doubt any pilot would want to experience that.

Ok, we've got a clear goal: reboot the ERIU. Just like waking up for a morning swim, it is much easier said than done. In fact, the only way to reboot the ERIU is to reboot the whole damn onboard computer. This of course comes with its own set of issues and risks.

The onboard computer, as part of its normal operation, continuously restarts a watchdog timer. If the watchdog timer has not been restarted and instead times out after ~30 seconds, the satellite enters something called **safemode**. Safemode is when all non critical functions are automatically shut down and the satellite becomes entirely focused on generating power by pointing its solar panels towards the Sun and trying to reestablish any communication that was lost. It's a state the vehicle goes into when something bad happens, like total loss of attitude control or some other system failure. 

<div class = "container">
    <div class = "row">
    <div class="col"></div>
        <div class = "col-8">
        {% include figure.liquid path="assets/img/vaderfighter.gif" title="" class="img-fluid rounded z-depth-1" %}
        </div>
    <div class="col"></div>
    </div>
    <div class = "caption">
    Vader's tie fighter entering safemode
    </div>
</div>

Safemode is the satellite equivalent of a blue screen of death. An unexpected safemode occurring on the satellite during testing is something that must be communicated to the customer, even though it's completely recoverable. Again, I wish I could say this hasn't happened before. Long story short, the US government isn't burning taxpayer dollars on a ten figure spaceship just to have us push a Crowdstrike update on it. They would be pretty upset, to put it lightly.

Alright. Let's not focus on the worst outcome. We can disable the watchdog timer, turn off some other telemetry, finally power cycle the onboard computer, and then turn everything else back on. Oh, and we need to do the exact same thing at the same time on the redundant computer as well. And there are a couple dozen commands in this manually created sequence, and getting one of them wrong could send the satellite to safemode. We got this. Did I mention that all of this is happening at midnight on a Saturday?

After confirming our commands with a very tired flight software lead, we send the sequence, and it works! The onboard computer is back on, the ERIU is back on, and we're getting what looks like reasonable telemetry from the other systems. We continue with the power off sequence and sure enough everything is powering down as expected. After ~12 hours of troubleshooting and communicating with other system engineers, we finally power down the vehicle safely at 1:45am. 

## Retrospective

I hope I managed to explain the relevant systems in enough detail and not use too many acronyms, and at least explain the ones I did use. Working on space systems has exposed me to a seemingly infinite amount of different acronyms, some of which are the same but have different meanings. There was also a lot I glossed over, with the main time sink being the proper identification of the issue. There were many red herrings that were chased before we correctly determined that it was an ERIU crash. This would have ended much worse without the support of the amazing systems engineers who answered our calls in the middle of the night on a Saturday.

I think what surprised me the most was how nonchalant the response was. We had documented all of our actions, so other people had read what happened and knew something had gone on. I wasn't expecting any fanfare but we weren't even debriefed on what happened. I guess this is the event that really got the point across to me about how if you do your job right, it'll be like nothing ever happened. But I'll take that over a BSOD on a multi billion dollar satellite any day.

## 2024-09-26 Update Re: Hacker News Post

[Link to Hacker News post](https://news.ycombinator.com/item?id=41650534)

Yesterday (Sept 25th) I posted this article to Hacker News. It wasn't anything groundbreaking but it did get considerably more popular than I expected. I'll try to clarify things that were asked in the HN post.

1. *Why is the satellite running Windows?*
    
    The satellite was not running Windows. It has a custom OS called Flight Software (yes, that's its actual name) or FSW for short. I believe it is an RTOS based on Wind River's vxworks but my details are a bit shaky on this and could be wrong. I was using a BSOD as an analogy for the satellite going into safemode. I can see how the title of the article and other parts of the post would give the impression that the satellite was running Windows. In the future, I'll try to clarify this types of things more. 

2. *Why wasn't this caught in test verification?*
    
    The Functional Test Assembly (FTA) which we use to vet our scripts is incredibly unreliable. Additionally, other teams are constantly fighting for space in this lab to perform their own testing. For the CLT mentioned in the article, we were unable to get a clean run in the FTA, but it was decided to go ahead and run on the vehicle due to time constraints. At the time, the memory address mapping and ERIU crashing issue was not well understood. If it was, it certainly would not have been run on the satellite. 

Overall, I can see how the title of the article a little click-baity. After reading some of the comments, I debated changing the title, but eventually decided to leave it as is. I still believe that safemode, while not exactly a BSOD, is close enough that the comparison still holds. Outside of correcting a couple typos (which I got some nice emails correcting me about haha) and clarification in the first sentence to specify *testing*, the content is unchanged. Although maybe I should remove the Crowdstrike reference, given how dated that it'll seem in a few months/years.

I'm most happy that the post seems to have struck a chord with people and got them interested, engaged, and asking questions. An interesting symbiosis happened where the readers were able to learn about something new, and I was given an outside perspective on my writing that'll allow me to communicate ideas more clearly. This is definitely an iterative process for me, and I'm looking forward to taking this feedback into account on my future writings.

-Clark

    


