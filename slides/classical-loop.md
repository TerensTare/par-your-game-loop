## Your classical game loop

```cpp
while (game.is_running())
{
    float dt = timer.get_elapsed_time();

    for (auto &object : game.objects())
        object.update(dt);

    physics.step(dt);

    particles.update(dt);

    for (auto &object : game.objects())
        object.animate();

    game.clear_screen();

    for (auto &object : game.objects())
        object.render();

    particles.render();
    ui.render();

    game.display_screen();
}
```

<end-page>


## Shortcomings

1. Serial <!-- .element: class="fragment" -->
    - Code executed linearly, causing unnecessary waits. <!-- .element: class="fragment" -->
    - Only one core used. <!-- .element: class="fragment" -->

2. Implicit <!-- .element: class="fragment" -->
    - Which function reads what? <!-- .element: class="fragment" -->
    - How about writing? <!-- .element: class="fragment" -->