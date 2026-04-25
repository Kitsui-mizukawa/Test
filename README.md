```
import bpy
import bmesh
from math import pi

def create_perfect_can_final():
    # 1. Bersihkan Scene
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

    # Parameter Geometri (dalam unit Blender/Meter)
    vertices_radial = 64
    can_radius_body = 0.5
    can_height_body = 1.85 # Bagian lurus
    
    can_radius_neck_final = 0.405 # Radius leher ramping
    can_height_neck_total = 0.23 # Tinggi transisi necking
    can_height_lip = 0.045 # Tinggi bibir paling atas
    
    can_depth_cap = -0.055 # Kedalaman cekungan tutup
    can_depth_dome = 0.11 # Kedalaman kubah bawah

    # 2. Buat Silinder Dasar DENGAN TUTUP (NGON) agar mudah dimodifikasi
    bpy.ops.mesh.primitive_cylinder_add(
        vertices=vertices_radial, 
        radius=can_radius_body, 
        depth=can_height_body, 
        location=(0, 0, can_height_body / 2),
        end_fill_type='NGON'
    )
    can = bpy.context.object
    can.name = "Kaleng_Aluminium_Realistis_Final"
    mesh = can.data

    # 3. Pindah ke Edit Mode untuk Geometri Kompleks
    bpy.ops.object.mode_set(mode='EDIT')
    bm = bmesh.from_edit_mesh(mesh)
    bm.faces.ensure_lookup_table()

    # Cari permukaan (face) paling atas dan paling bawah
    top_face = max(bm.faces, key=lambda f: f.calc_center_bounds().z)
    bottom_face = min(bm.faces, key=lambda f: f.calc_center_bounds().z)

    # --- Bagian Atas: Necking, Bibir, Tutup ---
    
    # Langkah 1: Penciutan Leher (Necking)
    # Gunakan ekstrusi 1-langkah lalu skala untuk transisi yang tajam tapi bisa dihaluskan via subsurf
    ret_neck = bmesh.ops.extrude_discrete_faces(bm, faces=[top_face])
    top_face = ret_neck['faces'][0]
    bmesh.ops.scale(bm, vec=(can_radius_neck_final / can_radius_body, can_radius_neck_final / can_radius_body, 1.0), verts=top_face.verts)
    bmesh.ops.translate(bm, vec=(0, 0, can_height_neck_total), verts=top_face.verts)

    # Langkah 2: Bibir Kaleng Tegak (Lip)
    ret_lip = bmesh.ops.extrude_discrete_faces(bm, faces=[top_face])
    top_face = ret_lip['faces'][0]
    bmesh.ops.translate(bm, vec=(0, 0, can_height_lip), verts=top_face.verts)

    # Langkah 3: Ketebalan Bibir Kaleng (Inset)
    ret_inset_top = bmesh.ops.inset_region(bm, faces=[top_face], thickness=0.035)
    top_face = ret_inset_top['faces'][0]

    # Langkah 4: Cekungan Area Tutup (Extrude Down)
    ret_cap = bmesh.ops.extrude_discrete_faces(bm, faces=[top_face])
    top_face = ret_cap['faces'][0]
    bmesh.ops.translate(bm, vec=(0, 0, can_depth_cap), verts=top_face.verts)

    # --- Bagian Bawah: Rim, Dome ---
    
    # Langkah 1: Bibir Penopang Bawah (Inset)
    ret_inset_bot = bmesh.ops.inset_region(bm, faces=[bottom_face], thickness=0.05)
    bottom_face = ret_inset_bot['faces'][0]

    # Langkah 2: Tarik ke Atas untuk Cekungan Khas Kaleng (Rim-Dome)
    ret_rim_bot = bmesh.ops.extrude_discrete_faces(bm, faces=[bottom_face])
    bottom_face = ret_rim_bot['faces'][0]
    bmesh.ops.translate(bm, vec=(0, 0, 0.035), verts=bottom_face.verts)

    # Langkah 3: Membuat Kubah (Dome) - Ekstrusi dan Skala
    ret_dome = bmesh.ops.extrude_discrete_faces(bm, faces=[bottom_face])
    bottom_face = ret_dome['faces'][0]
    bmesh.ops.translate(bm, vec=(0, 0, can_depth_dome), verts=bottom_face.verts)
    bmesh.ops.scale(bm, vec=(0.82, 0.82, 1.0), verts=bottom_face.verts)

    # Finalisasi Geometri di Edit Mode
    bmesh.update_edit_mesh(mesh)
    bpy.ops.object.mode_set(mode='OBJECT')

    # 4. Modifiers & Shading
    # Terapkan Smooth Shading
    bpy.ops.object.shade_smooth()

    # Tambahkan Modifier Bevel Terbatas Sudut untuk mempertegas bibir dan leher
    bevel_mod = can.modifiers.new(name="Bevel", type='BEVEL')
    bevel_mod.limit_method = 'ANGLE'
    bevel_mod.angle_limit = 0.523599 # ~30 derajat, hanya pertahankan bibir
    bevel_mod.width = 0.003
    bevel_mod.segments = 3
    
    # Tambahkan Modifier Subdivision Surface untuk kelengkungan super mulus
    subsurf_mod = can.modifiers.new(name="Subdivision", type='SUBSURF')
    subsurf_mod.levels = 2
    subsurf_mod.render_levels = 2

    # 5. Material Aluminium Polos
    mat_name = "Material_Aluminium"
    mat = bpy.data.materials.get(mat_name) or bpy.data.materials.new(name=mat_name)
    mat.use_nodes = True
    bsdf = mat.node_tree.nodes.get("Principled BSDF")
    
    if bsdf:
        bsdf.inputs['Base Color'].default_value = (0.75, 0.75, 0.8, 1) # Perak terang
        bsdf.inputs['Metallic'].default_value = 1.0
        bsdf.inputs['Roughness'].default_value = 0.25 # Tidak terlalu memantul, matte
    
    if len(can.data.materials) == 0:
        can.data.materials.append(mat)
    else:
        can.data.materials[0] = mat

# Jalankan Fungsi
create_perfect_can_final()
```
