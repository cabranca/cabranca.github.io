---
title: "Why I Built a Game Engine"
date: 2026-05-09
draft: false
---

{{< lang-toggle >}}

{{% lang "en" %}}
There is a question I've been asked more than once since starting this project: *why would you build a game engine when Unity, Unreal, and Godot already exist?*

The short answer is that I wasn't trying to replace them. The longer answer is the whole reason this series exists.
{{% /lang %}}

{{% lang "es" %}}
Hay una pregunta que me han hecho más de una vez desde que empecé este proyecto: *¿por qué construirías un motor de videojuegos cuando Unity, Unreal y Godot ya existen?*

La respuesta corta es que no estaba tratando de reemplazarlos. La respuesta larga es la razón por la que existe esta serie.
{{% /lang %}}

---
{{% lang "en" %}}
## Before the engine

Like many others, it all started with video games. My passion since childhood, they were always a place of refuge, creativity, and incredible stories. So it was no surprise that one day I asked myself how these games I loved were made — that's when I discovered Unity and Unreal Engine.

However, I didn't dive straight into programming. I've always been very curious and had many interests, so when I finished high school I enrolled in Geology, then Physics, and finally Computer Science. Almost without realizing it, I took a programming workshop at my university and discovered I really enjoyed coding. Then one day I came back to video games.

I decided to start with Unity. I watched a tutorial on YouTube, and with what I learned I said "time to build something on my own." Being a bit obsessive, I had to start with Pong (obviously). I slowly scripted it together, drew some blocks in MS Paint, and added my own touch: a randomized RickRoll (very fashionable at the time).

After a while I got interested in C and C++ programming, so it didn't take long before I tried making a game without a licensed engine. I followed the LearnOpenGL tutorials (thank you for everything) until I felt brave enough to strike out on my own and make my first game. Naturally, I cloned Pong again — this time without the gags.

