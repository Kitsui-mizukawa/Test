```
import bpy
import bmesh
from math import pi

def create_accurate_can():
    # 1. Bersihkan Scene
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

    # Parameter Geometri (dalam unit Blender/Meter)
    # Sesuaikan untuk proporsi yang berbeda
    subdivisions_radial = 128 # Gunakan lebih banyak vertices untuk lengkungan mulus
    can_radius_body = 0.5
    can_height_body = 2.0  # Ini akan menjadi bagian lurus
    
    can_radius_neck = 0.41 # Radius leher ramping
    can_height_neck = 0.25 # Tinggi transisi necking
    can_height_lip = 0.06 # Tinggi bibir paling atas
    
    can_depth_dome = 0.12 # Kedalaman kubah bawah

    # 2. Buat Silinder Dasar (Hanya Badan Lurus, tanpa tutup)
    bpy.ops.mesh.primitive_cylinder_add(
        vertices=subdivisions_radial, 
        radius=can_radius_body, 
        depth=can_height_body, 
        location=(0, 0, can_height_body / 2),
        end_fill_type='NOTHING' # Loop terbuka untuk manipulasi bmesh
    )
    can = bpy.context.object
    can.name = "Kaleng_Aluminium_Akurat"
    mesh = can.data

    # 3. Masuk ke Edit Mode untuk Geometri Kompleks
    bpy.ops.object.mode_set(mode='EDIT')
    bm = bmesh.from_edit_mesh(mesh)
    bm.verts.ensure_lookup_table()

    # Temukan Loop Vertikal Atas dan Bawah
    max_z = max(v.co.z for v in bm.verts)
    min_z = min(v.co.z for v in bm.verts)
    
    top_loop_verts = [v for v in bm.verts if v.co.z > max_z - 0.001]
    bottom_loop_verts = [v for v in bm.verts if v.co.z < min_z + 0.001]

    # --- Bagian Atas: Necking, Bibir, Tutup ---
    # Langkah Penciutan Leher (2-step transition untuk kehalusan)
    # Langkah 1 (Transisi pertama)
    max_z = max(v.co.z for v in bm.verts)
    top_loop_edges = set(e for v in bm.verts if v.co.z > max_z - 0.001 for e in v.link_edges if e.is_manifold and e.verts[0].co.z > max_z - 0.001 and e.verts[1].co.z > max_z - 0.001)
    ret_neck1 = bmesh.ops.extrude_edge_collection(bm, edges=list(top_loop_edges))
    bmesh.ops.translate(bm, vec=(0, 0, can_height_neck / 2.0), verts=ret_neck1['verts'])
    bmesh.ops.scale(bm, vec=((can_radius_body + can_radius_neck) / (2.0 * can_radius_body), (can_radius_body + can_radius_neck) / (2.0 * can_radius_body), 1.0), verts=ret_neck1['verts'])
    
    # Langkah 2 (Final necking)
    max_z = max(v.co.z for v in bm.verts)
    top_loop_edges = set(e for v in bm.verts if v.co.z > max_z - 0.001 for e in v.link_edges if e.is_manifold and e.verts[0].co.z > max_z - 0.001 and e.verts[1].co.z > max_z - 0.001)
    ret_neck2 = bmesh.ops.extrude_edge_collection(bm, edges=list(top_loop_edges))
    bmesh.ops.translate(bm, vec=(0, 0, can_height_neck / 2.0), verts=ret_neck2['verts'])
    bmesh.ops.scale(bm, vec=(can_radius_neck / ((can_radius_body + can_radius_neck) / 2.0), can_radius_neck / ((can_radius_body + can_radius_neck) / 2.0), 1.0), verts=ret_neck2['verts'])

    # Bibir Kaleng (Lip)
    max_z = max(v.co.z for v in bm.verts)
    top_loop_edges = set(e for v in bm.verts if v.co.z > max_z - 0.001 for e in v.link_edges if e.is_manifold and e.verts[0].co.z > max_z - 0.001 and e.verts[1].co.z > max_z - 0.001)
    ret_lip1 = bmesh.ops.extrude_edge_collection(bm, edges=list(top_loop_edges))
    bmesh.ops.translate(bm, vec=(0, 0, can_height_lip), verts=ret_lip1['verts'])

    # Pelek Bibir (Menghadap Ke Dalam)
    max_z = max(v.co.z for v in bm.verts)
    top_loop_edges = set(e for v in bm.verts if v.co.z > max_z - 0.001 for e in v.link_edges if e.is_manifold and e.verts[0].co.z > max_z - 0.001 and e.verts[1].co.z > max_z - 0.001)
    ret_lip_inner = bmesh.ops.extrude_edge_collection(bm, edges=list(top_loop_edges))
    bmesh.ops.scale(bm, vec=(0.95, 0.95, 1.0), verts=ret_lip_inner['verts']) # Skala ke dalam

    # Kedalaman Tutup
    max_z = max(v.co.z for v in bm.verts)
    top_loop_edges = set(e for v in bm.verts if v.co.z > max_z - 0.001 for e in v.link_edges if e.is_manifold and e.verts[0].co.z > max_z - 0.001 and e.verts[1].co.z > max_z - 0.001)
    ret_cap_depth = bmesh.ops.extrude_edge_collection(bm, edges=list(top_loop_edges))
    bmesh.ops.translate(bm, vec=(0, 0, -0.06), verts=ret_cap_depth['verts']) # Extrude ke bawah

    # Isi Tutup
    max_z = max(v.co.z for v in bm.verts)
    cap_loop_verts = [v for v in bm.verts if v.co.z > max_z - 0.001]
    bmesh.ops.contextual_create(bm, geom=cap_loop_verts)

    # --- Bagian Bawah: Rim, Dome ---
    # Rim Bawah (Skala Ke Dalam untuk ketebalan)
    min_z = min(v.co.z for v in bm.verts)
    bottom_loop_edges = set(e for v in bm.verts if v.co.z < min_z + 0.001 for e in v.link_edges if e.is_manifold and e.verts[0].co.z < min_z + 0.001 and e.verts[1].co.z < min_z + 0.001)
    ret_rim_bot = bmesh.ops.extrude_edge_collection(bm, edges=list(bottom_loop_edges))
    bmesh.ops.scale(bm, vec=(0.90, 0.90, 1.0), verts=ret_rim_bot['verts'])

    # Dome (Extrude Ke Atas)
    min_z = min(v.co.z for v in bm.verts)
    dome_loop_edges = set(e for v in bm.verts if v.co.z < min_z + 0.001 for e in v.link_edges if e.is_manifold and e.verts[0].co.z < min_z + 0.001 and e.verts[1].co.z < min_z + 0.001)
    ret_dome = bmesh.ops.extrude_edge_collection(bm, edges=list(dome_loop_edges))
    bmesh.ops.translate(bm, vec=(0, 0, can_depth_dome), verts=ret_dome['verts'])

    # Isi Dome
    min_z = min(v.co.z for v in bm.verts)
    dome_loop_verts = [v for v in bm.verts if v.co.z < min_z + 0.001]
    bmesh.ops.contextual_create(bm, geom=dome_loop_verts)

    # 4. Finalisasi Geometri
    bmesh.update_edit_mesh(mesh)
    bpy.ops.object.mode_set(mode='OBJECT')

    # 5. Penghalusan (Smooth)
    bpy.ops.object.shade_smooth()
    
    # Tambahkan Bevel Modifier di Object Mode (untuk menghaluskan bibir kaleng dengan lembut tanpa mengubah bentuk keseluruhan)
    bevel_mod = can.modifiers.new(name="SoftBevel", type='BEVEL')
    bevel_mod.limit_method = 'ANGLE'
    bevel_mod.angle_limit = 0.523599 # ~30 derajat, hanya haluskan sudut tajam
    bevel_mod.width = 0.003
    bevel_mod.segments = 3

    # 6. Material Aluminium Metalik
    mat_name = "Material_Aluminium"
    mat = bpy.data.materials.get(mat_name) or bpy.data.materials.new(name=mat_name)
    mat.use_nodes = True
    bsdf = mat.node_tree.nodes.get("Principled BSDF")
    
    if bsdf:
        bsdf.inputs['Base Color'].default_value = (0.8, 0.8, 0.8, 1) # Perak lebih terang
        bsdf.inputs['Metallic'].default_value = 1.0
        bsdf.inputs['Roughness'].default_value = 0.22
        bsdf.inputs['Specular'].default_value = 0.6
    
    if len(can.data.materials) == 0:
        can.data.materials.append(mat)
    else:
        can.data.materials[0] = mat

# Jalankan Fungsi
create_accurate_can()
```
