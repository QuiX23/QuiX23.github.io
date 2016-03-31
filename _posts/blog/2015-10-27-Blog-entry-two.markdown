---
layout: post
title:  "MVC pattern in game development"
date:   2016-03-28 18:11:16
categories: blog
---

[podcast]:      http://www.se-radio.net/2011/05/episode-175-game-development-with-andrew-brownsword/
[game loop pattern]: http://gameprogrammingpatterns.com/game-loop.html

Recently I was listening to a [podcast][podcast] with a Andrew Brownsword, a lead developer from EA Games, one of the creators of popular racing seriers- Need for Speed. He touched many interesting subjects, introducing the listeners with practices in "big boys" league. 

As I'm also taking a shot on creating my own 3D engine, I was mostly interested with issues concerning games engine architecture. Most of the time creating my engine I spend on figuering how should each modul and object comunicate, and how to create the most elegant dependencies, to keep the expected level of abstraction. 

Andrew mentioned spliting a simulation and rendering into separate parts. Simulation would be responsible for updating the world state, rendering for presenting it on screen, both audio and visuals. It isn't a suprising aproach, as it is used more or less by [game loop pattern][game loop pattern]. However, I'm trying to implement the game loop pattern, I don't have such a formal split between two modules. Andrew speech encourged me to try to achive that, so I could just control the data that passes between.

My goal will be to create some kind of a data structure, let's call it a RenderState. I can fill it by the simulation engine in the CPU and then, later on that structure would be handled by rendering module, prepared and passed to the GPU for presenting it on screen. So let's see, what do we need to pass, to render an 3D Object.

I think that we should firstly focus on shaders. It's mostly because, if the Objects (by Object I mean the buffer pointers representing it) we want to render have got the same shader, we can perform some batching magic in order to reduce the number of draw calls. So firstly in my structure, I would split Objects into groups, identified by shader they use. I could use a hash table to achive that. Then in the same structure we could put the global uniform value. 

So the flow would simple look like- the SceneManager after any change would update the state of the RenderState structure, like adding new Object, updating positions. Then when RenderState would be closed and finished changing, it would be passed to the Renderer, which would call specific OpenGL functions, in specific order- Shader.Use(), glUniform(...), n x glDraw(...) so the calls to underlying GPU driver layer would be limited to minimum. It's worth mantioning that possible future occlusion calculation would be handled from the CPU, by the simulation engine, when preparing RenderState structure. 
So how is it diffrenet form the current implementation? Well, now the SceneManager calls the glUniform method as it represents the global world state. It calls also Draw method on every object on scene, one by one, and we don't have clearly separated RenderState structure. This could lead us to breaking project modularity, by creating a big monster class, not sticking to the SOLID rule. So yeah, I can see some benefits.

All above is a highly estimated implementation plan, looking more as a collection of loose thoughts. If I will have enough time at the weekend, I will take a shot on making it real. Then I could throw some UML with the exact implementation. 

Cheers.

