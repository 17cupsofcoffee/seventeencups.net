+++
title = "Introducing Tetra"
date = 2018-12-02

[taxonomies]
tags = ["rust", "gamedev"]
+++

I'm extremely excited to announce the release of my first public Rust crate! [Tetra](https://github.com/17cupsofcoffee/tetra) is a 2D game framework, primarily inspired by the likes of XNA, MonoGame and Raylib.

It's got a lot in common with some of the other Rust game engines ([GGEZ](https://github.com/ggez/ggez) especially was a big inspiration with regards to how to structure the API), but some of the nice features are:

* Draw call batching by default
* Deterministic game loop by default (à la [Fix Your Timestep](https://gafferongames.com/post/fix_your_timestep/))
* Simplified input handling (an imperative layer over the top of SDL's events)
* Pixel-perfect screen scaling (for those chunky retro pixels)

They say a code sample is worth a thousand words (or something like that...), so here's one of the examples from the docs:

```rust
extern crate tetra;

use tetra::error::Result;
use tetra::glm::Vec2;
use tetra::graphics::{self, Color, DrawParams, Texture};
use tetra::input::{self, Key};
use tetra::{Context, ContextBuilder, State};

struct GameState {
    texture: Texture,
    position: Vec2,
}

impl State for GameState {
    fn update(&mut self, ctx: &mut Context) {
        if input::is_key_down(ctx, Key::A) {
            self.position.x -= 2.0;
        }

        if input::is_key_down(ctx, Key::D) {
            self.position.x += 2.0;
        }

        if input::is_key_down(ctx, Key::W) {
            self.position.y -= 2.0;
        }

        if input::is_key_down(ctx, Key::S) {
            self.position.y += 2.0;
        }
    }

    fn draw(&mut self, ctx: &mut Context, _dt: f64) {
        graphics::clear(ctx, Color::rgb(0.769, 0.812, 0.631));

        graphics::draw(
            ctx,
            &self.texture,
            DrawParams::new()
                .position(self.position)
                .origin(Vec2::new(8.0, 8.0)),
        );
    }
}

fn main() -> Result {
    let ctx = &mut ContextBuilder::new()
        .title("Rendering a Texture")
        .size(160, 144)
        .scale(4)
        .resizable(true)
        .quit_on_escape(true)
        .build()?;

    let state = &mut GameState {
        texture: Texture::new(ctx, "./examples/resources/player.png")?,
        position: Vec2::new(160.0 / 2.0, 144.0 / 2.0),
    };

    tetra::run(ctx, state)
}
```

The API is likely to evolve a lot over the first few versions as I actually start using the framework for my own projects, but hopefully that's enough to give you a gist of whether or not you'll like Tetra. If you're interested, please give it a try, and let me know what you think on [Twitter](https://twitter.com/17cupsofcoffee) or [Discord](https://bit.ly/rust-community).