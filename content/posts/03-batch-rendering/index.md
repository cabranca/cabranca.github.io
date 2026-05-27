---
title: "Batch Rendering"
date: 2026-05-26
draft: false
---

{{< lang-toggle >}}

{{% lang "en" %}}
The first renderer worked. It opened a window, drew a quad with a texture, and that was honestly enough to feel good about for a day. Then I started thinking about what it would take to draw a hundred quads — and realized I had a problem.
{{% /lang %}}

{{% lang "es" %}}
El primer renderer funcionaba. Abría una ventana, dibujaba un quad con textura, y con eso era suficiente para sentirse bien por un día. Después empecé a pensar en lo que haría falta para dibujar cien quads — y me di cuenta de que tenía un problema.
{{% /lang %}}

---

{{% lang "en" %}}
## The cost of one draw call per object

Every draw call has overhead. When you tell the GPU to draw something, the driver has to validate state, pack commands, and synchronize with the GPU — and that work happens every single time you call it. For a handful of objects you'll never notice. For a 2D scene with hundreds of sprites animating at once, calling `glDrawElements` per sprite is a fast way to tank your framerate.

This isn't a secret. It's one of the most well-known performance problems in real-time graphics, and the solution has a name: batch rendering. Instead of one draw call per object, you accumulate all the geometry into a single buffer on the CPU, then send it to the GPU in one shot.

Simple to describe. Less simple to implement correctly — at least the first time.
{{% /lang %}}

{{% lang "es" %}}
## El costo de un draw call por objeto

Cada draw call tiene overhead. Cuando le decís a la GPU que dibuje algo, el driver tiene que validar el estado, empaquetar comandos y sincronizarse con la GPU — y ese trabajo ocurre cada vez que lo llamás. Para unos pocos objetos nunca lo vas a notar. Para una escena 2D con cientos de sprites animándose al mismo tiempo, llamar a `glDrawElements` por cada sprite es una forma rápida de destruir el framerate.

Esto no es ningún secreto. Es uno de los problemas de performance más conocidos en gráficos en tiempo real, y la solución tiene nombre: batch rendering. En lugar de un draw call por objeto, acumulás toda la geometría en un solo buffer en la CPU, y después la mandás a la GPU de una vez.

Simple de describir. Menos simple de implementar bien — al menos la primera vez.
{{% /lang %}}

---

{{% lang "en" %}}
## What the vertex buffer actually is

Here's what I had to unlearn: I was thinking about drawing in terms of objects. "Draw this sprite. Now draw that one." That mental model maps naturally to object-oriented code, but it maps terribly to how the GPU actually works.

The GPU doesn't care about sprites. It cares about vertices. A vertex buffer is just an array of vertex data — position, color, texture coordinates, whatever attributes you pack into each vertex. A single quad is four vertices. Two hundred quads is eight hundred vertices. There's nothing stopping you from stuffing all of them into the same array before any draw call happens.

That was the shift. Stop thinking "draw an object" and start thinking "write vertices into a buffer."

```cpp
struct QuadVertex {
    glm::vec3 position;
    glm::vec4 color;
    glm::vec2 texCoord;
    float texIndex;
};
```

At startup, I allocate a large CPU-side array — enough for the maximum number of quads per batch. Every time `drawQuad` is called, nothing is sent to the GPU. Instead, four vertices are written into that array and a pointer is advanced.

```cpp
// Writing one corner of a quad into the buffer
m_QuadVertexBufferPtr->position = transform * quadVertexPositions[i];
m_QuadVertexBufferPtr->color = color;
m_QuadVertexBufferPtr->texCoord = texCoords[i];
m_QuadVertexBufferPtr->texIndex = texIndex;
m_QuadVertexBufferPtr++;
```

The index buffer follows the same logic and can be precomputed entirely. Every quad is two triangles, which means every quad contributes the same six indices — offset by four for each preceding quad. You fill it once at initialization and never touch it again.

```cpp
uint32_t offset = 0;
for (uint32_t i = 0; i < maxIndices; i += 6) {
    quadIndices[i + 0] = offset + 0;
    quadIndices[i + 1] = offset + 1;
    quadIndices[i + 2] = offset + 2;
    quadIndices[i + 3] = offset + 2;
    quadIndices[i + 4] = offset + 3;
    quadIndices[i + 5] = offset + 0;
    offset += 4;
}
```

At the end of the frame — or when the buffer is full — the accumulated data is uploaded to the GPU and drawn in a single call. That's the entire idea.

{{< video src="batch-100-quads.webm" alt="100 quads in a single draw call" >}}

Going from one draw call per sprite to one draw call per frame was the kind of result that makes you want to keep going. A hundred quads, a thousand quads — the draw call count stays at one. Seeing it in a profiler for the first time is unreasonably satisfying.

So I pushed it. Ten thousand quads, still a single draw call:

{{< video src="batch-10000-quads.webm" alt="10,000 quads in a single draw call, holding 50 FPS" >}}

It holds 50 FPS. It ain't much, but it's honest work.
{{% /lang %}}

{{% lang "es" %}}
## Lo que el vertex buffer realmente es

Esto es lo que tuve que desaprender: estaba pensando en dibujar en términos de objetos. "Dibujá este sprite. Ahora ese." Ese modelo mental mapea naturalmente al código orientado a objetos, pero mapea pésimo a cómo la GPU realmente funciona.

