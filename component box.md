```python
# component box

// Parameters
box_length = 130;
box_width = 60;
box_height = 60;
wall_thickness = 2;

fiber_hole_diameter = 9;
led_hole_diameter = 6;

$fn = 80; // smooth circles

// Open box with hollow inside and open top
module open_box_open_top() {
    difference() {
        // Outer box
        cube([box_length, box_width, box_height]);

        // Hollow inside
        translate([wall_thickness, wall_thickness, wall_thickness])
            cube([
                box_length - 2*wall_thickness,
                box_width - 2*wall_thickness,
                box_height - wall_thickness
            ]);

        // Remove top face for open box
        translate([0, 0, box_height - wall_thickness])
            cube([box_length, box_width, wall_thickness + 1]);
    }
}

// Fiber hole fully drilled through left wall, extends outside the box by 3mm
module fiber_hole() {
    translate([-3, box_width/2, box_height/2])
        rotate([0,90,0])
        cylinder(h=wall_thickness + 6, d=fiber_hole_diameter);
}

// LED hole fully drilled through right wall, extends outside the box by 3mm
module led_hole() {
    translate([box_length + 3, box_width/2, box_height/2])
        rotate([180,90,0])
        cylinder(h=wall_thickness + 6, d=led_hole_diameter);
}

// Final model: box open at top with fully drilled holes extending outside
difference() {
    open_box_open_top();
    fiber_hole();
    led_hole();
}

```
