# C(T) specimen

## Baseline material response, no hydrogen

### Abaqus files:

* `noH.inp`: Abaqus job file
* `_AISI.inp`: include file with AISI steel material block
* `_mesh.inp`: include file with mesh definition

### Hydra generated files:

* `_coh.inp`: include file with coehesive TSL law (generated)

### Run instructions.

1. Generate the `_coh.inp` material block using the `hydra-utils/gencoh.py` program

        $ python ../../hydra-utils/gencoh.py 0.00075 0.00975 0.0157 2600 > _coh.inp
   
   The  generated file should be identical to `./results/_coh.inp`

2. Run abaqus

        $ abaqus analysis job=noH

3. Post-process results

        $ abaqus python ../../abapython/saveh.py noH

   Post-processing will generate the data file `noH.npz`, which is stored
   in numpy [`.npz` format](http://docs.scipy.org/doc/numpy/reference/generated/numpy.savez.html).
   A reference results file is given in the `results` directory.

## Hydrogen charged response (3-step procedure)

### Abaqus job files:

* `H-1static.inp`: preliminary static analysis job file
* `H-2massdiff.inp`: mass diffusion analysis job file
* `H-3cohesive.inp`: cohesive analysis job file
* `_AISI.inp`: include file with AISI steel material block
* `_mesh_sa.inp`: include file with mesh definition for static analysis
* `_mesh_md.inp`: include file with mesh definition for mass diffusion 
  analysis
* `_mesh.inp`: include file with mesh definition with cohesive elements
* `conc.f`: user subroutine
* `conc.inc`: user subroutine include file

### Hydra generated files:

* _coh-H.inp: include file with coehesive TSL law (generated)
* H-3cohesive.map: solid to cohesive mapping file (generated)

### Run instructions.

1. Perform a preliminary static analysis to calculate the hydrostatic
   stress field.

        $ abaqus analysis job=H-1static

2. Mass diffusion analysis

        $ abaqus analysis job=H-2massdiff

3. Generate cohesive element TSL

        $ python ../../hydra-utils/gencoh.py 0.00075 0.00975 0.0157 2600 --hydrogen > _coh-H.inp

4. Generate the solid-to-cohesive map file (`H-3cohesive.map`)

        $ abaqus datacheck job=H-3cohesive
        $ abaqus python ../../abapython/mkmap.py H-3cohesive

5. Perform the cohesive analysis

        $ abaqus continue job=H-3cohesive user=conc.f

6. Post-process results

        $ abaqus python ../../abapython/saveh.py H-3cohesive

Reference values for the expected results can be found in the `results`
directory.