A la GPU no le importan los sprites. Le importan los vértices. Un vertex buffer es solo un array de datos de vértices — posición, color, coordenadas de textura, los atributos que le metés a cada vértice. Un quad son cuatro vértices. Doscientos quads son ochocientos vértices. No hay nada que te impida meter todos en el mismo array antes de que ocurra cualquier draw call.

Ese fue el cambio. Dejar de pensar "dibujar un objeto" y empezar a pensar "escribir vértices en un buffer."

```cpp
struct QuadVertex {
    glm::vec3 position;
    glm::vec4 color;
    glm::vec2 texCoord;
    float texIndex;
};
```

Al arrancar, reservo un array grande del lado de la CPU — suficiente para la cantidad máxima de quads por batch. Cada vez que se llama a `drawQuad`, nada se manda a la GPU. En cambio, se escriben cuatro vértices en ese array y se avanza un puntero.

```cpp
// Escribiendo una esquina de un quad en el buffer
m_QuadVertexBufferPtr->position = transform * quadVertexPositions[i];
m_QuadVertexBufferPtr->color = color;
m_QuadVertexBufferPtr->texCoord = texCoords[i];
m_QuadVertexBufferPtr->texIndex = texIndex;
m_QuadVertexBufferPtr++;
```

El index buffer sigue la misma lógica y se puede precomputar completamente. Cada quad son dos triángulos, lo que significa que cada quad contribuye los mismos seis índices — desplazados por cuatro por cada quad anterior. Lo llenás una sola vez durante la inicialización y nunca más lo tocás.

```cpp
uint32_t offset = 0;
for (uint32_t i = 0; i < maxIndices; i += 6) {
    quadIndices[i + 0] = offset + 0;
    quadIndices[i + 1] = offset + 1;
    quadIndices[i + 2] = offset + 2;
    quadIndices[i + 3] = offset + 2;
    quadIndices[i + 4] = offset + 3;
    quadIndices[i + 5] = offset + 0;
    offset += 4;
}
```

Al final del frame — o cuando el buffer se llena — los datos acumulados se suben a la GPU y se dibujan en un solo llamado. Esa es la idea completa.

{{< video src="batch-100-quads.webm" alt="100 quads en un solo draw call" >}}

Pasar de un draw call por sprite a un draw call por frame fue el tipo de resultado que te hace querer seguir. Cien quads, mil quads — la cantidad de draw calls se mantiene en uno. Verlo por primera vez en un profiler es ridículamente satisfactorio.

Así que lo empujé. Diez mil quads, todavía un solo draw call:

{{< video src="batch-10000-quads.webm" alt="10.000 quads en un solo draw call, manteniendo 50 FPS" >}}

Se mantiene en 50 FPS. No es mucho, pero es trabajo honesto.
{{% /lang %}}

---

{{% lang "en" %}}
## The same trick, applied to text

When I started integrating FreeType for text rendering, I almost made the same mistake again. FreeType renders individual glyphs — you give it a character, it gives you a bitmap. The natural impulse is to upload that bitmap and draw it immediately. One character, one draw call.

But a glyph is just a textured quad. A word is a row of textured quads. The font atlas — where all glyphs live baked into a single texture — exists precisely so that all of them can be drawn from the same texture in the same draw call. Once I recognized the pattern, it wasn't a new problem. It was the same problem.

The text renderer ended up sharing the same architecture as the 2D sprite renderer: a CPU-side buffer that fills up glyph by glyph, a shared font atlas texture, and a single flush at the end. The main difference was in the vertex data — instead of a sprite's texture coordinates, each glyph vertex carries the coordinates into the atlas where that character lives.

Hitting the same design pattern a second time was more valuable than hitting it the first time. The first time you learn a technique. The second time, you start to see the shape of the problems it solves.
{{% /lang %}}

{{% lang "es" %}}
## El mismo truco, aplicado al texto

Cuando empecé a integrar FreeType para el renderizado de texto, casi cometí el mismo error de nuevo. FreeType renderiza glifos individuales — le das un carácter, te devuelve un bitmap. El impulso natural es subir ese bitmap y dibujarlo de inmediato. Un carácter, un draw call.

Pero un glifo es solo un quad con textura. Una palabra es una fila de quads con textura. El font atlas — donde todos los glifos viven horneados en una sola textura — existe precisamente para que todos puedan dibujarse desde la misma textura en el mismo draw call. Una vez que reconocí el patrón, no era un problema nuevo. Era el mismo problema.

El text renderer terminó compartiendo la misma arquitectura que el renderer 2D de sprites: un buffer del lado de la CPU que se llena glifo por glifo, una textura de font atlas compartida y un solo flush al final. La diferencia principal estaba en los datos de vértices — en lugar de coordenadas de textura de un sprite, cada vértice de glifo lleva las coordenadas dentro del atlas donde vive ese carácter.

Encontrar el mismo patrón de diseño por segunda vez fue más valioso que encontrarlo la primera. La primera vez aprendés una técnica. La segunda, empezás a ver la forma de los problemas que resuelve.
{{% /lang %}}

---

{{% lang "en" %}}
*Next: Building a Math Library — why I wrote my own vector and matrix types instead of relying entirely on GLM, and what that taught me about the math I'd been using without fully understanding.*
{{% /lang %}}

{{% lang "es" %}}
*Siguiente: Construyendo una librería matemática — por qué escribí mis propios tipos de vectores y matrices en lugar de depender completamente de GLM, y qué me enseñó eso sobre la matemática que venía usando sin entender del todo.*
{{% /lang %}}
