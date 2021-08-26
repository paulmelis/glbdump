# glbdump - Dump the contents of a Binary glTF (.glb) file

[glTF](https://github.com/KhronosGroup/glTF) is a really nice format, especially 
when used in the binary (.glb) form. But sometimes you want to inspect what 
exactly is inside a .glb file. For example, to check what makes a file so large, 
to see mesh properties after export or for verifying certain material properties.

You could check the JSON chunk at the start of a .glb file (e.g. by loading
it in a text editor, or grepping the .glb itself), but that's not really convenient. 
Hence this little utility script.

Dependencies: 
- Python 3.x
- Optional: [Pillow](https://pypi.org/project/Pillow/) (or PIL). This is to use the `-l` option, see below.


## Example

```
# Inspecting the well-known damaged helmet example file
$ ./glbdump ~/glTF-Sample-Models/2.0/DamagedHelmet/glTF-Binary/DamagedHelmet.glb
Total file size 3,773,916 bytes

JSON chunk (2,148 bytes)

Asset version 2.0
Asset generator "Khronos Blender glTF 2.0 exporter"

Elements:
   1 nodes
   5 images     (3,213,224 bytes)
   1 materials
   1 meshes
   5 textures

   1 buffers    (3,771,740 bytes)
   4 accessors  (558,504 bytes)

Images:
[   0]      935,629 bytes  image/jpeg   <unnamed>
[   1]    1,300,661 bytes  image/jpeg   <unnamed>
[   2]       97,499 bytes  image/jpeg   <unnamed>
[   3]      361,678 bytes  image/jpeg   <unnamed>
[   4]      517,757 bytes  image/jpeg   <unnamed>

Meshes:
[   0] "mesh_helmet_LP_13930damagedHelmet"
            558,504 bytes    1P T      14,556V   46,356I   14,556N   14,556T0

Materials:
[   0]     opaque       [BC MR N E O]   "Material_MR"

Buffers:
[   0]   3,771,740 bytes

Accessors:
[   0]   46,356x  SCALAR  UNSIGNED_SHORT       92,712 bytes  (M0 "mesh_helmet_LP_13930damagedHelmet", P0, I)
[   1]   14,556x  VEC3    FLOAT               174,672 bytes  (M0 "mesh_helmet_LP_13930damagedHelmet", P0, P)
[   2]   14,556x  VEC3    FLOAT               174,672 bytes  (M0 "mesh_helmet_LP_13930damagedHelmet", P0, N)
[   3]   14,556x  VEC2    FLOAT               116,448 bytes  (M0 "mesh_helmet_LP_13930damagedHelmet", P0, T0)

```

Apart from the detailed contents it can be seen that of the 3.7 MB making up
the .glb file 3.2 MB is used for 5 images (textures) and 559 kB for mesh data. 


## Output

Most of the output should be straightforward to understand, but here is a bit
more explanation for certain types of data (using the example file shown above).

### Meshes

```
Meshes:
[   0] "mesh_helmet_LP_13930damagedHelmet"
            558,504 bytes    1P T      14,556V   46,356I   14,556N   14,556T0
```

This shows that mesh 0 consists of a single primitive (`1P`). Each set of primitive 
data is usually turned into a single draw call. This particular primitive is 
drawn as a set of TRIANGLES (`T`). The possible types are:

| Type  | Primitive mode   |
| ----- | ---------------- |
| `T`   | `TRIANGLES`      |
| `TS`  | `TRIANGLE_STRIP` |
| `TF`  | `TRIANGLE_FAN`   |
| `L`   | `LINES`          |
| `LL`  | `LINE_LOOP`      |
| `LS`  | `LINE_STRIP`     |
| `P`   | `POINTS`         |
    
The primitive has 14,556 vertices (`14,556V`), is drawn using 46,356 indices, 
has 14,556 normals and 14,556 texture coordinates (set 0). If there were a second 
set of texture coordinates these would be listed as `T1`. Vertex colors would be 
listed as `C0`. 

### Materials

```
Materials:
[   0]     opaque       [BC MR N E O]   "Material_MR"
```

This material has no transparency (`opaque`), other options are `alpha-mask`
and `alpha-blend`. The set of characters between brackets lists the different
textures used in this material. The possible types are:

| Type | Texture                  | Notes                                |
| ---- | ------------------------ | ------------------------------------ |
|`BC`  | baseColorTexture         | (for pbrMetallicRoughness materials) |
|`MR`  | metallicRoughnessTexture | (for pbrMetallicRoughness materials) |
|`N`   | normalTexture            |                                      |
|`E`   | emissiveTexture          |                                      |
|`O`   | occlusionTexture         |                                      |
   
If a material line includes `2S` then this means that the material is
double-sided (i.e. back-face culling disabled).


## Options

### Write images to files (`-i`)

You can dump the embedded images to files with the `-i` option. These will be
written to files of the form `img-0000.<ext>`, with the extension depending
on the mime-type. Note that this writes the image file bytes as embedded 
in the .glb file, no processing or conversion is done whatsoever.

### Load images (`-l`)

With the `-l` option each block of image data is loaded, for determining
image properties (resolution and channels). 

```
$ ./glbdump -l ~/models/glTF-Sample-Models/2.0/DamagedHelmet/glTF-Binary/DamagedHelmet.glb
...
Images:
[   0]      935,629 bytes  image/jpeg   <unnamed>                                  2048x2048 RGB
[   1]    1,300,661 bytes  image/jpeg   <unnamed>                                  2048x2048 RGB
[   2]       97,499 bytes  image/jpeg   <unnamed>                                  2048x2048 RGB
[   3]      361,678 bytes  image/jpeg   <unnamed>                                  2048x2048 RGB
[   4]      517,757 bytes  image/jpeg   <unnamed>                                  2048x2048 RGB
```

As this requires reading and parsing the image data (which may take some time 
for large files, or many images) this option is not enabled by default.

Note that this option requires the Pillow (or PIL) `Image` module to be available.

### Dump accessor values (`-a`)

You can dump the row values in the accessors with the `-a` option, which allows
inspecting of the low-level values. For example:

```
$ ./glbdump -a ~/glTF-Sample-Models/2.0/DamagedHelmet/glTF-Binary/DamagedHelmet.glb
...

Meshes:
[   0] "mesh_helmet_LP_13930damagedHelmet"
            558,504 bytes    1P T      14,556V   46,356I   14,556N   14,556T0
...

Accessors:
[   0]   46,356x  SCALAR  UNSIGNED_SHORT       92,712 bytes  (M0 "mesh_helmet_LP_13930damagedHelmet", P0, I)
  [0000]     0                                      # triangle indices, 3 per triangle
  [0001]     1
  [0002]     2
  [0003]     2
  [0004]     3
  [0005]     0
  ......
  [46354] 14555
  [46355] 14542
[   1]   14,556x  VEC3    FLOAT               174,672 bytes  (M0 "mesh_helmet_LP_13930damagedHelmet", P0, P)
  [0000]    -0.611995    -0.030941     0.483090     # vertex positions
  [0001]    -0.579505     0.056274     0.521758 
  [0002]    -0.573584     0.063534     0.486858 
  ......
  [14554]     0.645380    -0.719006    -0.067919 
  [14555]     0.646051    -0.691662    -0.047538 
[   2]   14,556x  VEC3    FLOAT               174,672 bytes  (M0 "mesh_helmet_LP_13930damagedHelmet", P0, N)
  [0000]    -0.918302     0.383801     0.096835     # normals
  [0001]    -0.918668     0.389630    -0.064608 
  [0002]    -0.880795     0.444777    -0.162206 
  [0003]    -0.865139     0.498337    -0.056307 
  ......
  [14553]     0.000000     0.069063     0.997589 
  [14554]     0.000000    -0.597613     0.801752 
  [14555]     0.000000    -0.597613     0.801752 
[   3]   14,556x  VEC2    FLOAT               116,448 bytes  (M0 "mesh_helmet_LP_13930damagedHelmet", P0, T0)
  [0000]     0.704686     1.245604                  # texture coordinates
  [0001]     0.675778     1.256622              
  [0002]     0.672684     1.245967 
  [0003]     0.697708     1.233138 
  ......
  [14553]     0.987441     1.465754 
  [14554]     0.996629     1.471167 
  [14555]     0.996779     1.471621   
```


## Disclaimer

This tool has only been tested on a limited number of .glb files, mostly as
exported by Blender, plus some samples files from the Khronos repository.

Not all possible contents of a glTF file is dumped as output. I.e, there may
be more things in a .glb file than are shown. Plus, some exotic features are 
not recognized and/or not handled correctly (such as multiple buffers).
