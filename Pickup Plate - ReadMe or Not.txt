import FreeCAD as App
import Part

doc = App.newDocument("PickupSelectionPlate")

# Function to create a hexagon from points and extrude it
def create_hexagon(points, thickness):
    wire = Part.makePolygon(points + [points[0]])
    face = Part.Face(wire)
    solid = face.extrude(App.Vector(0, 0, thickness))
    return solid

# Layer 1 (L1)
points_L1 = [
    App.Vector(0, 0),
    App.Vector(-20, 21.8),
    App.Vector(0, 43.6),
    App.Vector(50, 43.6),
    App.Vector(70, 21.8),
    App.Vector(50, 0)
]
L1 = create_hexagon(points_L1, 2)

# Layer 2 (L2)
points_L2 = [
    App.Vector(3.5, 3.5),
    App.Vector(-13, 21.8),
    App.Vector(3.5, 40.1),
    App.Vector(46.5, 40.1),
    App.Vector(63, 21.8),
    App.Vector(46.5, 3.5)
]
L2 = create_hexagon(points_L2, 1)
L2.translate(App.Vector(0, 0, 2))  # Translate to stack on top of L1

# Layer 3 (L3)
points_L3 = [
    App.Vector(7, 7),
    App.Vector(-6, 21.8),
    App.Vector(7, 36.6),
    App.Vector(43, 36.6),
    App.Vector(56, 21.8),
    App.Vector(43, 7)
]
L3 = create_hexagon(points_L3, 1)
L3.translate(App.Vector(0, 0, 3))  # Translate to stack on top of L2

# Combine layers
pickup_plate = L1.fuse([L2, L3])

# Function to create screw and lever holes
def create_hole(x, y, diameter, plate, countersink_diameter=None, countersink_depth=2):
    hole = Part.makeCylinder(diameter / 2, plate.BoundBox.ZLength)
    hole.translate(App.Vector(x, y, 0))
    plate = plate.cut(hole)
    
    if countersink_diameter:
        countersink = Part.makeCylinder(countersink_diameter / 2, countersink_depth)
        countersink.translate(App.Vector(x, y, plate.BoundBox.ZLength - countersink_depth))  # Translate downward for the countersink
        plate = plate.cut(countersink)
    
    return plate

# Function to create rectangular lever holes
def create_lever_hole(x, y, width, height, plate):
    lever_hole = Part.makeBox(width, height, plate.BoundBox.ZLength)
    lever_hole.translate(App.Vector(x, y, 0))
    return plate.cut(lever_hole)

# Offset values to align HL13 with the center of the hexagon
offset_x = -0.45
offset_y = -0.23

# Adjusted screw hole and lever hole positions
screw_positions = [
    (5.23 + offset_x, 11.53 + offset_y),  # HS1
    (15.29 + offset_x, 11.53 + offset_y), # HS2
    (25.35 + offset_x, 11.53 + offset_y), # HS3
    (35.41 + offset_x, 11.53 + offset_y), # HS4
    (45.47 + offset_x, 11.53 + offset_y), # HS5
    (5.23 + offset_x, 32.53 + offset_y),  # HS6
    (15.29 + offset_x, 32.53 + offset_y), # HS7
    (25.35 + offset_x, 32.53 + offset_y), # HS8
    (35.41 + offset_x, 32.53 + offset_y), # HS9
    (45.47 + offset_x, 32.53 + offset_y), # HS10
    (-10, 21.8), # MH1
    (60, 21.8) # MH2
]

lever_positions = [
    (3.33 + offset_x, 15.03 + offset_y),  # HL11
    (13.39 + offset_x, 15.03 + offset_y), # HL12
    (23.45 + offset_x, 15.03 + offset_y), # HL13
    (33.51 + offset_x, 15.03 + offset_y), # HL14
    (43.57 + offset_x, 15.03 + offset_y)  # HL15
]

# Create screw and lever holes with adjusted positions
for pos in screw_positions:
    pickup_plate = create_hole(pos[0], pos[1], 3, pickup_plate, countersink_diameter=5)

for pos in lever_positions:
    pickup_plate = create_lever_hole(pos[0], pos[1], 4, 14, pickup_plate)

# Creating the rectangular cut on the bottom side
# Center of HL13 is now correctly at (25, 21.8)
rect_center_x = 25
rect_center_y = 21.8

# Updated dimensions of the rectangle
rect_width = 50
rect_height = 28

# Create the rectangular cutout
rect_cutout = Part.makeBox(rect_width, rect_height, 1)
rect_cutout.translate(App.Vector(rect_center_x - rect_width/2, rect_center_y - rect_height/2, 0))  # Align with the bottom side

# Subtract the rectangle from the pickup plate
pickup_plate = pickup_plate.cut(rect_cutout)

# Add the final object to the document
Part.show(pickup_plate)

doc.recompute()
