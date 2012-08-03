---
title: Displaying YouTube video in WebGL
date: '2012-07-28'
description: My adventures getting YouTube videos to display on WebGL canvas
categories: [webgl, youtube, video, html5]
---

<style>
   .footnote {
      font-size: 80%;
      padding-top: 20px; /* TODO - NOT how I want to do this, tidy up later */
   } 

</style>

<span style="font-style: italic">My adventures in getting YouTube videos to display on a 3D WebGL canvas.</span>

<!-- maybe add postramble to the template? -->
<a href="#postramble">Cut To The Chase </a>

<p>I started learning webgl because it seemed fun, and thanks to the ace tutorials at <a href="http://learningwebgl.com">Learning WebGL</a> I got up to speed hand-coding WebGL relatively quickly. Drunk on the achievement of drawing some 3d spinning squares <a href="#footnote1">*</a>, I decided to pursue my goal of making a 3D data visualisation of some charts we make in work. Oh and it should display video too. Turns out this is kindof hard...but not impossible. I'm quite sure this would be easier with the <a href="http://mrdoob.github.com/three.js/">freaking awesome three.js</a>, but I wanted to start off learning it by hand. Anyway, this is a summary of where I got to with displaying YouTube video on WebGL.</p>

<a name="postramble"></a>
<a name="webgl_video"></a><h2>Video in WebGL</h2>
<p>First things first: normal video in WebGL. Displaying videos in WebGL is surprisingly easy, once you're used to setting up image textures for your objects. Basically all you do is use a playing HTML5 video element as the source for your texture instead of an image element; WebGL is smart enough to know to use the current frame of the video. You only need to update the texture as normal, pointing your texture at a video element instead of an image element, and your video will play on your WebGL canvas. Awesome, right? </p>

<!-- also need to link to the MDN article about this stuff -->
<script src="https://gist.github.com/3206901.js"> </script>
<script src="https://gist.github.com/3206919.js"> </script>

<a name="webgl_npot"></a><h2>Video WebGL Argh 1: NPOT</h2>
<p>There are a few problems with this. The usual caveats of needing a power-of-two-sized image for your texture to render apply(often called the NPOT problem) - you basically need an image that is something like 256x512 for it to be usable by WebGL, so if your video is not this size, it won't be used by the texture. To get around this, you can throw images or video through a canvas that <span style="font-style: italic">is</span> the correct size, black out the bits around your video, and the use the canvas element as the texture source. Again, I have strong feelings towards the HTML5 specs. right now. :D </p>

<!-- deffo need a diagram and gist 2! -->
<script src="https://gist.github.com/3206887.js"> </script>

<a name="webgl_xdomain"></a><h2>Video WebGL Argh 2: X-Domain Issues</h2>
<p>Another problem you'll hit very fast is cross-domain issues with WebGL textures. As of when I wrote this, you can't use an image from another domain as a texture (you'll get a DOM Warning [Exception 18?] if you try and the texture won't be used. To get around this I fired up a python server and used it to proxy requests for videos, which is not the most beautiful solution imaginable, but its effective. I'm hoping that a cleaner way to use texture sources from other domains is developed (if there is one, please let me know!).</p>

<a name="webgl_youtube"></a><h2>Video WebGL On YouTube</h2>
<p>So those are the major problems with getting HTML5 video to play on a WebGL canvas. Unfortunately getting this to work with YouTube is not that simple! YouTube support for HTML5 video is still experimental, because its a developing standard and some of the Flash features they rely on aren't available in HTML5 (content protection and adaptive streaming are the biggies so far I think). That said, a lot of videos are available in HTML5 video form, and you can request that you get the HTML5 version by appending "HTML5=1" to the query string when you request your iframe embed from YT.</p>

<a name="webgl_youtubeembed"></a><h2>Video WebGL On YouTube Argh 1: Embed Code Access</h2>
<p>YouTube's current video embed is an iframe tag, which presents another obstacle to using it for our texture - you can't access the contents of the iframe (and thus the video tag) because the iframe is hosted by youtube.com's domain, not ours. Proxying the iframe request doesn't work here either, because the iframe contents refers to relative URLs. I guess actually you could map everything through the proxy, but that's not the approach I ended up taking.</p>

<p>I noticed that the YouTube embed requests a file with information about videos it could play (http://www.youtube.com/get_video_info?html5=1&video_id=<FOO>), and this contains URLs of videos that the iframe embed might play. I'm not sure if YouTube use the user agent to optimise which video codecs to return (e.g. preferring webm in FF or similar), but I always forwarded the original user agent when I made my request just in case. Its possible to use the response to figure out the URL of the video the YouTube player would play, and we can use that to create a video embed we have access to, which can in turn be used by the WebGL texture. Phew!</p>

<script src="https://gist.github.com/3206868.js"> </script>

<p>Last of all, there's a slight delay from doing the request to get the video info, parsing the video info, and returning the video file, so I pushed that logic into a background process, and cached the video files on S3 . Its also worth being aware that the video URLs in the get_video_info response expire after a random(?) amount of time, so caching that file probably isn't ideal.</p>

<p>So that's how I got YouTube videos playing in WebGL! I'm working on tidying up my source code, and will put the github link up here when its ready for public consumption, but in the mean time if anybody has any feedback, please comment or <a href="mailto:vikki.read@gmail.com">talk to me</a>. Thanks!</p>

<a name="footnote1"></a>
<p class="footnote">I appreciate "3D squares" sounds ridiculous, but do the tutorials, it'll all make sense. I promise.</p>

