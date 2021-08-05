# glbdump - Dump the contents of a Binary glTF (.glb) file

[glTF](https://github.com/KhronosGroup/glTF) is a really nice format, especially 
when used in the binary (.glb) form. But sometimes you want to inspect what 
exactly is inside a .glb file. For example, to check what makes a file so large, 
or to see the sizes of the different meshes.

You could check the JSON chunk at the start of a .glb file, but it's not really
convenient. Hence this little utility script.

Dependencies: only Python 3.x

## Example

```
# Dumping the well-known damaged helmet example file
$ ./glbdump.py ~/glTF-Sample-Models/2.0/DamagedHelmet/glTF-Binary/DamagedHelmet.glb
Total file size 3,773,916 bytes

JSON chunk (2,148 bytes)

Asset version 2.0
Asset generator "Khronos Blender glTF 2.0 exporter"

Elements:
   1 nodes
   5 images     (total 3,213,224 bytes)
   1 materials
   1 meshes
   5 textures

   1 buffers    (total 3,771,740 bytes)
   4 accessors  (total 558,504 bytes)

Images:
[   0]     935,629 bytes  image/jpeg   <unnamed>
[   1]   1,300,661 bytes  image/jpeg   <unnamed>
[   2]      97,499 bytes  image/jpeg   <unnamed>
[   3]     361,678 bytes  image/jpeg   <unnamed>
[   4]     517,757 bytes  image/jpeg   <unnamed>

Meshes:
[   0] "mesh_helmet_LP_13930damagedHelmet"     1P T       14,556V   46,356I   14,556N   14,556T0

Materials:
[   0] "Material_MR"

Buffers:
[   0]   3,771,740 bytes

Accessors:
[   0]   46,356x  SCALAR  UNSIGNED_SHORT       92,712 bytes
[   1]   14,556x  VEC3    FLOAT               174,672 bytes
[   2]   14,556x  VEC3    FLOAT               174,672 bytes
[   3]   14,556x  VEC2    FLOAT               116,448 bytes

```