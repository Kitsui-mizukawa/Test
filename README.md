```
import bpy
import math

# Clean scene
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete(use_global=False)

# Create curve profile
curve_data = bpy.data.curves.new(name="CanProfile", type='CURVE')
curve_data.dimensions = '2D'

spline = curve_data.splines.new(type='BEZIER')
spline.bezier_points.add(7)

# Define profile points (Z = height, X = radius)
points = [
    (0.0, 0.0, 0.0),   # bottom center
    (0.035, 0.0, 0.01),
    (0.033, 0.0, 0.02),
    (0.033, 0.0, 0.11),
    (0.032, 0.0, 0.115),
    (0.028, 0.0, 0.12),
    (0.025, 0.0, 0.125),
    (0.03, 0.0, 0.13),  # top lip
]

for i, p in enumerate(points):
    bp = spline.bezier_points[i]
    bp.co = p
    bp.handle_left_type = 'AUTO'
    bp.handle_right_type = 'AUTO'

# Create object
curve_obj = bpy.data.objects.new("CanProfileObj", curve_data)
bpy.context.collection.objects.link(curve_obj)

# Add Screw modifier (lathe)
screw = curve_obj.modifiers.new(name="Screw", type='SCREW')
screw.angle = math.radians(360)
screw.steps = 64
screw.render_steps = 128
screw.axis = 'Z'
screw.use_merge_vertices = True
screw.merge_threshold = 0.0001

# Convert to mesh
bpy.context.view_layer.objects.active = curve_obj
bpy.ops.object.convert(target='MESH')

# Add Solidify (thickness)
solidify = curve_obj.modifiers.new(name="Solidify", type='SOLIDIFY')
solidify.thickness = 0.0015
solidify.offset = 1

# Add Bevel (soft edges)
bevel = curve_obj.modifiers.new(name="Bevel", type='BEVEL')
bevel.width = 0.001
bevel.segments = 3

# Add Subdivision
subsurf = curve_obj.modifiers.new(name="Subdivision", type='SUBSURF')
subsurf.levels = 2

# Shade smooth
bpy.ops.object.shade_smooth()

# Add metal material
mat = bpy.data.materials.new(name="CanMaterial")
mat.use_nodes = True
bsdf = mat.node_tree.nodes["Principled BSDF"]
bsdf.inputs["Metallic"].default_value = 1.0
bsdf.inputs["Roughness"].default_value = 0.25

curve_obj.data.materials.append(mat)

# Optional: Add slight vertical stretch to match proportions
curve_obj.scale[2] = 1.2

# Apply scale
bpy.ops.object.transform_apply(scale=True)
