---
layout: post
status: publish
published: true
title: Getting started with 3D Elements in WPF
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 197
wordpress_url: http://statichippo.com/blog/archive/2009/12/08/getting-started-with-3d-elements-in-wpf.aspx
date: 2009-12-09 02:33:49.000000000 -05:00
categories:
- XAML
tags: []
comments: []
---

I was watching the video entitled [How Do I: Get started with 3D Elements in WPF](http://windowsclient.net/learn/video.aspx?v=293715) and was just completely thrown off by author, Todd Miranda’s, complete lack of visuals.  So I played around with his example until I better understood what was going on and I figured I’d share.  Please understand that I am not a XAML master and certainly not in 3D, so if you find any errors please let me know. 
  
The XAML that Todd starts off with is:
  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 90%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">   <div style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">     <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum1">   1:</span> <Page xmlns=<span style="color: #006080">"http://schemas.microsoft.com/winfx/2006/xaml/presentation"</span></pre></div></div>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum2">   2:</span>  xmlns:sys=<span style="color: #006080">"clr-namespace:System;assembly=mscorlib"</span> xmlns:x=<span style="color: #006080">"http://schemas.microsoft.com/winfx/2006/xaml"</span> ></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum3">   3:</span>   <grid /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum4">   4:</span>     <viewport3d /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum5">   5:</span>         <viewport3d.camera /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum6">   6:</span>             <PerspectiveCamera Position=<span style="color: #006080">"-50,20,15"</span> LookDirection=<span style="color: #006080">"50,-15,-10"</span> UpDirection=<span style="color: #006080">"0,0,1"</span> /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum7">   7:</span>         </pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum8">   8:</span>         <modelvisual3d /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum9">   9:</span>             <modelvisual3d.content /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum10">  10:</span>                 <model3dgroup /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum11">  11:</span>                     <DirectionalLight Color=<span style="color: #006080">"White"</span> Direction=<span style="color: #006080">"-1,-1,-3"</span> /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum12">  12:</span>                     <geometrymodel3d /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum13">  13:</span>                         <geometrymodel3d.geometry /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum14">  14:</span>                             <MeshGeometry3D Positions=<span style="color: #006080">"0,0,0 10,0,0 10,10,0 0,10,0 0,0,10 0,10,10"</span></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum15">  15:</span>                                         TriangleIndices=<span style="color: #006080">"0 1 3  1 2 3  0 4 3  4 5 3"</span> /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum16">  16:</span>                         </pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum17">  17:</span>                         <geometrymodel3d.material /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum18">  18:</span>                             <DiffuseMaterial Brush=<span style="color: #006080">"Red"</span> /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum19">  19:</span>                         </pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum20">  20:</span>                     </pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum21">  21:</span>                 </pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum22">  22:</span>             </pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum23">  23:</span>         </pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum24">  24:</span>     </pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum25">  25:</span>   </pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum26">  26:</span> </pre>
<!--CRLF-->



The output is:


[![xaml_3d_1](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_1_thumb.png "xaml_3d_1")](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_1.png) 

## Camera Perspective


First off, let’s play with the Camera perspective.  I think the best way to do this is first to define the different parts of the Camera Perspective, the most important being **Position** and **LookDirection**.


[![wpf_3d_grid_1](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/wpf_3d_grid_1_thumb.png "wpf_3d_grid_1")](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/wpf_3d_grid_1_2.png)


Take the grid above.  It’s hard to show a Z axis, so I made the font smaller to try to trick your eye into believing it’s farther away.

### Position


So the grid above is akin to a room, right?  You have a top & bottom, right & left, and depth – we’ll call Z0 Backward and Z50 Forward so that if you’re walking into the room you’re walking Forward, ok?


[![xaml_3d_2](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_2_thumb.png "xaml_3d_2")](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_2.png) 


Now here we have a video camera pointed at a box.  It’s way too close to the box, as you can see, since all you can see in the viewfinder is part of the box.  Well to get a better picture we have 3 directions to move the camera: forward/back, right/left, up/down:


[![xaml_3d_3](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_3_thumb.png "xaml_3d_3")](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_3.png) (This might be a Position of -5(Z), 25(X), and 25(Y))


 


So in this instance, I might want to move the camera backwards, like so:


[![xaml_3d_4](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_4_thumb.png "xaml_3d_4")](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_4.png) (This might be a position of –25,25,25)


Now notice how I moved the camera backwards and got the whole box in the viewfinder.  I’m farther away from the box so I can see the whole thing.


The same principle holds true for the X & Y axis:


[![xaml_3d_5](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_5_thumb.png "xaml_3d_5")](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_5.png)  Perhaps this is –25,5,25 -- I moved the camera too far to the right and only see the right most part of the box.


The above is all the job of the Position attribute.  But there’s another important element to how the camera picks up the image and that is the angle of the camera

### LookDirection


“Well my head went back and to the left.”














In WPF, just like in real life, once you have your camera setup **where** you want it, you also need to angle the camera to point in the direction of the object you want to see.  For instance, if you want to catch a right-hand side (from your view) shot of a box, what would you need to do?  You would place your camera to the right side of the box at a depth that was equal with the center of the box.  Then you would need to turn your camera to the left in order for the lens to face the box.


Step 1, move your camera to the side of the box


[![xaml_3d_6](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_6_thumb.png "xaml_3d_6")](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_6.png) 


Step 2, turn your camera to the left:


[![xaml_3d_7](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_7_thumb.png "xaml_3d_7")](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_7.png) 

### XAML


This post was actually a lot of work to write.  I think I’ll have to break up the other elements into other blog posts and write those at another time.  However, for the sake of completion, let’s fold these new concepts back into the XAML from before.


Suppose you want to look change the direction so that you’re facing the red shape head-on.  In other words, you want to look at the shape so that it looks just like a square (instead of an L shaped 3D shape).  You might change the XAML so that line 3 reads

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper" />
  <div style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet" />
    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum1">   1:</span> <PerspectiveCamera Position=<span style="color: #006080">"-50,0,15"</span> LookDirection=<span style="color: #006080">"50,0,-10"</span> UpDirection=<span style="color: #006080">"0,0,1"</span> /></pre>
<!--CRLF-->



The output looks like this:


[![xaml_3d_8](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_8_thumb.png "xaml_3d_8")](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/Gettingstartedwith3DElementsinWPF_F519/xaml_3d_8.png) 


Now don’t mind that it’s black on top – that has to do with the light source and is out of the context of this post.  But let’s just see how we got here.  Here are the two lines juxtaposed (I added spacing so that everything lines up):







<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper" />
  <div style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet" />
    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum1">   1:</span> <PerspectiveCamera Position=<span style="color: #006080">"-50,20,15"</span> LookDirection=<span style="color: #006080">"50,-15,-10"</span> UpDirection=<span style="color: #006080">"0,0,1"</span> /></pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum2">   2:</span> <PerspectiveCamera Position=<span style="color: #006080">"-50,0 ,15"</span> LookDirection=<span style="color: #006080">"50, 0 ,-10"</span> UpDirection=<span style="color: #006080">"0,0,1"</span> /></pre>
<!--CRLF-->






 


So first I changed the Position’s X to 0 from 20, so I’m moving the camera to the right.  Now if I didn’t change the LookDirection it would mean that the camera would be facing slightly downwards (-10)  and slightly to the left (-15), so you’re not looking at it straight on.  That’s why I dropped the X value of LookDirection up from –15 to 0.


Just keep in mind that it is _very_ easy to place the Postion and/or LookDirection of the camera so that the shape is completely out of view. 


Hopefully this helps put some of what Todd says in his video in context.
