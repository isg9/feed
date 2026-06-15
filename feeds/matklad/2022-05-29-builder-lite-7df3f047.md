---
title: Builder Lite
url: https://matklad.github.io/2022/05/29/builder-lite.html
published: "2022-05-29T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2022/05/29/builder-lite.html
---

# Builder Lite

May 29, 2022

In this short post, I describe and name a cousin of the builder pattern — builder lite.

Unlike a traditional builder, which uses a separate builder object, builder lite re-uses the object itself to provide builder functionality.

Here’s an illustrative example

Builder Lite

```
pub struct Shape {
  position: Vec3,
  geometry: Geometry,
  material: Option,
}

impl Shape {
  pub fn new(geometry: Geometry) -> Shape {
    Shape {
      position: Vec3::default(),
      geometry,
      material: None,
    }
  }

  pub fn with_position(mut self, position: Vec3) -> Shape {
    self.position = position;
    self
  }

  pub fn with_material(mut self, material: Material) -> Shape {
    self.material = Some(material);
    self
  }
}

// Call site

let shape = Shape::new(Geometry::Sphere::with_radius(1))
  .with_position(Vec3(0, 9, 2))
  .with_material(Material::SolidColor(Color::Red));
```

In contrast, the full builder is significantly wordier at the definition site, and requires a couple of extra invocations at the call site:

Builder

```
pub struct Shape {
  position: Vec3,
  geometry: Geometry,
  material: Option,
}

pub struct ShapeBuilder {
  position: Option,
  geometry: Option,
  texture: Option,
}

impl Shape {
  pub fn builder() -> ShapeBuilder { ... }
}

impl ShapeBuilder {
  pub fn position(&mut self, position: Vec3) -> &mut Self { ... }
  pub fn geometry(&mut self, geometry: Geometry) -> &mut Self { ... }
  pub fn material(&mut self, material: Material) -> &mut Self { ... }
  pub fn build(&self) -> Shape { ... }
}

// Call site

let shape = Shape::builder()
  .position(Vec3(9, 2))
  .geometry(Geometry::Sphere::with_radius(1))
  .material(Material::SolidColor(Color::Red))
  .build();
```

The primary benefit of builder-lite is that it is an incremental, zero-cost evolution from the `new` method. As such, it is especially useful in the context where the code evolves rapidly, in an uncertain direction. That is, when building applications rather than library.

To pull a motivational example from work, we had the following typical code:

```
impl PeerManagerActor {
  pub fn new(
    store: Store,
    config: NetworkConfig,
    client_addr: Recipient,
    view_client_addr: Recipient,
    routing_table_addr: Addr,
  ) -> anyhow::Result<Self> {
```

Here’s a `new` method with a whole bunch of arguments for various dependencies. What we needed to do is to add yet another dependency, so that it could be overwritten in tests. The first attempt just added one more parameter to the `new` method:

```
  pub fn new(
    store: Store,
    config: NetworkConfig,
    client_addr: Recipient,
    view_client_addr: Recipient,
    routing_table_addr: Addr,
+   ping_counter: Box<dyn PingCounter>,
  ) -> anyhow::Result<Self> {
```

However, this change required update of the seven call-sites where the `new` was called to supply the default counter. Switching that to builder lite allowed us to only modify a single call-site where we cared to override the counter.

A note on naming:

If builder methods are to be used only occasionally, `with_foo` is the best naming. If most call-sites make use of builder methods, just `.foo` might work better. For boolean properties, sometimes it makes sense to have both:

```
pub fn fancy(mut self) -> Self {
  self.with_fancy(true)
}

pub fn with_fancy(mut self, yes: bool) -> Self {
  self.fancy = yes;
  self
}
```

Discussion on [/r/rust](https://old.reddit.com/r/rust/comments/v07kac/blog_post_builder_lite/).
