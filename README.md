# Blender Export to GLB + ConvexShape

This Blender script automates exporting selected objects to GLB format and generates corresponding Convex Hull shape files.  
It is useful for game engines such as Defold, Unity, Godot, or any workflow where both mesh and convex collision data are required.


based on: https://github.com/selimanac/defold-blender-convex-hull
## Features

1. Apply Transforms  
   For each selected object, Apply All Transforms (location, rotation, scale) is executed.

2. Export to GLB  
   - Each object is exported as a separate .glb file.  
   - Export options:
     - GLB → single binary file (default, recommended).  
     - GLTF_SEPARATE → .gltf + .bin (if you want text-based .gltf).  
   - File name matches the object name in Blender.  
   - +Y Up is disabled (export_yup=False).  
   - All transforms are applied during export (export_apply=True).  

3. Generate Convex Hull  
   - For each object, a convex hull is calculated from its vertices.  
   - The data is saved in a .convexshape file next to the .glb.  
   - File format example:
    
     shape_type: TYPE_HULL
     data: 1.0
     data: 2.0
     data: 3.0
     ...
     
4. Multiple Object Support  
   - All selected objects are processed one by one.  
   - Errors with a single object do not stop the script.
   
5. Usage  

Copy and paste the script below into Blender Scripting. Specify the export path in the script. Select the objects to export and run the script.

Script tested on versions 4.5.3


```python
import bpy
import bmesh
import os

# === Export folder path ===
# Change this to your export folder
export_path = "/"

if not os.path.exists(export_path):
    raise FileNotFoundError(f"Export folder {export_path} does not exist")

# Get all selected objects
selected_objects = bpy.context.selected_objects
if not selected_objects:
    raise ValueError("No objects selected!")

# Apply transforms for all selected objects
for obj in selected_objects:
    bpy.context.view_layer.objects.active = obj
    obj.select_set(True)
    bpy.ops.object.transform_apply(location=True, rotation=True, scale=True)
    obj.select_set(False)


def export_convex_hull_points(obj, filepath):
    """Generate convex hull for the object and save vertices to file."""
    bm = bmesh.new()
    bm.from_mesh(obj.data)

    # Build convex hull
    bmesh.ops.convex_hull(bm, input=bm.verts)

    # Extract vertices
    vertices = [v.co.copy() for v in bm.verts]

    # Write to file
    with open(filepath, 'w') as f:
        f.write("shape_type: TYPE_HULL\n")
        for v in vertices:
            f.write(f"data: {v.x}\n")
            f.write(f"data: {v.y}\n")
            f.write(f"data: {v.z}\n")

    bm.free()


# Export each selected object
for obj in selected_objects:
    try:
        obj.select_set(True)
        bpy.context.view_layer.objects.active = obj

        # File names
        glb_file = os.path.join(export_path, f"{obj.name}.glb")
        convex_file = os.path.join(export_path, f"{obj.name}.convexshape")

        # Export GLB (one file per object)
        print(f"Exporting object: {obj.name} -> {glb_file}")
        bpy.ops.export_scene.gltf(
            filepath=glb_file,
            export_format='GLB',  # Single binary file
            use_selection=True,
            export_yup=False,     # Disable +Y Up
            export_apply=True
        )

        # Export convex hull points
        print(f"Generating convex hull for {obj.name} -> {convex_file}")
        export_convex_hull_points(obj, convex_file)

        obj.select_set(False)

    except Exception as e:
        print(f"Error processing {obj.name}: {e}")
        obj.select_set(False)
        continue

print("Export finished!")

``` 
