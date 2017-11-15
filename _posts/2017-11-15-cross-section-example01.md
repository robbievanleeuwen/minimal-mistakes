---
title: "cross-section analysis examples"
excerpt: "analysis of built-up sections, a channel section and a split hollow section"
category : [Computational Mechanics, Finite Element Analysis]
tags : [section properties, structural engineering, programming]
header:
  overlay_image: /assets/images/posts/2017-11-15-cross-section-example01/header.png
  overlay_filter: 0.3 # same as adding an opacity of 0.5 to a black background
  image_description: "Built-up section."
  caption: # "" # for photo credit
  teaser: /assets/images/posts/2017-11-15-cross-section-example01/header.png
---

# Cross-section Analysis Examples

## Contents

* [Split 200 x 100 x 6 RHS](#rhs)
* [250 PFC](#pfc)
* [Built-up 200UB25 + 150 x 100 x 9 RHS](#builtup1)
* [Built-up 50 x 5 SHS + 100 x 50 x 5 RHS + 200 x 6 Flat](#builtup2)

<a name="rhs"></a>
## Split 200 x 100 x 6 RHS

A box section provides torsional stiffness by providing a closed path for shear stresses to flow at a considerable distance from a rotational centre. Preventing this enclosed path dramatically reduces the torsional rigidity of the section. This is illustrated through the analysis and of a 200 x 100 x 6 RHS, and a comparison to that of the same section with a 1 mm wide cut in one of the sides. A torsion of 1 kN.m is applied to both sections.

The analysis can be carried out with the following python commands:

```python
import main
import sectionGenerator

# Closed RHS
(points, facets, holes) = sectionGenerator.RHS(100, 200, 6, 15, 16)
mesh1 = main.crossSectionAnalysis(points, facets, holes, meshSize=2.5, nu=0.3)

# Open RHS
(points, facets, holes) = sectionGenerator.RHS_Split(100, 200, 1, 6, 15, 16)
mesh2 = main.crossSectionAnalysis(points, facets, holes, meshSize=2.5, nu=0.3)

# Perform stress analysis
main.stressAnalysis(mesh1, Nzz=0, Mxx=0, Myy=0, M11=0, M22=0, Mzz=1e6, Vx=0, Vy=0)
main.stressAnalysis(mesh2, Nzz=0, Mxx=0, Myy=0, M11=0, M22=0, Mzz=1e6, Vx=0, Vy=0)
```

### Closed RHS results

The torsion constant was calculated by the python program to be J = 14.236 x 10<sup>6</sup> mmmm<sup>4</sup>.

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/RHS_mesh.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/RHS_mesh.png" alt="RHS mesh.">
  </a>
  <figcaption>Mesh discretisation for the closed 200 x 100 x 6 RHS.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/RHS_stress.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/RHS_stress.png" alt="RHS stress.">
  </a>
  <figcaption>Shear stress due to torsion for the closed 200 x 100 x 6 RHS.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/RHS_vectors.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/RHS_vectors.png" alt="RHS vectors.">
  </a>
  <figcaption>Shear stress vectors due to torsion for the closed 200 x 100 x 6 RHS at the top right of the section.</figcaption>
</figure>

### Split RHS results

The torsion constant was calculated by the python program to be J = 39.648 x 10<sup>3</sup> mm<sup>4</sup>, approximately 360 times less stiff than the closed section.

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/RHS_Split_mesh.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/RHS_Split_mesh.png" alt="RHS_Split mesh.">
  </a>
  <figcaption>Mesh discretisation for the split 200 x 100 x 6 RHS.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/RHS_Split_stress.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/RHS_Split_stress.png" alt="RHS_Split stress.">
  </a>
  <figcaption>Shear stress due to torsion for the split 200 x 100 x 6 RHS, approximately 30 times higher than the closed section.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/RHS_Split_vectors1.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/RHS_Split_vectors1.png" alt="RHS_Split vectors.">
  </a>
  <figcaption>Shear stress vectors due to torsion for the split 200 x 100 x 6 RHS at the top right of the section. The shear stress now has to flow within the thickness of the wall, rather than around the entire section.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/RHS_Split_vectors2.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/RHS_Split_vectors2.png" alt="RHS_Split vectors.">
  </a>
  <figcaption>Shear stress vectors due to torsion for the split 200 x 100 x 6 RHS adjacent to the split.</figcaption>
</figure>

<a name="pfc"></a>
## 250 PFC

The analysis of a 250 PFC (250 mm deep parallel flange channel) can be carried out with the following python commands:

```python
import main
import sectionGenerator

(points, facets, holes) = sectionGenerator.PFC(d=250, b=90, tf=15, tw=8, r=12, n_r=16)
mesh = main.crossSectionAnalysis(points, facets, holes, meshSize=4, nu=0.3)
```

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/PFC_centroids.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/PFC_centroids.png" alt="PFC centroids">
  </a>
  <figcaption>Mesh discretisation for the 250 PFC with principal axes and centroids.</figcaption>
</figure>

This produces the following output for the warping dependent properties:

$$
J = 237.85 \times 10^3 \textrm{mm}^4 \\
I_w = 35.556 \times 10^9 \textrm{mm}^6 \\
x_s = -57.758 \textrm{mm}
$$

These results, which use a mesh size of 4 mm<sup>2</sup>, can be compared to the tabulated values in the OneSteel catalogue, and simple hand calculations. The OneSteel catalogue gives the following properties:

$$
J = 248 \times 10^3 \textrm{mm}^4 \\
I_w = 35.9 \times 10^9 \textrm{mm}^6 \\
x_s = -58.5 \textrm{mm}
$$

A mesh refinement study using the python cross-section program shows that the OneSteel values for the torsion constant is slightly (4%) overestimated, whereas the warping constant and shear centre show closer convergence. The numerical results obtained from the python program can be compared to simple hand calculations:

* Torsion Constant [1]

$$
J \approx \sum \left(\frac{b t^3}{3}\right) = \frac{2(90)(15)^3 + (220)(8)^3}{3} = 240.0 \times 10^3 \textrm{mm}^4
$$

* Warping Constant [1]

$$
I_w = \frac{ {b_f}^3 t_f {b_w}^2 }{48} \left(8 - \frac{3 b_f t_f {b_w}^2 }{I_x}\right) = 40.29 \times 10^9 \textrm{mm}^6
$$

* Shear Centre [2] (from centre of web to shear centre):

$$
x_s = -x_c + \frac{t_w}{2} -\frac{3 t_f (b_f - 0.5t_w)^2 }{6 (b_f - 0.5t_w) t_f + (d - t_f) t_w} = 59.20 \textrm{mm}
$$

which gives a distance to the centroid of -59.2 mm.

The above hand calculations align well with the results from the python program and the OneSteel catalogue.

<a name="builtup1"></a>
## Built-up 200UB25 + 150 x 100 x 9 RHS

A steel section is fabricated by placing a 150 x 100 x 9 RHS on its side on top of a 200UB25. The section is subjected to a major axis bending moment of 50 kN.m, a torsion moment of 10 kN.m and a y-direction shear force of -25 kN.

The analysis can be carried out by using the section builder function with the following python commands:

```python
import main
import sectionGenerator

(UBpoints, UBfacets, UBholes) = sectionGenerator.ISection(203, 133, 7.8, 5.8, 8.9, 8)
(RHSpoints, RHSfacets, RHSholes) = sectionGenerator.RHS(100, 150, 9, 22.5, 8)

UB = {  'points': UBpoints,
        'facets': UBfacets,
        'holes' : UBholes,
        'x'     : -0.5 * 133,
        'y'     : 0 }

RHS = { 'points': RHSpoints,
        'facets': RHSfacets,
        'holes' : RHSholes,
        'x'     : -75,
        'y'     : 203 }

(points, facets, holes) = sectionGenerator.combineShapes([UB, RHS])
mesh = main.crossSectionAnalysis(points, facets, holes, meshSize=5, nu=0.3)

# Perform stress analysis
main.stressAnalysis(mesh, Nzz=0, Mxx=50e6, Myy=0, M11=0, M22=0, Mzz=10e6, Vx=0, Vy=-25e3)
```

The mesh, centroids and stress contours are shown below:

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built1_centroids.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built1_centroids.png" alt="Built-up 1 centroids">
  </a>
  <figcaption>Mesh discretisation for the built-up section with principal axes and centroids.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built1_bending.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built1_bending.png" alt="Built-up 1 bending stress">
  </a>
  <figcaption>Bending stress.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built1_torsion.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built1_torsion.png" alt="Built-up 1 torsion">
  </a>
  <figcaption>Shear stress due to torsion.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built1_shear.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built1_shear.png" alt="Built-up 1 shear">
  </a>
  <figcaption>Shear stress due to transverse shear force.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built1_vm.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built1_vm.png" alt="Built-up 1 vm">
  </a>
  <figcaption>von Mises stress.</figcaption>
</figure>

<a name="builtup2"></a>
## Built-up 50 x 5 SHS + 100 x 50 x 5 RHS + 200 x 6 Flat

A zed shaped steel section is fabriacted by welding a 50 x 5 SHS and a 100 x 50 x 5 RHS to a 200 x 6 flat section. The section is subjected to a major axis bending moment of 10 kN.m, a torsion moment of 5 kN.m and a y-direction shear force of -15 kN.

The analysis can be carried out by using the section builder function with the following python commands:

```python
import main
import sectionGenerator

(Flatpoints, Flatfacets, Flatholes) = sectionGenerator.Flat(200, 6)
(SHSpoints, SHSfacets, SHSholes) = sectionGenerator.RHS(50, 50, 5, 12.5, 8)
(RHSpoints, RHSfacets, RHSholes) = sectionGenerator.RHS(50, 100, 5, 12.5, 8)

Flat = {'points': Flatpoints,
        'facets': Flatfacets,
        'holes' : Flatholes,
        'x'     : 50,
        'y'     : -100}

SHS = {'points' : SHSpoints,
        'facets': SHSfacets,
        'holes' : SHSholes,
        'x'     : 56,
        'y'     : -100}

RHS = {'points' : RHSpoints,
        'facets': RHSfacets,
        'holes' : RHSholes,
        'x'     : -50,
        'y'     : 50}

section = [Flat, RHS, SHS]

(points, facets, holes) = sectionGenerator.combineShapes(section)
mesh5 = main.crossSectionAnalysis(points, facets, holes, meshSize=1.5, nu=0.3)

# Perform stress analysis
main.stressAnalysis(mesh5, Nzz=0, Mxx=10e6, Myy=0, M11=0, M22=0, Mzz=5e6, Vx=0, Vy=-15e3)
```

The mesh, centroids and stress contours are shown below:

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built2_centroids.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built2_centroids.png" alt="Built-up 2 centroids">
  </a>
  <figcaption>Mesh discretisation for the built-up section with principal axes and centroids.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built2_bending.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built2_bending.png" alt="Built-up 2 bending stress">
  </a>
  <figcaption>Bending stress.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built2_torsion.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built2_torsion.png" alt="Built-up 2 torsion">
  </a>
  <figcaption>Shear stress due to torsion.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built2_shear.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built2_shear.png" alt="Built-up 2 shear">
  </a>
  <figcaption>Shear stress due to transverse shear force.</figcaption>
</figure>

<figure>
  <a href="/assets/images/posts/2017-11-15-cross-section-example01/built2_vm.png">
    <img src="/assets/images/posts/2017-11-15-cross-section-example01/built2_vm.png" alt="Built-up 2 vm">
  </a>
  <figcaption>von Mises stress.</figcaption>
</figure>

## References
1. AS 4100-1998: Steel Structures
2. W.D.Pilkey, Analysis and Design of Elastic Beams: Computational Methods, John Wiley & Sons, Inc., New York, 2002.
