Files are here on my drive:https://drive.google.com/drive/folders/1kQMFwQMJ-v_3vh076vNgN4zUj689AiD4?usp=sharing

On the preview, I feel the STEP one has best rendering...

A preview of the files can be seen here:

STEP: https://jlc3dp.com/forface3dPreview.html?params=eyJtb2RlbFVybCI6Imh0dHBzOi8vYXBpLmZvcmZhY2UzZC5uZXQvZm9yZmFjZS9idWlsZGVyL2ZvcmZhY2VNb2RlbFJlc291cmNlT3ZlcnNlYXMvMjAyNDA4MjAwNzMyNTJyYXEyL01ldGFkYXRhLmYzZD9maWxlQWNjZXNzSWQ9ODUxNDEzOTE1NjIyMTI5NjY0MCZmVHlwZT1mM2QifQ==

OBJ: https://jlc3dp.com/forface3dPreview.html?params=eyJtb2RlbFVybCI6Imh0dHBzOi8vY2FydC5qbGNwY2IuY29tL3RkcEZpbGUvZG93bmxvYWRUZHBGaWxlP2ZpbGVBY2Nlc3NJZD04NTE0MTM5MTcxMzE2ODY3MDcyJmZUeXBlPW9iaiJ9

STL: https://jlc3dp.com/forface3dPreview.html?params=eyJtb2RlbFVybCI6Imh0dHBzOi8vY2FydC5qbGNwY2IuY29tL3RkcEZpbGUvZG93bmxvYWRUZHBGaWxlP2ZpbGVBY2Nlc3NJZD04NTE0MTM5MTU0NjI3NDYxMTIwJmZUeXBlPXN0bCJ9

3MF: https://jlc3dp.com/forface3dPreview.html?params=eyJtb2RlbFVybCI6Imh0dHBzOi8vY2FydC5qbGNwY2IuY29tL3RkcEZpbGUvZG93bmxvYWRUZHBGaWxlP2ZpbGVBY2Nlc3NJZD04NTE0MTM5MTYwMDk3MjM5MDQwJmZUeXBlPTNtZiJ9

Description... for my old brain...

This is a 3 step Duesenberg style pickup ring that matches the outer dimensions of a Schecter Ultra III and the inner dimensions of regular TV Jones humbuckers.

It has been made with FreeCAD 0.2.1.2

Total height is 4 mm
Base level height is 2 mm
Second level height is 1 mm
Last level height is 1 mm
Outer dimensions are 94 mm*44 mm
Inner dimensions are 72 mm*35 mm
Screw holes diameter is 3 mm
Countersink screw holes diameter is 5.5 mm
Distance between mounting screw holes on length is 85 mm
Distance between mounting screw holes in width is 36 mm
Distance between pickup height adjustments screw holes is 78.5 mm
Mounting screw holes countersink are chamfered at 1 mm
Base level edges are filleted at 0.5 mm
Second level edges are filleted at 0.25 mm
Last level edges are filleted at 0.1 mm


Oh and here is the Python code I used to start the shape, but beware...
- Some (obvious) artefacts still need to be defeatured
- Chamfer of the mounting screw holes is not done
- Filleting of the edges is not done either.

# Code starts here...
import FreeCAD as App
import Part

# Create a new document
doc = App.newDocument("Pickup Ring v2")

# Define outer dimensions (in millimeters)
outer_length = 94    # Outer length (bottom level)
outer_width = 44     # Outer width (bottom level)
inner_length = 72    # Inner length (final level before cutout)
inner_width = 35     # Inner width (final level before cutout)

# Number of levels + 1 extra level
levels = 4

# Define the thickness for each level
thickness_levels = [2, 1, 1, 1]  # 2mm for the first level, 1mm for the subsequent levels

# Calculate step dimensions
length_step = (outer_length - inner_length) / (2 * (levels - 1))
width_step = (outer_width - inner_width) / (2 * (levels - 1))

