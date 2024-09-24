---
title: How to avoid a BSOD on your 2 billion dollar spacecraft
layout: post
date: 2024-09-04 18:57:41
tags: aerospace
description: A close call
hidden: true
---

Short answer: turn it off and turn it back on

Long Answer…

### Background

The lifecycle of most spacecraft consists of a final phase where all the systems are tested to various levels of synergy. One of the most important and complex set of tests are the **C**losed **L**oop **T**ests (**CLT**s), where the spacecraft is sent simulated orbital data, and then it’s attitude response is observed. It’s a closed loop because the attitude telemetry is fed back into the simulation while the test is occurring, effectively making the spacecraft and whatever hardware is currently being used part of the simulation. This particular test involved observing the response from control thrusters on the spacecraft when commanded to perform a slew and engine burn to a transfer orbit. 

To get the spacecraft response data back to in the loop, a set of memory addresses mapped to the sim needs to be uploaded onto the spacecraft RAM. These memory addresses were not determined while this test was being developed. Instead, a set of placeholder addresses used on the previous spacecraft was used in the development environment.

Ok, fair enough. When we’re developing our tests prior to them being run on the spacecraft, it’s useful to have *something* to upload, even if it's not the final product.

## Testing the Tests

All developed tests need to be run against our hardware functional test assembly, which is some spare units of the most important hardware components on the spacecraft, all held together by what seems like duct tape and the hopes and dreams of every engineer who steps into the lab. 

## The Problem

The placeholder memory addresses were the **actual memory addresses used for a previously developed spacecraft**. As such, the name of the file containing the addresses was something like “XProp_transducer_addr_val.upx”. In all of the review meetings, this was most likely glossed over because of how official it looked. All the engineers who worked on the last spacecraft assumed the naming convention was the same on this one, and those like me who were working on their first spacecraft were none the wiser. 

The day comes and it's finally time to run this test. Naturally, we get an unexplainable error, which is something I wish I could say we weren't used to. The standard procedure is to stop the test and asses the vehicle state before deciding if we can redo the test or continue with other tests on different systems. We decide to run a seperate test but are still getting some strange errors, so we agree to turn the spacecraft off and back on to get it in a nominal configuration. Again, fairly standard procedure.

Except turning the spacecraft off and on isn't as simple as just flipping a switch or unplugging it. There are about a dozen steps that need to happen in a fairly specific order to ensure no hardware gets damaged in the process. We're constantly checking telemetry during this teardown to nothing is on when a component further down the process is about to turn off. 

In our teardown script, we get an error we've never seen before. Some of the motors on the vehicle are not turning off. Ok, that's weird. Let's send the off command again, maybe there was a routing failure. Still nothing, but the commands are showing as "recieved". Hmmmm. Well, unfortunately we can't just continue, but we've got a few ideas as to what's happening.

Some of the telemetry that's being downlinked from the satellite is showing as active but showing no variation. I.e., a voltage or temperature reading is actively being downlinked as the same value, down to five sig figs, with no change. Further inspection of telemetry shows errors on the connection between the onboard computer and the ERIU.

The **E**hanced **R**emote **I**nterface **U**nits (**ERIU**s) are how the computer onboard the spacecraft communicates with all the different sensors and systems that read actual orbit and mission data. Think of the ERIUs as the pony express, relaying commands from the onboard computer to the wild west frontier of the satellite sensors, and sensors in turn sending telemetry back through the express to the onboard computer for processing. In order to know where to send commands and telemetry, the ERIU reads addresses from a specifc part of memory onboard the spacecraft.

I think you can see where this is going. The addresses we loaded at the start of the test are meant for a completely different spacecraft and simulation setup. When the ERIU tried to send telemetry back to the sim, the addresses it was given weren't actually pointing to anything, and it bascially throws a null pointer dereference and crashes. Somehow, the last valid telemetry it received is still being repeatedly transmitted to the onboard computer. 

There are two rather significant problems that are immediately apparent to us:

1. Because the ERIU has crashed, we have no way of commanding different subsystems off, and cannot safely enter a powered off vehicle state.
2.  We are no longer getting active telemetry from the vehicle, which means that if something bad were happening to any system, we would not know about it.

Problem #1, can luckily be put off for the time being while we focus on the more time sensitive problem #2. I should clarify that the vehicle is very safe and stable in the configuration that it's in. There is almost zero chance that something significant would break while we're in this tricky spot of not recieving valid telemetry. That being said, as an engineer, being unable to describe your state with a high degree of accuracy is incredibly concering. An apt comparison would be if you were the flying an airplane at cruise altitude and suddenly your altimeter, airspeed indicator, and attitude indicator suddenly stopped updating. 