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

Ok, fair enough. When we’re developing our tests prior to them being run on the spacecraft, it’s useful to have *something* to upload.

## Testing the Tests

All developed tests need to be run against our hardware functional test assembly, which is some spare units of the most important hardware components on the spacecraft, all held together by what seems like duct tape and the hopes and dreams of every engineer who steps into the lab. 

## The Problem

The placeholder memory addresses were the **actual memory addresses used for a previously developed spacecraft**. As such, the name of the file containing the addresses was something like “XProp_transducer_addr_val.upx”. In all of the review meetings, this was most likely glossed over because of how official it looked. All the engineers who worked on the last spacecraft assumed the naming convention was the same on this one, and those like me who were working on their first spacecraft were none the wiser. 

The day comes and it's finally time to run this test. Naturally, we get an unexplainable error, which is something I wish I could say we weren't used to. The standard procedure is to stop the test and asses the vehicle state before deciding if we can redo the last test or continue with other tests on different systems. We decide to run a seperate test but are still getting some strange errors, so we agree to turn the spacecraft off and back on to get it in a nominal configuration. Again, fairly standard procedure.

Except turning the spacecraft off and on isn't as simple as just flipping a switch or unplugging it. There are about a dozen steps and checks on all the subsystems that need to happen in a fairly specific order to ensure no hardware gets damaged in the process. We're constantly checking telemetry during this teardown to nothing is on when a component further down the process is about to turn off. 

In our teardown script, we get an error we've never seen before. Some of the motors on the vehicle are not turning off. Ok, that's weird. Let's send the off command again, maybe there was a routing failure. No luck. Hmmmm. Well, unfortunately we can't just continue, but we've got a few ideas  

The **E**xtended **R**emote **I**nterface **U**nits (**ERIU**s) 