# Create the base layer
layers = []
for i in range(levels):
    length = outer_length - 2 * i * length_step
    width = outer_width - 2 * i * width_step
    thickness = thickness_levels[i]
    layer = Part.makeBox(length, width, thickness)
    layer.translate(App.Vector(i * length_step, i * width_step, sum(thickness_levels[:i])))
    layers.append(layer)

# Apply fillets (rounded corners) to each layer before combining
corner_fillet_radius = 1.5  # Fillet radius for the corners

# Apply fillets to each layer
for i, layer in enumerate(layers):
    edges_to_fillet = [
        layer.Edges[0],  # L side edge
        layer.Edges[2],  # R side edge
        layer.Edges[4],  # B side edge
        layer.Edges[6],  # U side edge
    ]
    for edge in edges_to_fillet:
        try:
            layers[i] = layers[i].makeFillet(corner_fillet_radius, [edge])
        except Part.OCCError:
            pass

# Combine all layers into one solid
pickup_ring = layers[0]
for layer in layers[1:]:
    pickup_ring = pickup_ring.fuse(layer)

# Create the rectangular inner cutout with rounded corners
inner_hole = Part.makeBox(inner_length, inner_width, sum(thickness_levels))
inner_hole.translate(App.Vector(
    (outer_length - inner_length) / 2,
    (outer_width - inner_width) / 2,
    0))

# Apply smaller fillets (rounded corners) to the inner hole's edges before cutting
inner_hole_edges = inner_hole.Edges
inner_fillet_radius = 1.5  # Smaller fillet radius for the inner hole corners
inner_hole = inner_hole.makeFillet(inner_fillet_radius, inner_hole_edges)

# Cut the inner hole from the pickup ring
pickup_ring = pickup_ring.cut(inner_hole)

# Recompute to apply the changes
doc.recompute()

# Add the screw holes
mounting_hole_radius = 1.5  # Radius of mounting screw holes
pickup_height_hole_radius = 1.5  # Radius of pickup height adjustment screw holes
countersink_diameter = 5.5     # Diameter of countersink
countersink_depth = 3        # Depth of countersink

# Mounting screw hole positions
mounting_positions = [
    (3, 3),
    (outer_length - 3, 3),
    (3, outer_width - 3),
    (outer_length - 3, outer_width - 3)
]

# Pickup height adjustment screw hole positions (centered)
pickup_height_positions = [
    (outer_length / 2 - 39.25, outer_width / 2),
    (outer_length / 2 + 39.25, outer_width / 2)
]

# Add mounting screw holes
for pos in mounting_positions:
    hole = Part.makeCylinder(mounting_hole_radius, sum(thickness_levels))
    hole.translate(App.Vector(pos[0], pos[1], 0))
    pickup_ring = pickup_ring.cut(hole)

# Add pickup height adjustment screw holes
for pos in pickup_height_positions:
    hole = Part.makeCylinder(pickup_height_hole_radius, sum(thickness_levels))
    hole.translate(App.Vector(pos[0], pos[1], 0))
    pickup_ring = pickup_ring.cut(hole)

# Add countersinks for mounting screws
for pos in mounting_positions:
    countersink = Part.makeCylinder(countersink_diameter / 2, countersink_depth)
    countersink.translate(App.Vector(pos[0], pos[1], sum(thickness_levels) - countersink_depth))
    pickup_ring = pickup_ring.cut(countersink)

# Add countersinks for pickup height adjustment screws
for pos in pickup_height_positions:
    countersink = Part.makeCylinder(countersink_diameter / 2, countersink_depth)
    countersink.translate(App.Vector(pos[0], pos[1], sum(thickness_levels) - countersink_depth))
    pickup_ring = pickup_ring.cut(countersink)

# Recompute to apply the changes
doc.recompute()

# Create a box that represents the material above 4mm
cut_box = Part.makeBox(outer_length, outer_width, sum(thickness_levels) - 4)
cut_box.translate(App.Vector(0, 0, 4))

# Subtract the box from the pickup ring to remove any excess material above 4mm
pickup_ring = pickup_ring.cut(cut_box)

# Recompute to apply the changes
doc.recompute()

# Add the final shape to the document
Part.show(pickup_ring)
