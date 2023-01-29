
## Introducing job systems

- Allow for a task-based parallelism model. <!-- .element: class="fragment" -->
- Dependencies required to be specified beforehand. <!-- .element: class="fragment" -->
- tasks + dependencies -> task graph <!-- .element: class="fragment" -->
- Handle synchronization and ordering internally. <!-- .element: class="fragment" -->
- Can use any parallel computing model: threads, fibers, coroutines, you name it. <!-- .element: class="fragment" -->

<end-page>

## Adding a job system to our game

1. Identify dependencies between each function call. <!-- .element: class="fragment" -->
2. Group serial function calls into a single job. <!-- .element: class="fragment" -->
3. Submit jobs to the job system and let it do its magic. <!-- .element: class="fragment" -->

<end-page>


## Defining the job system

- Regular job: just does what you say to it (jobs::create). <!-- .element: class="fragment" -->
- Serial job: a job composed of two or more jobs that should be run in serial (jobs::serial). <!-- .element: class="fragment" -->
- Parallel job: a job composed of two or more jobs that should be run in parallel (jobs::parallel). <!-- .element: class="fragment" -->
- Executor: helper type used to run the jobs. <!-- .element: class="fragment" -->

<end-page>

## Identifying the jobs

```cpp [|5-7|9-10|12-13|15-17|19-26|28-29]
while (game.is_running())
{
    float dt = timer.get_elapsed_time();

    // Block 1. Update objects
    for (auto &object : game.objects())
        object.update(dt);

    // Block 2. Physics
    physics.step(dt);

    // Block 3. Visual Effects
    particles.update();

    // Block 4. Animations
    for (auto &object : game.objects())
        object.animate();

    // Block 5. Rendering
    game.clear_screen();

    for (auto &object : game.objects())
        object.render();

    particles.render();
    ui.render();

    // Block 6. Telling the GPU what to render
    game.display_screen();
}
```

<end-page>

## Integrating the job system

```cpp [|4-7|9|10|12-20|22-30|32-37|39,44|43,47]
float dt = 0.0f;

// the new jobs
auto objects_update_job = jobs::create([&] {
    for (auto &object : game.objects())
        object.update(dt);
});

auto physics_job = jobs::create([&] { physics.step(dt); });
auto vfx_job = jobs::create([&] { particles.update(dt); });

auto animation_job = [&] {
    // spawn one job for every object to be animated
    auto animation_jobs = game.objects()
        | std::views::transform([&](auto &object) {
            return jobs::create([&] { object.animate(); });
        });

    return jobs::parallel(animation_jobs);
}();

auto rendering_job = jobs::create([&] {
    game.clear_screen();

    for (auto &object : game.objects())
        object.render();

    particles.render();
    ui.render();
});

auto frame = jobs::serial(
    objects_update_job,
    physics_job,
    jobs::parallel(vfx_job, animation_job),
    rendering_job
);

job_executor executor; // used to run the tasks on the graph

while (game.is_running())
{
    dt = timer.get_elapsed_time();
    executor.run(frame); // run all the jobs, wait for them to finish

    game.display_screen();
    timer.reset();
}
```

<end-page>

## Benefits

- More efficient usage of available cores. <!-- .element: class="fragment" -->
- No explicit synchronization. <!-- .element: class="fragment" -->
- Can we do better? <!-- .element: class="fragment" -->
