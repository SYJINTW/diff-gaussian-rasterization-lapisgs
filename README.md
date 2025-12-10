# Differential Gaussian Rasterization

This is the modified version of differential Gaussian rasterization for object-based LapisGS representation.

## Changing
For inputs, we modify the type of background, and adding background depth and gaussians' resolutions.
For output, we add final opacity of each pixel.

## Usage
### Background Generation
Generate a pure color background and pure depth background. The depth background (bg_depth) is not used in this version of rasterization, but we keep it for future usage.
```python
bg_color = torch.tensor([0, 0, 0], dtype=torch.float32, device="cuda").view(3, 1, 1)
bg_color = bg_color.expand(3, viewpoint_camera.image_height, viewpoint_camera.image_width)
bg_depth = torch.full((1, viewpoint_camera.image_height, viewpoint_camera.image_width), 0, dtype=torch.float32, device="cuda")
```

### Gaussian Rasterization Settings
```python
raster_settings = GaussianRasterizationSettings(
        image_height=int(viewpoint_camera.image_height),
        image_width=int(viewpoint_camera.image_width),
        tanfovx=tanfovx,
        tanfovy=tanfovy,
        bg=bg_color, # [YC] note: we change the type of bg_color
        scale_modifier=scaling_modifier,
        viewmatrix=viewpoint_camera.world_view_transform,
        projmatrix=viewpoint_camera.full_proj_transform,
        sh_degree=pc.active_sh_degree,
        campos=viewpoint_camera.camera_center,
        prefiltered=False,
        debug=pipe.debug,
        depth=bg_depth # [YC] add
        # antialiasing=pipe.antialiasing # [YC] remove: because this code is based on older version of Gaussian Rasterization code
    )
rasterizer = GaussianRasterizer(raster_settings=raster_settings)
```

### Gaussian Rendering
```python
gs_res = 1.0 # [YC] note: 1.0 --> 1/1, 2.0 --> 1/2, 4.0 --> 1/4
resolutions = torch.tensor([gs_res for _ in range(len(pc.get_xyz))], device="cuda")
rendered_image, radii, final_opacity = rasterizer(
            means3D = means3D,
            means2D = means2D,
            dc = dc,
            shs = shs,
            colors_precomp = colors_precomp,
            opacities = opacity,
            resolutions = resolutions, # [YC] add
            scales = scales,
            rotations = rotations,
            cov3D_precomp = cov3D_precomp) # [YC] note: final_opacity is the (1 - (total opacity of all the Gaussian in each pixel))
```


## Acknowledgements
This repo (rasterization) is based on the rasterization engine for the paper "3D Gaussian Splatting for Real-Time Rendering of Radiance Fields". If you can make use of it in your own research, please be also kind to cite their excellent works.

<section class="section" id="BibTeX">
  <div class="container is-max-desktop content">
    <pre><code>@Article{kerbl3Dgaussians,
      author       = {Kerbl, Bernhard and Kopanas, Georgios and Leimk{\"u}hler, Thomas and Drettakis, George},
      title        = {3D Gaussian Splatting for Real-Time Radiance Field Rendering},
      journal      = {ACM Transactions on Graphics},
      number       = {4},
      volume       = {42},
      month        = {July},
      year         = {2023},
      url          = {https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/}
}</code></pre>
  </div>
</section>

## Citation
If you find this repository/work helpful in your research, welcome to cite these papers and give a ⭐.
