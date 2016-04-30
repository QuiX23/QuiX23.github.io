---
layout: post
title:  "MVC pattern in game development"
date:   2016-03-28 18:11:16
categories: blog
comments: true
---

[podcast]:      http://www.se-radio.net/2011/05/episode-175-game-development-with-andrew-brownsword/
[game loop pattern]: http://gameprogrammingpatterns.com/game-loop.html


Recently I was listening to a [podcast][podcast] with Andrew Brownsword, a lead developer from EA Games, one of the creators of popular racing seriers- Need for Speed. He touched many interesting subjects, introducing the listeners with practices in "big boys" league. 

As I'm also taking a shot on creating my own 3D engine, I was mostly interested with issues concerning games engine architecture. Most of the time creating my engine I spend on figuering how should each modul and object comunicate, and how to create the most elegant dependencies, to keep the expected level of abstraction. 

Andrew mentioned spliting a simulation and rendering into separate parts. Simulation would be responsible for updating the world state, rendering for presenting it on screen, both audio and visuals. It isn't a suprising aproach, as it is used more or less by [game loop pattern][game loop pattern]. However, I'm trying to implement the game loop pattern, I don't have such a formal split between the two data types. 

As you see it is very similar to the calssic MVC pattern, but maybe it shuold be named SMR (Simulation Structure Renderer)pattern. The "Simulation" stands for controller part, "Renderer" stands for the view part and the "Structure" stands for the model part. The pattern makes the implementation very data-driven, which is great for number of reasons.

"Make the **Simulation**, update the data **Structure**, **Render** the structure"

I should mention, that using this pattern, you can pass data from rendering to the simulation, and then send it back, which would help to create some image post-processing effects or implement shadow mapping.

Andrew speech encourged me to try to achive that in my engine. If I will have enough time at the weekend, I will take a shot on making it real. Then I could throw some UML and code with the exact implementation. 



Cheers.

