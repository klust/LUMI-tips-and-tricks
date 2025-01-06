# ParaView

## Running the OOD container by hand as it is broken in OOD (end of 2024)

*Instructions provided by Orian.*

The `getEglCard` script in the container in use at the end of 2024 provides the wrong GPU device.
It sets `VGL_DISPLAY` to `/dev/dri/card0` which does not correspond to what `nvidia-smi`
suggests:

```
/dev/dri/by-path/pci-0000:48:00.0-card -> ../card1
```

The solution seems to be to run in a desktop session and then

```
singularity exec --nv /appl/local/ood/$SLURM_OOD_ENV/container_images/paraview_5.11.sif \
  bash -c 'export VGL_DISPLAY=/dev/dri/card1; vglrun paraview'
```

Check the "About" dialog in ParaView to confirm that the GPU is being used. It should show
a proper OpenGL version and as the OpenGL renderer "NVIDIA A40/PCIe/SSE2". 
