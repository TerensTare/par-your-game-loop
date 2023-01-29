
## Parallelizing part 2: Rendering

```cpp
game.clear_screen();

for (auto &object : game.objects())
    object.render();

particles.render();
ui.render();
```

- The rendering job seems kinda heavy. <!-- .element: class="fragment" -->
- But rendering objects/particles/UI don't depend on each other... <!-- .element: class="fragment" -->
- How can we parallelize that? 🤔 <!-- .element: class="fragment" -->

<end-page>

## Enter: Multithreaded rendering

- Common practice on modern APIs
    - In Vulkan and Metal, you can create multiple command buffers (ideally one per thread) and submit them to the GPU. <!-- .element: class="fragment" -->
    - DirectX 12 calls them graphics command lists. <!-- .element: class="fragment" --> 
    - DirectX 11 offers deferred rendering contexts. <!-- .element: class="fragment" -->

<end-page>

## Enter: Multithreaded rendering (cont.)

- Create one "execution unit" per thread. <!-- .element: class="fragment" -->
- Write rendering commands to it, then submit to the GPU. <!-- .element: class="fragment" -->
- Ideally we want to have one thread to render objects, one for particles and one for the UI. <!-- .element: class="fragment" -->
- No implementation shown as it is API-specific. <!-- .element: class="fragment" -->