```
import bpy
import bmesh

def create_perfect_can():
    # 1. Bersihkan Scene
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

    # 2. Buat Silinder Dasar DENGAN TUTUP (NGON) agar mudah dimodifikasi
    bpy.ops.mesh.primitive_cylinder_add(
        vertices=64, 
        radius=0.5, 
        depth=2.0, 
        location=(0, 0, 1.0),
        end_fill_type='NGON'
    )
    can = bpy.context.object
    can.name = "Kaleng_Aluminium_Realistis"

    # 3. Pindah ke Edit Mode
    bpy.ops.object.mode_set(mode='EDIT')
    bm = bmesh.from_edit_mesh(can.data)
    bm.faces.ensure_lookup_table()

    # Cari permukaan (face) paling atas dan paling bawah
    top_face = max(bm.faces, key=lambda f: f.calc_center_bounds().z)
    bottom_face = min(bm.faces, key=lambda f: f.calc_center_bounds().z)

    # --- Modifikasi Bagian Atas (Necking & Tutup) ---
    
    # Langkah 1: Penciutan Leher (Necking)
    ret_neck = bmesh.ops.extrude_discrete_faces(bm, faces=[top_face])
    top_face = ret_neck['faces'][0]
    bmesh.ops.translate(bm, vec=(0, 0, 0.2), verts=top_face.verts)
    bmesh.ops.scale(bm, vec=(0.82, 0.82, 1.0), verts=top_face.verts)

    # Langkah 2: Bibir Kaleng Tegak (Lip)
    ret_lip = bmesh.ops.extrude_discrete_faces(bm, faces=[top_face])
    top_face = ret_lip['faces'][0]
    bmesh.ops.translate(bm, vec=(0, 0, 0.05), verts=top_face.verts)

    # Langkah 3: Ketebalan Bibir Kaleng
    ret_inset_top = bmesh.ops.inset_region(bm, faces=[top_face], thickness=0.04)
    top_face = ret_inset_top['faces'][0]

    # Langkah 4: Cekungan Area Tutup
    ret_cap = bmesh.ops.extrude_discrete_faces(bm, faces=[top_face])
    top_face = ret_cap['faces'][0]
    bmesh.ops.translate(bm, vec=(0, 0, -0.06), verts=top_face.verts)

    # --- Modifikasi Bagian Bawah (Cekungan / Dome) ---
    
    # Langkah 1: Bibir Penopang Bawah
    ret_inset_bot = bmesh.ops.inset_region(bm, faces=[bottom_face], thickness=0.06)
    bottom_face = ret_inset_bot['faces'][0]

    # Langkah 2: Tarik ke Atas untuk Cekungan Khas Kaleng
    ret_dome = bmesh.ops.extrude_discrete_faces(bm, faces=[bottom_face])
    bottom_face = ret_dome['faces'][0]
    bmesh.ops.translate(bm, vec=(0, 0, 0.15), verts=bottom_face.verts)
    # Sedikit diperkecil agar membentuk kubah (dome) melengkung
    bmesh.ops.scale(bm, vec=(0.8, 0.8, 1.0), verts=bottom_face.verts)

    # Perbarui mesh ke Blender
    bmesh.update_edit_mesh(can.data)
    bpy.ops.object.mode_set(mode='OBJECT')

    # 4. Modifiers & Shading
    # Terapkan Smooth Shading
    bpy.ops.object.shade_smooth()

    # Tambahkan Bevel Modifier untuk mempertegas sudut bibir kaleng
    bevel = can.modifiers.new(name="Bevel", type='BEVEL')
    bevel.limit_method = 'ANGLE'
    bevel.angle_limit = 0.52 # Berlaku untuk sudut di atas 30 derajat
    bevel.width = 0.005
    bevel.segments = 3
    
    # Tambahkan Subdivision Surface untuk kelengkungan sempurna (Super Mulus)
    subsurf = can.modifiers.new(name="Subdivision", type='SUBSURF')
    subsurf.levels = 2
    subsurf.render_levels = 2

    # 5. Material Besi / Aluminium
    mat = bpy.data.materials.new(name="Aluminium_Polos")
    mat.use_nodes = True
    bsdf = mat.node_tree.nodes.get("Principled BSDF")
    if bsdf:
        bsdf.inputs['Base Color'].default_value = (0.8, 0.8, 0.85, 1) # Abu-abu keputihan
        bsdf.inputs['Metallic'].default_value = 1.0
        bsdf.inputs['Roughness'].default_value = 0.25 # Tidak terlalu memantul, seperti matte
    can.data.materials.append(mat)

# Eksekusi Script
create_perfect_can()
```
