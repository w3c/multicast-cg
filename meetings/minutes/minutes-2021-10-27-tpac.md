Please see the [IRC Log minutes](https://www.w3.org/2021/10/27-multicast-minutes.html) for a more readable version.

The text dump from that is here, with many thanks to Chris Needham:

~~~
IRC log of multicast on 2021-10-27
Timestamps are in UTC.

15:03:40 [RRSAgent]
RRSAgent has joined #multicast
15:03:40 [RRSAgent]
logging to https://www.w3.org/2021/10/27-multicast-irc
15:03:41 [takio]
takio has joined #multicast
15:03:46 [Zakim]
Zakim has joined #multicast
15:03:55 [cpn]
Meeting: Multicast CG TPAC meeting
15:04:07 [cpn]
Topic: Introduction
15:04:17 [cpn]
Jake: Welcome to the Multicast CG meeting
15:05:48 [cpn]
Present: Jake_Holland, Eric_Siow, Dominique_Hazael-Massieux, Gang_Shen, Kazuhiro_Hoya, Kyle_Rose
15:06:02 [cpn]
Present+ Chris_Needham
15:06:16 [cpn]
Present+ Takio_Yamaoka
15:07:31 [cpn]
... [Reviews Code of Conduct and IPR policy]
15:08:06 [cpn]
Present+ Sudeep_Divakaran
15:08:37 [cpn]
Jake: I'll walk through what we're doing and where we are, then a review of next steps for the work
15:08:51 [cpn]
... We'll schedule some sessions for the next month. Anything to add to the agenda?
15:09:01 [cpn]
... (none)
15:09:24 [dom]
Present+ AnssiKostiainen, XiaoqianWu
15:09:27 [cpn]
... We're aiming to add a multicast receive capability to browsers
15:09:45 [cpn]
... We've not decided on whether an API is needed
15:09:57 [cpn]
... It could possibly fit inside WebTransport
15:10:17 [cpn]
... There probably will be some form of API to allow web pages to use it, to leverage the multicast transport that exists
15:10:42 [cpn]
... The reasons for that are: user experience
15:10:44 [EricSiow]
EricSiow has joined #multicast
15:11:21 [cpn]
... [Shows histogram] During peak times, it gets a bit worse. Increases at the low-end of rates achieved
15:11:36 [cpn]
... This is normal, but on a heavy traffic day it gets bad
15:11:58 [cpn]
... Half the goodput for about 9 hours, e.g., when a game download is released
15:12:10 [cpn]
... These can be addressed with multicast, doesn't have to hammer the network
15:12:27 [anssik]
anssik has joined #multicast
15:12:35 [anssik]
Present+ Anssi_Kostiainen
15:12:39 [cpn]
... Sometimes it's CDN traffic, sometimes others. We get complaints from networks because users see lower than usual performance
15:12:56 [cpn]
... The key problem that multicast solves is congestion on the access networks to homes
15:13:12 [cpn]
... These are typically oversubscribed. You can put caches in networks, but there's no IP addressability
15:13:29 [cpn]
... Not just cable networks, also fiber networks and different access types. 5G can leverage these thigns
15:13:49 [cpn]
... The slides show a rough guide to how much offload is achievable
15:14:23 [cpn]
... With 3000 customers on a cable loop, if a large percentage is watching a particular event or downloading a game, traffic could be shared
15:14:40 [cpn]
... If you try to do it from cache, you have the same problem as you send it 1:1 to each user
15:15:07 [cpn]
... Possible to share the traffic, e.g., video on demand that's not popular enough. Large file downloads and live video streams
15:15:34 [cpn]
... Can talk separately about the transport protocols, but there's interest from ISPs
15:16:07 [cpn]
... Also the climate impact. The internet takes a lot of power. A BBC research project concluded that the internet takes 3.7% of total carbon footprint
15:16:12 [dom]
Present+ LouayBassbouss
15:16:32 [cpn]
... Some problems can't be addressed by multicast. We're looking to get that down to 1% of total carbon footprint
15:16:43 [EricSiow]
Q+
15:16:49 [xiaoqian]
xiaoqian has joined #multicast
15:16:55 [xiaoqian]
present+
15:17:06 [cpn]
... The article talked about proof of work bitcoin. Usage changes, but 20% day to day or 50% on big days is addressable through wider use of multicast
15:17:25 [cpn]
... The other reason is cost reduction
15:18:16 [cpn]
Eric: About climate change, we had a separate breakout session on sustainability
15:18:24 [dom]
-> https://www.w3.org/2021/10/19-sustainability-minutes.html Environmental Concerns and Sustainability (s12y) of Web Technologies - TPAC 2021 breakout
15:18:32 [cpn]
... There's some interest and movement to include sustainability as a horizontal review
15:18:41 [cpn]
... It's good you're including this
15:19:16 [cpn]
... If we're quoting percentages, we should be consistent, e.g, give the same measure for video games as well
15:19:36 [cpn]
... It would be good to educate stakeholders in terms of how much impact multicast would have
15:20:14 [cpn]
Jake: This is where we started. Before we came to W3C we did some work in IETF, then we went to ISPs to ask if they'd use it
15:20:36 [cpn]
... We showed them the traffic data, and potential reduction through multicast offload
15:20:54 [cpn]
... I've talked to more than 30 ISPs. There's been a range of responses
15:21:17 [cpn]
... Two said they don't plan to use it. One said they thought it would reduce revenue, they charge by traffic served
15:21:46 [cpn]
... Almost all ISPs were more positive, good to reduce traffic. Some identified logistical difficulties to roll it out
15:21:59 [cpn]
... We ran lab trials with four ISPs, showed it's practical
15:22:28 [cpn]
... Many were positive, different ranges of skepticism
15:23:16 [cpn]
... Some saw offloading peak traffic as beneficial, more so than the day to day, but that's where reductions are
15:23:32 [cpn]
... It's not an Akamai black-box, hence the standards approach
15:24:08 [cpn]
... [Avoidable traffic: video] Example of reduction in peak traffic in a sports event
15:24:42 [cpn]
Jake: The CG has created a getting started primer, to try out the API. I can help people get set up
15:25:00 [cpn]
... Where we are now, on the IETF side, is we're trying to get engagement on the security model
15:25:28 [cpn]
... We got feedback from the Chromium net-dev list. They rejected our PR as it would be a new type of mixed content, and needs a better security model
15:26:25 [cpn]
... With that feedback, Kyle wrote a security document. I'm hoping to talk about this at IETF secdispatch in a few weeks, then take it into a standard we can build from to run multicast safely for web traffic
15:26:35 [cpn]
... That's a precondition for getting into browsers properly
15:27:10 [cpn]
... We need something integrated in the browser for authenticating packets. Also QUIC integration
15:27:16 [EricSiow]
Q+ to ask question.
15:27:47 [cpn]
... The way backpressure works in that setup is different. There's a proposal for that in MBONED
15:28:07 [cpn]
... Could send the same kind of information in a QUIC frame in a WebTransport context
15:28:33 [cpn]
... Then there's signalling for browser APIs. The BBC QUIC multicast draft is a good place to start experimenting
15:29:04 [cpn]
... The current multicast CG charter says we'll go to WebTransport first, to get datagram support to plug in existing multicast applications thar un in walled garden context
15:29:33 [cpn]
... If you have WebTransport you can put something in front at the sender side that puts it into QUIC datagram framing, then unpack at the receiver side
15:29:59 [cpn]
... I can share a demo video. We ported our existing multicast receiver to WASM and used it to play video
15:30:23 [cpn]
Eric: How are active are browser vendors involved in the IETF discussion?
15:30:56 [cpn]
Jake: Not very. Eric Rescorla at Mozilla commented, but I'm hoping to see support
15:31:08 [cpn]
Eric: What about companies like Amazon?
15:31:46 [cpn]
Jake: I did get a call from Amazon. They signed a deal with the NFL for Thursday night football. We've explored whether it's viable to do this with multicast
15:32:15 [cpn]
... That has a lot of viewers, at the high end of what the internet can support
15:32:34 [cpn]
... They're supportive of the standards work we're doing. Not clear we'll be ready in their timeframe
15:32:57 [cpn]
... I've not seen them engage in the standards work, but I hope they will as they have a driving use case
15:33:24 [cpn]
... I've talked privately with people at Google. Not everyone is convinced. YouTube Live traffic
15:33:43 [cpn]
... I hope we can drive their interest, but I don't think we've successfully done that yet
15:34:16 [cpn]
... People in networking may be skeptical from the last attempt at multicast. A vast amount of work went into any source multicast, that didn't pan out
15:34:43 [cpn]
... RFC8815 deprecated anysource multicast for inter-domain usage. That was a big false start for multicast
15:34:59 [dom]
-> https://datatracker.ietf.org/doc/rfc8815/ Deprecating Any-Source Multicast (ASM) for Interdomain Multicast - RFC8815
15:35:12 [cpn]
... It's widely deployed in the operating systems. IGMP is already there
15:35:51 [cpn]
Eric: I think the first task for the CG is to articulate the pain points and benefit to stakeholders
15:36:32 [cpn]
... You have some natural stakeholders. Google has multiple groups, so YouTube could be a supporter. The goal is to get them to feel the pain points and get them on side
15:36:48 [cpn]
... How do you get the Chrome browser people comfortable with the security issue?
15:37:22 [cpn]
Jake: There's a chicken and egg problem. It needs ISPs interested in delivering it. I'd like to have the technical capability in place, that creates interest
15:38:05 [cpn]
... A few ISPs have joined the group but not active, e.g., Verizon and Comcast. I'd like to get more
15:38:11 [dom]
-> https://www.w3.org/community/multicast/participants Participants in the Multicast Community Group
15:38:26 [cpn]
... They'll be less interested in the browser work, which is the charter for this group
15:38:56 [cpn]
... I'd like to have a big-picture plan for how this comes together. The multicast group is about browser support, but it plays into the overall story of what needs to happen
15:39:06 [cpn]
... Some is outside standards groups, getting people to invest
15:39:24 [cpn]
Eric: If the business need is there, they'll commit people to standards work
15:39:51 [cpn]
... The benefit is obvious, but how do we rally the ecosystem? That's the immediate challenge
15:40:25 [cpn]
Jake: Getting demos closer to what's releasable helps. The standards work is important, but a browser fork that does something usable is a key next step to make the case
15:41:04 [cpn]
... The other piece is getting buy-in from the browser tech side. Most objections have come from the security side. The proposal needs to be credible for people to pay attention
15:41:41 [cpn]
... So my proposal for next steps is the demo. If it can play video and there's some consensus in standards that it satisfies security concerns
15:41:58 [cpn]
... We don't have the level of buy-in from the people who stand to gain
15:42:19 [cpn]
... Similar story for CDNs and content owners, who can realise cost savings
15:43:25 [cpn]
Dom: The conversation may be bound to having a timeline. How much time is needed to align things?
15:43:56 [cpn]
... How long is needed for the demo, then getting stakeholder input using the demo?
15:44:28 [cpn]
Jake: So I have a demo running, that's close to ready. There's a Chromium binary you can download and install
15:44:36 [dom]
-> https://www.w3.org/2021/10/TPAC/demos/multicast.html Multicast for the Web - TPAC 2021 demo
15:44:38 [cpn]
... I can share a link to the demo
15:45:15 [cpn]
... There's the open question of whether it can ship in a browser. If it's there, will ISPs use it, then will content owners use it?
15:45:46 [cpn]
Dom: I'm hearing that the security is a key part of the plan to build confidence to use it
15:46:36 [cpn]
Jake: ?? and Internet2 have multicast working groups. They are supporting research, and I'm working with them
15:46:57 [cpn]
... Ingest protocol for how traffic gets into the network. Our labs tests with ISPs looked at this
15:47:08 [cpn]
... A number of moving parts are essential to adoption, but outside W3C
15:47:22 [cpn]
... In this group, what I'm hoping to do is move forward the receive side of the story
15:47:37 [cpn]
... I encourage people to engage in IETF, to get the whole picture to stakeholders
15:47:49 [cpn]
... There's a lot of money at stake and potential big impact
15:48:12 [cpn]
Gang: I think customers want this to be as transparent as possible
15:48:28 [dom]
s??/GEANT/
15:48:47 [cpn]
... DVB enable this through a MPEG DASH player. They have an easier problem
15:49:27 [cpn]
... We're considering the receiver side, but what about the server side, e.g., the 5G edge which is closer to the last mile. Our target is to resolve the last mile bandwidth issue
15:49:39 [cpn]
... Can we simplify this security model issue?
15:50:01 [cpn]
Jake: If you really make use of the broadcast capability at the edge, it may be possible to do something
15:50:57 [cpn]
... The issue is that you're trusting the edge servers. Handing off control, if you send somethign that an ISP on the path can change, they could do that. This is a big source of concern at IETF
15:51:55 [cpn]
... It's part of what we addressed with AMBI authentication, there's not a way for it to be spoofed, it creates and end to end authentication chain, so there's no scope any more for ad theft or unauthenticated injection
15:52:03 [cpn]
... That's one of several problems
15:52:37 [cpn]
... In a walled garden, there's a different set of problems. Could be how you get the scale in the end
15:52:59 [cpn]
... But it only works for video, gameplay and other downloads need different solutions
15:53:18 [cpn]
... 4G had this capability, but people didn't use it. The north-bound interface is hard
15:53:27 [cpn]
... Multicast TV has been used, but not inter-domain yet
15:54:01 [kazho]
kazho has joined #multicast
15:54:10 [cpn]
Gang: There's an incentive for ISPs to make changes, saving bandwidth. With unicast fallback it's easy to make changes
15:54:37 [cpn]
Jake: I think we way we proposed is easy, just pass through the streams. A design question is which way that can be done
15:55:18 [cpn]
... Give them a coherent object to unpack and distribute, or how to pass through an ingested stream
15:55:40 [cpn]
... ISPs tend to prefer this model, they have less to maintain, as they can focus on moving packets rather than end-user applications
15:56:08 [cpn]
... If it's just IP that you pass through, and senders and receivers decide how it's interpreted, there's less for them to do
15:56:13 [cpn]
... It's a great question
15:56:37 [cpn]
Jake: What I'd like to do in the next month is some experiments with QUIC and WebTransport API integration
15:56:59 [cpn]
... If anyone would like to join me, I'd like to collaborate with people, probably after IETF
15:57:47 [cpn]
... Not sure whether the BBC nghq QUIC or aioquic will be easier, that's part of the investigation
15:58:00 [cpn]
... If you can comment on secdispatch, it's an important moment
15:58:20 [cpn]
... There's a hackathon opportunity to work on the browser demo, next week
15:58:37 [cpn]
... I can introduce you to the right people to follow up
15:58:52 [cpn]
present+ LouayBassbouss
15:58:57 [cpn]
Topic: Wrap up
15:59:33 [cpn]
Jake: Thank you for all for coming. I hope I've piqued your interest to look into this
16:00:05 [cpn]
... The CG meets monthly, first Wednesday in December 7:30am Pacific. Feel free to contact me offline before then
16:01:14 [cpn]
rrsagent, draft minutes
16:01:14 [RRSAgent]
I have made the request to generate https://www.w3.org/2021/10/27-multicast-minutes.html cpn
16:01:18 [cpn]
rrsagent, make log public
16:02:51 [takio]
takio has left #multicast
18:58:31 [Zakim]
Zakim has left #multicast
~~~
