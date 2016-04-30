---
layout: post
title:  "Shadow cast, one letter bug and the 6 hours adventure."
date:   2016-04-25 16:11:16
categories: blog
comments: true
---

[learnopengl]:      http://learnopengl.com/#!Advanced-Lighting/Shadows/Shadow-Mapping
[NSight]: https://developer.nvidia.com/nvidia-nsight-visual-studio-edition



Few days ago, after two weeks of break, I've decided to add to mine engine a shadow cast (shadow mapping) mechanism. At first, I'v done some research on the subject and found that the solution is even simpler then expected. With forward rendering algorithm goes as follows:

**1.** Create an Frame Buffer (FBO that contains a texture buffer to be filled with depht values.

**2.** Render the scene to the FBO from the litght perspective, filling only the depth buffer.

**3.** Render the sce to the screen:

a. for each fragment, by using the light view matrix, check it's distance from the light.

b. if the depth of the fragment, seen from light perspecive is bigger than the fragment from previously created FBO, then the fragment is in shadow.

c. use some technics to enhance the overall look (light bias, PCF).

If you want to go deeper into the algorightm, I recommend you to visit [learnopengl][this site]. Here I want to focus on some bug, wich left me debuging it the whole night.

<h2>Bug </h2>

I've decided to implement the algorithm for one directional light. After coding for few hours methods to set up the light and the module for first render pass everithing looked nice in the unit tests. The shader code, nothing to fancy. Now, I should see nice shadows on my droid army, right?

<img src="/assets/img/Blog/post2_blackArmy.png" height="400" width="600">

Hmm, it looks like the shadow has covered almost the whole army and the first row is probably very near to the close plane of, the light orthogonal projection view. Pretty odd.

Firstly, I've checked if the z-buffer looked right in the first path. For debugging the GPU states, I've used a great tool called [NSight][NSight] by Nvidia. 


<img src="/assets/img/Blog/pos2_Zbuffor.png " >


Z-buffor looks preety smooth. So maybe the second pass messes up the texture and the comparision fails.

<img src="/assets/img/Blog/post2_Z2.png">

It's fine. So maybe the projection matrix of the light is wrong, so the depth in the second path of all fragments result in 1.0.

{% highlight ruby %}
	if(projCoords.z >= 1.0)
        shadow = 0.0;
{% endhighlight %}

```ruby
def foo
  puts 'foo'
end
```ruby

This two lines should ensure that if the tested fragment has the z value of 1.0, it's lighted. But the "army" is still black.

So to check if the orthogonal projection light matrix is properly created, I've decreased the camera far plane distance for first pass by a half. The result:


<img src="/assets/img/Blog/halfortho.png"  height="400" width="600">

The army is half lighted as expected and the matrix looks good. Ok, so to see how the comparision goes lets try to texture the soldiers with coresponding fragments from the FBO.

The vertex shader:

{% highlight ruby %}
void main()
{
	gl_Position = lightSpaceMatrix * model * vec4(position, 1.0f);
	vs_out.fragPosition = vec3(model * vec4(position, 1.0f));
	vs_out.Normal = mat3(transpose(inverse(model))) * normal;
	vs_out.TexCoords = texCoords;

}
{% endhighlight %}

Te fragment shader:

{% highlight ruby %}
void main()
{
	float x=gl_FragCoord.x/1200.0f;
	float y=gl_FragCoord.y/800.0f;
    
	float depthValue = texture(shadowMap, vec2(x,y)).r;
    color = vec4(vec3(depthValue), 1.0);
}
{% endhighlight %}

The further the soldier is from the light, the more bright it should be. Just like in the buffer image. Suprisingly I saw this:

<img src="/assets/img/Blog/textDeph.png"  height="400" width="600">


Here, I've assumed that, it's probably because mapping fragments between passes is wrong, so the texturing is disorted. This assumption was my biggest mistake. It made me spend few hours debugging the maping problem, when the real cause was much simpler.

{% highlight ruby %}
for (GLuint i = 0; i < textures.size(); i++)
{
	const OGLTextureBuffer &tBuffer = static_cast <const OGLTextureBuffer&>(*textures[i]->texturBuffer);
	glActiveTexture(GL_TEXTURE0 + i); // Activate proper texture unit before binding
	// Retrieve texture number (the N in diffuse_textureN)
	string name;
	switch (textures[i]->type)
	{
	case TextureType_DIFFUSE:
		name = "texture_diffuse"+to_string(diffuseNr++);
		break;
	case TextureType_SPECULAR:
		name = "texture_specular"+to_string(specularNr++);
		break;
	}

	glUniform1f(glGetUniformLocation(shader.Program, ("material." + name).c_str()), i );
	glBindTexture(GL_TEXTURE_2D, tBuffer.id);
}

glActiveTexture(GL_TEXTURE0 + textures.size());
glUniform1f(glGetUniformLocation(shader.Program, "shadowMap"), textures.size());
glBindTexture(GL_TEXTURE_2D, shadowMap.id);
{% endhighlight %}

In the snippet above, besides my shadow map texture, I pass to the shader other model textures. Everything looks right, but... instead of using *glUniform1f* maping uniform sampler to  the active texture, I should use *glUniform1i*. The handle is an integer type, not a float type. This little one letter error resulted in not sending this handle at all. Every uniform sampler was initialize as 0. This error was in my engine all the time, but it wasn't visible, as I don't use more than one diffuse texture and no specular texture, for models in the scene. I should have tested the texture asigned earlier. The texture passed at *GL_TEXTURE0* was the suit texture, which is black-gray-white, which I mistakely took as deformed passed shadow map. The final look should be more like this:

<img src="/assets/img/Blog/pos2_final.png"  height="400" width="600">

In conclusion, one letter bug and wrong assumptions made me crunch the night. It was frustrating but, sill learned a lot so no regrets.

Cheers.

