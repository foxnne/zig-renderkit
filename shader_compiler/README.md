# RenderKit Shader Compiler
The shader compiler leverages the fantastic Sokol Shader Compiler (thanks @floooh!) to compile glsl into all the other shader formats. For details on the GLSL input file see [these docs](https://github.com/floooh/sokol-tools/blob/master/docs/sokol-shdc.md).

```zig
const res = ShaderCompileStep.init(b, .{
    .shader = "shaders/shd.glsl",
    .additional_imports = &[_][]const u8{"usingnamespace @import(\"stuff\");"},
});

// override the float2 type, which defaults to [2]f32
res.float2_type = "math.Vec2";
exe.step.dependOn(&res.step);

// depending on the config passed to `ShaderCompileStep`, there may or may not be a package
if (res.package) |package| exe.addPackage(package);
```


## Writing Shaders
Lets take a look at what a commented example shader looks like. This is a simple sepia shader and illustrates a trick to make writing shaders without having to write any boilerplate. This trick is used in the GameKit repo for its shaders.

```bash
# we will leverage some reusable "@blocks" and the vertex shader from the default GameKit shader source file
@include relative/path/to/gamekit-defualt-shaders.glsl

# define our fragment shader name
@fs sepia_fs

# some magic: we include the "sprite_fs_main" block which is from the GameKit default shaders. It
# handles all the fragment shader boilerplate for us, so that all we have to do is define one function
# called "effect" and fill it in. This keeps our shaders nice and tidy
@include_block sprite_fs_main

# define the uniform struct. This will match the zig struct that gets generated by the shader compiler.
uniform sepia_fs_params {
	vec3 sepia_tone;
};

# implement the effect function. This is called by us from the "sprite_fs_main" block.
vec4 effect(sampler2D tex, vec2 tex_coord, vec4 vert_color) {
	vec4 color = texture(tex, tex_coord);
	color.rgb = mix(color.rgb, dot(color.rgb, vec3(0.3, 0.59, 0.11)) * sepia_tone, 0.75);
	return color;
}
@end

# define the shader program. It consists of a name, vert shader and frag shader. Notice that we
# just pull in the "sprite_vs" vert shader from the included GameKit shader file.
@program sepia sprite_vs sepia_fs
```
