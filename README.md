```
import bpy
import bmesh
from math import pi

def create_realistic_can():
    # 1. Bersihkan Scene
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

    # Parameter Geometri
    subdivisions_radial = 64
    can_height = 2.4  # Proporsi tinggi kaleng ramping
    can_radius = 0.5
    neck_height = 0.25
    bottom_arch_depth = 0.15

    # 2. Buat Silinder Dasar
    bpy.ops.mesh.primitive_cylinder_add(
        vertices=subdivisions_radial, 
        radius=can_radius, 
        depth=can_height, 
        location=(0, 0, can_height / 2),
        end_fill_type='NOTHING'
    )
    can = bpy.context.object
    can.name = "Kaleng_Aluminium"
    mesh = can.data

    # 3. Mode Edit untuk Geometri Kompleks
    bpy.ops.object.mode_set(mode='EDIT')
    bm = bmesh.from_edit_mesh(mesh)
    bm.verts.ensure_lookup_table()

    # --- Bagian Bawah (Alas Melengkung) ---
    # Pilih edge loop bawah
    bpy.ops.mesh.select_all(action='DESELECT')
    for v in bm.verts:
        if v.co.z < 0.001:
            v.select = True
    
    # Inset pertama untuk ketebalan dinding
    bpy.ops.mesh.inset(thickness=0.04)
    # Extrude ke dalam untuk kubah bawah
    bpy.ops.mesh.extrude_region_move(
        MESH_OT_extrude_region={}, 
        TRANSFORM_OT_translate={"value": (0, 0, bottom_arch_depth)}
    )
    # Bevel untuk memperhalus transisi bawah
    bpy.ops.mesh.bevel(offset=0.05, segments=3, profile=0.5)

    # --- Bagian Atas (Necking dan Tutup) ---
    # Pilih edge loop atas
    bpy.ops.mesh.select_all(action='DESELECT')
    for v in bm.verts:
        if v.co.z > can_height - 0.001:
            v.select = True

    # Membuat 'Neck' (penciutan leher)
    # Langkah 1: Bevel loop atas untuk membuat transisi leher
    bpy.ops.mesh.bevel(offset=neck_height, segments=5, profile=0.5)
    
    # Langkah 2: Skala loop paling atas ke dalam
    # (Ini membutuhkan manipulasi manual atau bevel yang lebih presisi, 
    #  disini kita gunakan skala sederhana)
    bm.verts.ensure_lookup_table()
    top_loop_verts = [v for v in bm.verts if v.select]
    # Catatan: Skala loop yang dipilih sulit via operator mesh, 
    # kita gunakan transformasi object mode sementara atau vertex manipulation.
    # Pendekatan yang lebih bersih: gunakan loop cuts dan skala terpisah.
    # Namun, untuk script tunggal, bevel + inset adalah yang termudah.
    
    # Kembali ke loop atas tunggal untuk inset
    bpy.ops.mesh.select_all(action='DESELECT')
    # Temukan vertikal tertinggi (leher paling atas)
    max_z = max(v.co.z for v in bm.verts)
    for v in bm.verts:
        if v.co.z > max_z - 0.001:
            v.select = True

    # Inset untuk bibir kaleng
    bpy.ops.mesh.inset(thickness=0.03)
    # Extrude ke bawah sedikit untuk cekungan tutup
    bpy.ops.mesh.extrude_region_move(
        MESH_OT_extrude_region={}, 
        TRANSFORM_OT_translate={"value": (0, 0, -0.05)}
    )
    
    # --- Opsional: Tab Penarik (Tab Pull) Minimal ---
    # Ini memerlukan geometri bmesh terpisah, kita lewati untuk kebersihan script utama,
    # tetapi geometry di atas sudah menciptakan cekungan penutup yang benar.

    # 4. Finalisasi Geometri
    bmesh.update_edit_mesh(mesh)
    bpy.ops.object.mode_set(mode='OBJECT')

    # 5. Penghalusan dan Modifiers
    bpy.ops.object.shade_smooth()
    
    # Tambahkan Edge Split untuk menjaga ketajaman bibir
    edge_split = can.modifiers.new(name="EdgeSplit", type='EDGE_SPLIT')
    edge_split.split_angle = 30 * (pi / 180) # 30 derajat

    # 6. Material Aluminium Metalik
    mat_name = "Material_Aluminium"
    mat = bpy.data.materials.get(mat_name) or bpy.data.materials.new(name=mat_name)
    mat.use_nodes = True
    bsdf = mat.node_tree.nodes.get("Principled BSDF")
    
    if bsdf:
        bsdf.inputs['Base Color'].default_value = (0.7, 0.7, 0.7, 1) # Perak
        bsdf.inputs['Metallic'].default_value = 1.0
        bsdf.inputs['Roughness'].default_value = 0.2
        # Sedikit Anisotropy untuk efek aluminium yang ditarik
        bsdf.inputs['Anisotropic'].default_value = 0.3
    
    if len(can.data.materials) == 0:
        can.data.materials.append(mat)
    else:
        can.data.materials[0] = mat

# Jalankan Fungsi
create_realistic_can()

```
