---
title: "arbitrary cross-section analysis in python"
excerpt: "an open source python program performing cross-section analysis on arbitrary sections"
category : [Computational Mechanics, Finite Element Analysis]
tags : [section properties, structural engineering, programming]
header:
  overlay_image: /assets/images/posts/2017-11-13-cross-section-analysis/torsion-stress.png
  overlay_filter: 0.3 # same as adding an opacity of 0.5 to a black background
  image_description: "Torsion stress in an I-section."
  caption: # "" # for photo credit
  teaser: /assets/images/posts/2017-11-13-cross-section-analysis/torsion-stress.png
---

# Introducing a free, rigorous cross-section analysis tool

###### - *check out the program and its documentation [here](https://github.com/robbievanleeuwen/section-properties).*


Over the past few months I have been creating and refining a program in python that calculates structural properties for any cross-section imaginable and displays the internal stresses on the section resulting from any combination of design actions. It is finally at a point where I'm ready to share it and its output has been verified through comparison of section property data for standard steel sections.

The motivation for the creation of this software is the lack of a **free** and rigorous analysis tool that can be used to calculate section properties for complex, built up sections. In developing the program, I also realised that the stress verification of cross-sections subject to multiple design actions can be streamlined through the visualisation of stress contour plots.

>This post is intended to introduce the program and highlight its capabilities and simplicity of operation. Later posts will detail the theory behind the calculation of each section property and reveal the inner workings of the program.

### Calculated cross-section properties

The program applies the finite element method to calculate the following cross-section properties:

* Area Properties: Area, elastic centroid, first and second moments of area, elastic section moduli, radii of gyration, principal bending axis rotation angle, second moments of area, elastic section moduli and radii of gyration about the principal bending axis, elastic centroid (for bending about the centroidal and principal axes), elastic section moduli (for bending about the centroidal and principal axes), corresponding shape factors.

* Warping Properties: St. Venant torsion constant, warping constant, shear centre, shear areas (for shear loading about global and principal axes).

### Stress Analysis

The program allows the user to enter the following design actions:

* Axial force
* Bending moments about the x and y axes
* Bending moments about the principal axes
* Torsion moment
* Shear forces in the x and y directions

The following stress plots will then be generated: axial, bending, torsion, transverse shear, combined normal, combined shear and von Mises.

### Try it yourself!

If you have a basic understanding of python, the program is very simple to use. Head over to [GitHub](https://github.com/robbievanleeuwen/section-properties) to download the program and check out the documentation. If you have trouble with anything, including getting the program up and running, performing an analysis or find any bugs, feel free to leave a comment below or flick me an [email](mailto:robbie.vanleeuwen@gmail.com).

### Example analysis: 150 x 90 x 12 UA

A cross-section analysis is performed on an unequal angle section (150D x 100W x 12THK). A mesh size of 2.5 mm<sup>2</sup> is used. The following design actions are applied to determine the cross-section stresses:

* Axial force = -50 kN
* Bending moment about x-axis = 3 kN.m
* Bending moment about y-axis = 5 kN.m
* Torsion moment = 1 kN.m
* Transverse shear force in the x-direction = 5 kN
* Transverse shear force in the y-direction = 3 kN

The following code executes the analysis:

```python
import main
import sectionGenerator

# Create angle section
(points, facets, holes) = sectionGenerator.AngleToe(d=90, b=150, t=12, r_root=10, r_toe=5, n_r=16)

# Perform cross-sectional analysis
mesh = main.crossSectionAnalysis(points, facets, holes, meshSize=2.5, nu=0.3)

# Perform stress analysis
main.stressAnalysis(mesh, Nzz=-50e3, Mxx=3e6, Myy=5e6, M11=0, M22=0, Mzz=1e6, Vx=5e3, Vy=3e3)
```

The mesh used for warping analysis is shown below:

{% include figure image_path="/assets/images/posts/2017-11-13-cross-section-analysis/mesh.png" alt="mesh" caption="Mesh used for warping analysis with maximum element area of 2.5 mm<sup>2</sup>." %}

The centroidal locations are shown in the figure below:

{% include figure image_path="/assets/images/posts/2017-11-13-cross-section-analysis/centroid.png" alt="centroids" caption="Principal bending axis and various centroidal locations." %}

Selected annotated section property output is displayed below:

```
Area = 2746.80182627
cx = 50.9905249131 (x-centroid)
cy = 21.2264755268
Ixx_c = 1719026.97045 (Second moment of area about centroidal x-axis)
Iyy_c = 6287251.54258
Ixy_c = -1886208.80899
Zxx+ = 24995.4758553 (Elastic section modulus about the centroidal x-axis, considering the top fibre)
Zxx- = 80985.0400405
Zyy+ = 63501.5137396
Zyy- = 123302.349864
rx_c = 25.016565267 (Radius of gyration about the centroidal x-axis)
ry_c = 47.8428182325
phi = -109.774864162 (Principal bending axis rotation angle)
I11_c = 6965393.90567 (Second moment of area about the 11-axis)
I22_c = 1040884.60736
Z11+ = 69409.4336954
Z11- = 97758.0099588
Z22+ = 27961.1431946
Z22- = 20764.843276
r1_c = 50.3569220566
r2_c = 19.4664890305
Sxx = 45729.6667122 (Plastic section modulus about the x-axis)
Syy = 113546.98153
S11 = 121036.527767
S22 = 43763.8376059
J = 135336.217483 (Torsion constant)
Iw = 159998036.954 (Warping constant)
x_s,e = -42.9105894905 (x-shear centre)
y_s,e = -15.0990405213
A_s,x = 1538.85221978 (Shear area for loading in the x-direction)
A_s,y = 855.667113775
A_s,11 = 882.326380418
A_s,22 = 1459.5422681
```

Selected stress analysis plots are shown below:

{% include figure image_path="/assets/images/posts/2017-11-13-cross-section-analysis/bending.png" alt="bending" caption="Contour plot showing bending stress." %}

{% include figure image_path="/assets/images/posts/2017-11-13-cross-section-analysis/torsion.png" alt="torsion" caption="Vector plot showing torsion stress." %}

{% include figure image_path="/assets/images/posts/2017-11-13-cross-section-analysis/shear.png" alt="shear" caption="Contour plot showing shear stress." %}

{% include figure image_path="/assets/images/posts/2017-11-13-cross-section-analysis/vonMises.png" alt="von Mises" caption="Contour plot showing von Mises stress." %}
