---
title: "Stream a remote screen with WebRTC"
author: "Rafael Viscarra"
cover: "/images/webrtc-remote-screen/cover.png"
tags: ["webrtc", "linux", "h264", "golang", "pion"]
date: 2019-07-31T22:13:29-06:00
---

Recently I partcipated in a project that involved some server-side GPU rendering, 
due to the nature of the technologies we used we needed to run an X server on 
our boxes. The initial setup was relatively painless until a weird bug started 
appearing in some server and the logs didn't provide enough information to 
debug effectively, this is how I ended up coding a remote viewer that streamed 
a remote server "screen" using WebRTC.

<!--more-->

TL; DR: The complete source code can be found on 
[this repo](https://github.com/rviscarra/webrtc-remote-screen).

### Ok, but why?

I'm aware of other solutions to this problem, like setting up 
[Virtual GL](https://wiki.archlinux.org/index.php/VirtualGL#Using_VirtualGL_with_VNC)
to stream a remote X server's video thru VNC. I didn't like this solution 
because:

1. It isn't as easy to setup as copying a zip file, decompress it and running it.
2. It would have to be installed and started on every instance beforehand.
3. It would be another component to maintain with its own unknowns and issues.

### General Overview

Here's the basic gist of what the tool does:

1. The service starts and listens on port 9000 by default, this can be changed 
with a flag.
2. The service exposes two endpoints, `POST /session` to start a session and 
`GET /screens` to get the available screens from the remote server.
3. Once a screen is chosen, a SDP offer is created in the browser,
for simplicity there's a workaround in place to _avoid_ trickle ICE.
4. Once all ICE candidates have been gathered, a `POST /session` request with 
the selected screen and browser's SDP is made.
5. The backend inspects the offer, checks if the browser supports h264 baseline 
3.1 and if everything goes well, the server responds with a SDP answer.
5. Now the browser should receive and show the video stream.

### Implementation details

#### h264

I decided to use h264 instead of VP8 because the Golang binding for h264 feels 
more idiomatic and well thought than the VP8 one. It also means we can support 
[older versions of Safari](https://webkit.org/blog/8672/on-the-road-to-webrtc-1-0-including-vp8/).

#### Sendonly support

Since this application only needs to stream media to the browser, I needed 
support for sendonly transceiver on the server. 
After going thru Pion's code I realized that it wasn't possible at the time. 
However, not everything's so bad, the need for this feature pushed me to 
implement it and 
[contribute to the Pion repo](https://pion.ly/knowledge-base/pion-internals/contributing/)! 

#### X Server (only?) support

The library I'm using to take screenshots supports multiple platforms, so in 
_theory_ it should work but I haven't tested it, if you do YMMV. The only _hard_ 
dependency is libx264 (see the h246  section above).

#### Client-side code

I wanted to keep things simple so I didn't setup Webpack/Babel to transpile to 
ES5 or divide my application between multiple files. The main reason for this is 
that I wanted to make _deployment_ as simple as possible, zip it, scp'it, unzip 
it and run!
 
### The curse of Firefox's WebRTC bug

I originally intended to release this repo / blog post a few weeks ago, since 
the project was a good MVP back then. However, life's not so simple ... just as 
I was preparing the repo's README and tried to take a screenshot in FF I 
realized that I wasn't working which I thought was weird since I got it working 
before.

It turns out that I hit [this](https://bugzilla.mozilla.org/show_bug.cgi?id=1333879) 
weird and relatively old bug that makes Firefox show a "black" video instead of 
the incoming stream when Firefox is the offerer. It took a while to debug and 
fix since there was basically no trace of errors (and as a _seasoned_ developer 
I automatically assume the bug is my fault).

Luckily it wasn't hard to fix, I ended up doing something like this:

{{<highlight go>}}

func findBestCodec(sdp *sdp.SessionDescription, profile string) (*webrtc.RTPCodec, error) {
	for _, md := range sdp.MediaDescriptions {
		for _, format := range md.MediaName.Formats {
			intPt, err := strconv.Atoi(format)
			payloadType := uint8(intPt)
			sdpCodec, err := sdp.GetCodecForPayloadType(payloadType)
			if err != nil {
				return nil, fmt.Errorf("Can't find codec for %d", payloadType)
			}

			if sdpCodec.Name == webrtc.H264 {
				packetSupport := strings.Contains(sdpCodec.Fmtp, "packetization-mode=1")
				supportsProfile := strings.Contains(sdpCodec.Fmtp, fmt.Sprintf("profile-level-id=%s", profile))
				if packetSupport && supportsProfile {
					var codec = webrtc.NewRTPH264Codec(payloadType, sdpCodec.ClockRate)
					codec.SDPFmtpLine = sdpCodec.Fmtp
					return codec, nil
				}
			}
		}
	}
	return nil, fmt.Errorf("Couldn't find a matching codec")
}
{{</highlight>}}

It basically looks for the codec I support based on the codec name and profile, 
H264 Constrained Baseline 3.1 (`42e01f`) in this case.

### How does it look?

Here it is, working on Firefox

![Screenshot](/images/webrtc-remote-screen/demo.png)

And finally, a brief capture of it (looks better in HD).

<iframe src='https://gfycat.com/ifr/ThunderousBaggyJunebug' frameborder='0' scrolling='no' allowfullscreen width='640' height='512'></iframe>

{{<include_md file="partials/job.md">}}