---
layout: post
title:  "Shadow cast, one letter error and the 6h adventure"
date:   2016-04-30 16:01:00
categories: blog
comments: true
---

[learnopengl]:      http://learnopengl.com/#!Advanced-Lighting/Shadows/Shadow-Mapping


Introduction
Few days ago, after two weeks break, I've decided to add to mine engine a shadow cast (shadow mapping) mechanism. At first I'v done some research on the subject and found that solution is even simpler then expected. With forward rendering algorithm go as follows:
1. Create an FBO that contains a texture buffer to represent a depth buffer
2. Render the scene to the Frame Buffer (FBO) from the litght perspective, filling only the depth buffer
3. Render the to the screen
	a. for each fragment, by using the light view matrix, check it's depth.
	b. if the depth of the fragment, seen from light perspecive is bigger then the fragment from previously created FBO then the fregmen is in shadow.
	c. use some technics to enhance the overall look (light bias, PCF)
If you want to know to know some more things about the algorightm I recommend you to visit [learnopengl][this site]. I want here to focus on some bug wich left me debuging it the whole night.

The Bug
For the moment I've decided to implement the algorithm for one directional light. After coding for thew hours the proper mechanics to set up the light, the module for first render pass everithing looked nice in the unit test. The the shader code, nothig to fancy. Now I should see nice shadows on my droid army right?

![black army](../../assets/img/Blog/post2_blackArmy.png)

Hmm, it looks like the shadow has covered almost the whole army and the first row is probably very near to the close plane of the light orthogonal projection. Preety odd.

Cheers.

