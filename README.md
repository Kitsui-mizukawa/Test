```
import bpy
import bmesh

def create_beverage_can():
    # 1. Bersihkan scene (menghapus objek yang ada agar tidak menumpuk)
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

    # 2. Buat objek Silinder dasar dengan proporsi kaleng
    bpy.ops.mesh.primitive_cylinder_add(
        vertices=64, 
        radius=0.33, 
        depth=1.2, 
        location=(0, 0, 0.6)
    )
    can = bpy.context.object
    can.name = "Kaleng_Minuman"

    # 3. Pindah ke Edit Mode untuk memodifikasi mesh
    bpy.ops.object.mode_set(mode='EDIT')
    bm = bmesh.from_edit_mesh(can.data)

    # Cari face bagian paling atas dan paling bawah berdasarkan posisi Z
    faces = [f for f in bm.faces]
    top_face = max(faces, key=lambda f: f.calc_center_bounds().z)
    bottom_face = min(faces, key=lambda f: f.calc_center_bounds().z)

    # --- Modifikasi Bagian Atas (Bibir Kaleng) ---
    # Buat Inset (lingkaran ke dalam)
    ret_top = bmesh.ops.inset_region(bm, faces=[top_face], thickness=0.03)
    new_top_face = ret_top['faces']
    # Extrude ke bawah untuk membuat cekungan atas
    ext_top = bmesh.ops.extrude_discrete_faces(bm, faces=new_top_face)
    bmesh.ops.translate(bm, vec=(0, 0, -0.03), verts=ext_top['faces'][0].verts)

    # --- Modifikasi Bagian Bawah (Cekungan Bawah) ---
    # Buat Inset 
    ret_bot = bmesh.ops.inset_region(bm, faces=[bottom_face], thickness=0.04)
    new_bot_face = ret_bot['faces']
    # Extrude ke atas untuk membuat cekungan bawah khas kaleng
    ext_bot = bmesh.ops.extrude_discrete_faces(bm, faces=new_bot_face)
    bmesh.ops.translate(bm, vec=(0, 0, 0.1), verts=ext_bot['faces'][0].verts)

    # Perbarui mesh dan kembali ke Object Mode
    bmesh.update_edit_mesh(can.data)
    bpy.ops.object.mode_set(mode='OBJECT')

    # 4. Penghalusan (Shading & Modifiers)
    # Terapkan Smooth Shading
    bpy.ops.object.shade_smooth()

    # Tambahkan Bevel Modifier agar sudut-sudutnya tidak tajam (realistis)
    bevel = can.modifiers.new(name="Bevel", type='BEVEL')
    bevel.limit_method = 'ANGLE'
    bevel.angle_limit = 0.523599 # Sekitar 30 derajat
    bevel.width = 0.005
    bevel.segments = 3

    # 5. Tambahkan Material Metalik
    mat = bpy.data.materials.new(name="Material_Kaleng")
    mat.use_nodes = True
    bsdf = mat.node_tree.nodes.get("Principled BSDF")
    if bsdf:
        # Mengatur parameter material agar terlihat seperti aluminium/besi yang dicat
        bsdf.inputs['Base Color'].default_value = (0.8, 0.05, 0.05, 1) # Warna Merah
        bsdf.inputs['Metallic'].default_value = 1.0
        bsdf.inputs['Roughness'].default_value = 0.25
    
    # Pasang material ke objek
    can.data.materials.append(mat)

# Jalankan fungsi
create_beverage_can()
```
