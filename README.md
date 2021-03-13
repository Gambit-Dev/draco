<p align="center">
<img width="350px" src="docs/artwork/draco3d-vert.svg" />
</p>

Description
===========

Draco is a library for compressing and decompressing 3D geometric [meshes] and
[point clouds]. It is intended to improve the storage and transmission of 3D
graphics.

Draco was designed and built for compression efficiency and speed. The code
supports compressing points, connectivity information, texture coordinates,
color information, normals, and any other generic attributes associated with
geometry. With Draco, applications using 3D graphics can be significantly
smaller without compromising visual fidelity. For users, this means apps can
now be downloaded faster, 3D graphics in the browser can load quicker, and VR
and AR scenes can now be transmitted with a fraction of the bandwidth and
rendered quickly.

Draco is released as C++ source code that can be used to compress 3D graphics
as well as C++ and Javascript decoders for the encoded data.

Usage
======
Command Line Applications
------------------------

The default target created from the build files will be the `draco_encoder`
and `draco_decoder` command line applications. For both applications, if you
run them without any arguments or `-h`, the applications will output usage and
options.

Encoding Tool
-------------

`draco_encoder` will read OBJ or PLY files as input, and output Draco-encoded
files. We have included Stanford's [Bunny] mesh for testing. The basic command
line looks like this:

~~~~~ bash
./draco_encoder -i testdata/bun_zipper.ply -o out.drc
~~~~~

A value of `0` for the quantization parameter will not perform any quantization
on the specified attribute. Any value other than `0` will quantize the input
values for the specified attribute to that number of bits. For example:

~~~~~ bash
./draco_encoder -i testdata/bun_zipper.ply -o out.drc -qp 14
~~~~~

will quantize the positions to 14 bits (default is 11 for the position
coordinates).

In general, the more you quantize your attributes the better compression rate
you will get. It is up to your project to decide how much deviation it will
tolerate. In general, most projects can set quantization values of about `11`
without any noticeable difference in quality.

The compression level (`-cl`) parameter turns on/off different compression
features.

~~~~~ bash
./draco_encoder -i testdata/bun_zipper.ply -o out.drc -cl 8
~~~~~

In general, the highest setting, `10`, will have the most compression but
worst decompression speed. `0` will have the least compression, but best
decompression speed. The default setting is `7`.

Encoding Point Clouds
---------------------

You can encode point cloud data with `draco_encoder` by specifying the
`-point_cloud` parameter. If you specify the `-point_cloud` parameter with a
mesh input file, `draco_encoder` will ignore the connectivity data and encode
the positions from the mesh file.

~~~~~ bash
./draco_encoder -point_cloud -i testdata/bun_zipper.ply -o out.drc
~~~~~

This command line will encode the mesh input as a point cloud, even though the
input might not produce compression that is representative of other point
clouds. Specifically, one can expect much better compression rates for larger
and denser point clouds.

Decoding Tool
-------------

`draco_decoder` will read Draco files as input, and output OBJ or PLY files.
The basic command line looks like this:

~~~~~ bash
./draco_decoder -i in.drc -o out.obj
~~~~~

C++ Decoder API
-------------

If you'd like to add decoding to your applications you will need to include
the `draco_dec` library. In order to use the Draco decoder you need to
initialize a `DecoderBuffer` with the compressed data. Then call
`DecodeMeshFromBuffer()` to return a decoded mesh object or call
`DecodePointCloudFromBuffer()` to return a decoded `PointCloud` object. For
example:

~~~~~ cpp
draco::DecoderBuffer buffer;
buffer.Init(data.data(), data.size());

const draco::EncodedGeometryType geom_type =
    draco::GetEncodedGeometryType(&buffer);
if (geom_type == draco::TRIANGULAR_MESH) {
  unique_ptr<draco::Mesh> mesh = draco::DecodeMeshFromBuffer(&buffer);
} else if (geom_type == draco::POINT_CLOUD) {
  unique_ptr<draco::PointCloud> pc = draco::DecodePointCloudFromBuffer(&buffer);
}
~~~~~

Please see [src/draco/mesh/mesh.h](src/draco/mesh/mesh.h) for the full `Mesh` class interface and
[src/draco/point_cloud/point_cloud.h](src/draco/point_cloud/point_cloud.h) for the full `PointCloud` class interface.

Metadata API
------------
Starting from v1.0, Draco provides metadata functionality for encoding data
other than geometry. It could be used to encode any custom data along with the
geometry. For example, we can enable metadata functionality to encode the name
of attributes, name of sub-objects and customized information.
For one mesh and point cloud, it can have one top-level geometry metadata class.
The top-level metadata then can have hierarchical metadata. Other than that,
the top-level metadata can have metadata for each attribute which is called
attribute metadata. The attribute metadata should be initialized with the
correspondent attribute id within the mesh. The metadata API is provided both
in C++ and Javascript.
For example, to add metadata in C++:

~~~~~ cpp
draco::PointCloud pc;
// Add metadata for the geometry.
std::unique_ptr<draco::GeometryMetadata> metadata =
  std::unique_ptr<draco::GeometryMetadata>(new draco::GeometryMetadata());
metadata->AddEntryString("description", "This is an example.");
pc.AddMetadata(std::move(metadata));

// Add metadata for attributes.
draco::GeometryAttribute pos_att;
pos_att.Init(draco::GeometryAttribute::POSITION, nullptr, 3,
             draco::DT_FLOAT32, false, 12, 0);
const uint32_t pos_att_id = pc.AddAttribute(pos_att, false, 0);

std::unique_ptr<draco::AttributeMetadata> pos_metadata =
    std::unique_ptr<draco::AttributeMetadata>(
        new draco::AttributeMetadata(pos_att_id));
pos_metadata->AddEntryString("name", "position");

// Directly add attribute metadata to geometry.
// You can do this without explicitly add |GeometryMetadata| to mesh.
pc.AddAttributeMetadata(pos_att_id, std::move(pos_metadata));
~~~~~

To read metadata from a geometry in C++:

~~~~~ cpp
// Get metadata for the geometry.
const draco::GeometryMetadata *pc_metadata = pc.GetMetadata();

// Request metadata for a specific attribute.
const draco::AttributeMetadata *requested_pos_metadata =
  pc.GetAttributeMetadataByStringEntry("name", "position");
~~~~~

License
=======
Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

<http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
