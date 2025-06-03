```python
# mounted holders
// === Base Dimensions ===
base_length = 60;
base_width = 20;
base_height = 3;

// === Common Spacing ===
spacing = 12;  // Separation between ring centers

// === LED Holder ===
led_post = [6, 6, 24];
led_ring_inner_d = 6;
led_ring_thickness = 2;
led_ring_depth = 3;
led_ring_outer_d = led_ring_inner_d + 2 * led_ring_thickness;

// === Lens Holder ===
lens_post = [6, 6, 21];
lens_ring_inner_d = 13.5;
lens_ring_thickness = 2;
lens_ring_depth = 4;
lens_ring_outer_d = lens_ring_inner_d + 2 * lens_ring_thickness;

// === Fiber Holder ===
fiber_post = [6, 6, 22]; // You updated fiber post to 6×6×22
fiber_ring_inner_d = 9;
fiber_ring_thickness = 2;
fiber_ring_depth = 4;
fiber_ring_outer_d = fiber_ring_inner_d + 2 * fiber_ring_thickness;

// === Ring Centers Along X ===
led_center_x = 12;                    // 1st ring at 12mm
lens_center_x = led_center_x + 12;    // 2nd ring at 24mm
fiber_center_x = lens_center_x + 12;  // 3rd ring at 36mm

// === Center Y of Base ===
center_y = base_width / 2;

// === Module: Post with vertical ring facing along X ===
module post_with_ring_aligned(post_size, ring_inner_d, ring_thick, ring_depth) {
    post_length = post_size[0];
    post_width = post_size[1];
    post_height = post_size[2];
    ring_outer_d = ring_inner_d + 2 * ring_thick;

    // Post
    cube([post_length, post_width, post_height]);

    // Ring mounted vertically at top center, axis along X
    translate([post_length / 2, post_width / 2, post_height + ring_outer_d / 2])
        rotate([0, 90, 0])  // Rotate to make axis horizontal (along X)
        difference() {
            cylinder(h = ring_depth, d = ring_outer_d, center = true);
            cylinder(h = ring_depth + 0.1, d = ring_inner_d, center = true);
        }
}

// === Module: Assembled Base with All Holders ===
module common_base_aligned() {
    // Base platform
    color("lightgray")
    cube([base_length, base_width, base_height]);

    // LED Holder
    translate([
        led_center_x - led_post[0] / 2,
        center_y - led_post[1] / 2,
        base_height
    ])
    color("red")
    post_with_ring_aligned(led_post, led_ring_inner_d, led_ring_thickness, led_ring_depth);

    // Lens Holder
    translate([
        lens_center_x - lens_post[0] / 2,
        center_y - lens_post[1] / 2,
        base_height
    ])
    color("blue")
    post_with_ring_aligned(lens_post, lens_ring_inner_d, lens_ring_thickness, lens_ring_depth);

    // Fiber Holder
    translate([
        fiber_center_x - fiber_post[0] / 2,
        center_y - fiber_post[1] / 2,
        base_height
    ])
    color("green")
    post_with_ring_aligned(fiber_post, fiber_ring_inner_d, fiber_ring_thickness, fiber_ring_depth);
}

// === Render it ===
common_base_aligned();


```


```python

```