Finally I had a return to engines when I discovered Godot (and it's Argentine!). Once again I cloned Pong and even entered some Game Jams — all of them a lot of fun.

But something didn't fit. I wasn't fully enjoying the process of making a game — parts of it even bored me. That's when I understood that what I was passionate about was *creating* an engine, not using one. Building Pong in C++ had led me to make many decisions about data structures, architecture, memory management. That I enjoyed, that drew me in. I would later confirm it when designing and implementing an API for my engine. I love building programs for programmers and designers, not games for their users.

My mind was made up. I had to create Cabrankengine.
{{% /lang %}}

{{% lang "es" %}}
## Antes del motor

Como muchos otros, todo comenzó con los videojuegos. Mi pasión desde pequeño, siempre fueron un lugar de refugio, de creatividad y de historias increíbles. No fue extraño entonces que un día me preguntara cómo se hacían estos juegos que yo amaba. Ahí es cuando me enteré de la existencia de Unity y Unreal Engine.

Sin embargo, no me metí de lleno a programar. Siempre fui muy curioso y con muchos gustos, por lo que al terminar la secundaria me anoté en Geología, luego en Física, finalmente en Computación. Casi sin quererlo, hice un taller de programación en mi universidad y descubrí que me divertía mucho programar. Entonces un día volví a los videojuegos.

Decidí empezar con Unity. Me vi un tutorial en Youtube y con lo que aprendía dije "es hora de hacer algo solo", y como soy un poco obsesivo, tuve que empezar con Pong (obviamente). De a poco fui scripteando, creando bloques en MS Paint y para agregarle algo propio, un RickRoll randomizado (muy de moda en aquella época).

Luego de un tiempo empecé a interesarme por la programación en C y C++, por lo que no tardó mucho en llegar el momento de hacer un juego sin un motor licenciado. Fui siguiendo las indicaciones de LearnOpenGL (gracias por todo) hasta animarme a desviarme y realizar mi primer juego. Como no podía ser de otra manera, volví a clonar el Pong solo que esta vez sin gags.

Por último tuve una vuelta a los motores cuando conocí Godot (encima argentino!). Una vez más cloné el Pong e incluso participé de algunas Game Jams, todas muy divertidas.

Sin embargo, sentía que algo no encajaba. No disfrutaba del todo el proceso de hacer un juego y había partes que hasta me aburrían. Ahí fue cuando entendí que lo que me apasionaba era _crear_ un motor, no usarlo. Haciendo el Pong en C++ me había llevado a tomar muchas decisiones de estructuras de datos, de arquitectura, de manejo de memoria. Eso me gustaba, eso me atraía. Luego lo confirmaría al diseñar e implementar una API para mi motor. Me encanta hacer programas para programadores, para diseñadores, no hacer juegos para sus usuarios.

Lo tenía decidido. Tenía que crear Cabrankengine.
{{% /lang %}}

---

{{% lang "en" %}}
## What I was trying to learn

As soon as I started researching game engines, I understood I'd have to master many fields of computer science. I couldn't have been happier with the decision. If there's one thing I love, it's learning and building — and there's a lot of ground to cover here.

So I started asking myself a lot of questions to guide my path:

- How do I go from raw OpenGL calls to a clean, simple API for the user?
- How does an ECS integrate with an engine?
- How do you build a cross-platform application?
- How do you separate engine logic from gameplay logic?

These are just a few of them. Luckily the list never ends and there's always something new to learn.
{{% /lang %}}

{{% lang "es" %}}
## Lo que estaba intentando aprender

Apenas empecé a investigar sobre motores de videojuegos, entendí que tendría que dominar muchos campos de las ciencias de la computación. No podría haber estado más contento con la decisión. Si hay algo que me gusta es aprender y hacer, y acá hay mucha tela para cortar.

Es entonces que comencé a hacerme muchas preguntas para guiarme en mi camino:

- ¿Cómo paso de llamados a OpenGL a una API limpia y simple para el usuario?
- ¿Cómo se integra un ECS a un motor?
- ¿Cómo se realiza una aplicación multiplataforma?
- ¿Cómo se separa la lógica del motor con la lógica de gameplay?

Estas son solo algunas de las preguntas. Por suerte la lista nunca acaba y siempre hay algo nuevo que aprender.
{{% /lang %}}

---

{{% lang "en" %}}
## What it became

What started as a learning exercise turned into something I'm genuinely proud of. Cabrankengine today covers:

- A full 2D batch renderer that draws hundreds of sprites in a single draw call
- A 3D rendering pipeline with Phong and physically-based rendering (PBR), supporting roughness maps, metalness maps, normal maps, and HDR lighting
- A custom Entity Component System built around data-oriented design — Registry, typed component arrays, signature-based system dispatch, up to 10,000 concurrent entities
- A custom binary asset format (`.cbkm` for models, `.cbkt` for textures), with a CLI converter tool that takes Blender exports and produces compact, engine-ready files
- A WebAssembly compilation target, so the engine can run in the browser
- A macOS Metal backend alongside the primary OpenGL one

None of that was planned from day one. It grew because every answer to "how does this work?" led to three more questions, and I found I wanted to answer those too.
{{% /lang %}}

{{% lang "es" %}}
## En lo que se convirtió

Lo que comenzó como un ejercicio de aprendizaje se convirtió en algo de lo que estoy genuinamente orgulloso. Cabrankengine hoy incluye:

- Un batch renderer 2D completo que dibuja cientos de sprites en un solo draw call
- Un pipeline de rendering 3D con Phong y physically-based rendering (PBR), con soporte para roughness maps, metalness maps, normal maps e iluminación HDR
- Un Entity Component System personalizado construido alrededor del diseño orientado a datos — Registry, arrays de componentes tipados, despacho de sistemas basado en signatures, hasta 10.000 entidades simultáneas
- Un formato de asset binario personalizado (`.cbkm` para modelos, `.cbkt` para texturas), con una herramienta CLI conversora que toma exports de Blender y produce archivos compactos y listos para el motor
- Un target de compilación a WebAssembly, para que el motor pueda correr en el navegador
- Un backend Metal para macOS además del backend OpenGL principal

Nada de eso estaba planeado desde el día uno. Fue creciendo porque cada respuesta a "¿cómo funciona esto?" llevaba a tres preguntas más, y descubrí que quería responderlas todas.
{{% /lang %}}

---

{{% lang "en" %}}
## Why document it now?

I decided to write these blog posts for several reasons:

- I like sharing what I do.
- I love talking about programming.
- It's always good to look back at the road you've traveled.
- Maybe it'll help someone — you never know.

This series will trace the development roughly in the order things were built. I'll try to be honest about what was hard, what I got wrong the first time, and what surprised me. This first post is the only one without code — every post after this will have something concrete to look at.
{{% /lang %}}

{{% lang "es" %}}
## ¿Por qué documentarlo ahora?

Decidí escribir estos blogs por varias razones:

- Me gusta compartir lo que hago.
- Me encanta debatir sobre programación.
- Siempre es bueno mirar atrás el camino recorrido.
- Capaz le sirva a alguien, nunca se sabe.

Esta serie va a seguir el desarrollo más o menos en el orden en que se fueron construyendo las cosas. Voy a tratar de ser honesto sobre qué fue difícil, qué hice mal la primera vez y qué me sorprendió. Este primer post es el único sin código — cada post después de este va a tener algo concreto para ver.
{{% /lang %}}

---

{{% lang "en" %}}
*Next: The Engine Foundation — how I got GLFW, an event system, and a layer stack in place before drawing a single pixel.*
{{% /lang %}}

{{% lang "es" %}}
*Siguiente: Los cimientos del motor — cómo puse en marcha GLFW, un sistema de eventos y una pila de capas antes de dibujar un solo píxel.*
{{% /lang %}}
