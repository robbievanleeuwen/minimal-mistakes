---
title: "calculating section properties - part 1"
excerpt: "a description of the calculation of area related properties."
category : Finite Element Analysis
tags : [programming, computational mechanics, section properties]
header:
  overlay_image: /assets/images/posts/2018-02-11-calculating-section-properties-part1/teaser.jpg
  overlay_filter: 0.3 # same as adding an opacity of 0.5 to a black background
  image_description:
  caption: # "" # for photo credit
  teaser: /assets/images/posts/2018-02-11-calculating-section-properties-part1/teaser.jpg
toc: true
toc_label: Table of Contents
toc_sticky: true
---

>This blog post presents some of the finite element theory used in the section properties program in a simplified and easy to understand manner. It is worth noting that the notions used in the section properties program are the same that are applied in commercial finite element solvers. For a more in-depth overview of the section properties program, check out the paper I wrote [here](/assets/cross-section.pdf).

Building on the preliminaries covered in the previous blog post [[finite element preliminaries](/finite%20element%20analysis/finite-element-preliminaries/)], we can now quite easily calculate the area related section properties. This blog post will cover the computation of the following properties: *cross-section area, first moments of area, second moments of area, area centroids, radii of gyration, elastic section moduli and principal axis properties.* A code extract will follow each explanation, showing the implementation of the theory.

The most important formula to remember from the previous post is shown below:

$$
\int_{\Omega} f(x,y) \, d\Omega = \sum_e \sum_{i=1}^n w_i \, f(x_i,y_i) \, J_e
$$

This expression starts with the integration of a function $$\, f(x,y)$$ over the domain of the cross section $$\Omega$$ and converts it to a discrete sum which can be evaluated at each Gauss point $$i$$ within each element $$e$$. We'll be using this formula in the calculation of most section properties.

## Cross-section Area

The area of a cross-section is given by:

$$
A = \int_A \, dx \, dy = \sum_e \sum_{i=1}^n w_i J_e
$$

Because the Jacobian for our quadratic triangular element is constant, only one Guass point needs to be used for each element. The above formula can be implemented in python with the following code:

```python
# Initialise area variable
area = 0

# Loop through all the elements in the mesh
for el in elements:
  # Determine the Gauss points for 1pt Gaussian quadrature
  gp = gauss_points(1)
  # Determine the shape functions, partial derivatives and Jacobian for the
  # current Gauss point and element described by coordinates xy
  (N, B, j) = shape_function(xy, gp)

  # Evaluate the integral at the current Gauss point and add to the total. Note:
  # gp[0] is the Gauss point weight, and j is the determinant of the Jacobian
  area += gp[0] * j

print "Area = {}".format(area)
```

## First Moments of Area

The first moments of area are defined as:

$$
Q_x = \int_A y \, dA = \sum_e \int_A \textbf{N} \textbf{y}_e \, dA \\
Q_y = \int_A x \, dA = \sum_e \int_A \textbf{N} \textbf{x}_e \, dA
$$

Discretising the above expression results in the following finite element implementation:

$$
Q_x = \sum_e \sum_{i=1}^n w_i \textbf{N}_i \textbf{y}_e J_e \\
Q_y = \sum_e \sum_{i=1}^n w_i \textbf{N}_i \textbf{x}_e J_e
$$

This time, the function to be evaluated is a quadratic function (owing to the quadratic shape functions $$\textbf{N}_i$$) and as a result three Gauss points need to be used. The following code shows a python implementation of the above expressions:

```python
# Initialise first moment of area variables
qx = 0
qy = 0

# Loop through all the elements in the mesh
for el in elements:
  # Determine the Gauss points for 3pt Gaussian quadrature
  gps = gauss_points(3)

  # Loop through each Gauss point for the current element
  for gp in gps:
    # Determine the shape functions, partial derivatives and Jacobian for the
    # current Gauss point and element described by coordinates xy
    (N, B, j) = shape_function(xy, gp)

    # Evaluate the integral at the current Gauss point and add to the total.
    # Note: gp[0] is the Gauss point weight, xy[1,:] are the y-coordinates of
    # the current element, xy[0,:] are the x-coordinates of the current element
    # and j is the determinant of the Jacobian
    qx += gp[0] * np.dot(N, np.transpose(xy[1,:])) * j
    qy += gp[0] * np.dot(N, np.transpose(xy[0,:])) * j

print "qx = {}".format(qx)
print "qy = {}".format(qy)
```

## Second Moments of Area

The second moments of area are defined as:

$$
I_{xx} = \int_A y^2 \, dA = \sum_e \int_A (\textbf{N} \textbf{y}_e)^2 \, dA \\
I_{yy} = \int_A x^2 \, dA = \sum_e \int_A (\textbf{N} \textbf{x}_e)^2 \, dA \\
I_{xy} = \int_A xy \, dA = \sum_e \int_A \textbf{N} \textbf{x}_e \textbf{N} \textbf{y}_e  \, dA
$$

Discretising the above expression results in the following finite element implementation:

$$
I_{xx} = \sum_e \sum_{i=1}^n w_i (\textbf{N}_i \textbf{y}_e)^2 J_e \\
I_{yy} = \sum_e \sum_{i=1}^n w_i (\textbf{N}_i \textbf{x}_e)^2 J_e \\
I_{xy} = \sum_e \sum_{i=1}^n w_i \textbf{N} \textbf{x}_e \textbf{N} \textbf{y}_e J_e \\
$$

This time, the function to be evaluated is a quartic function and as a result six Gauss points need to be used. The following code shows a python implementation of the above expressions:

```python
# Initialise first moment of area variables
ixx = 0
iyy = 0
ixy = 0

# Loop through all the elements in the mesh
for el in elements:
  # Determine the Gauss points for 3pt Gaussian quadrature
  gps = gauss_points(6)

  # Loop through each Gauss point for the current element
  for gp in gps:
    # Determine the shape functions,{ p}artial derivatives and Jacobian for the
    # current Gauss point and element described by coordinates xy
    (N, B, j) = shape_function(xy, gp)

    # Evaluate the integral at the current Gauss point and add to the total.
    # Note: gp[0] is the Gauss point weight, xy[1,:] are the y-coordinates of
    # the current element, xy[0,:] are the x-coordinates of the current element
    # and j is the determinant of the Jacobian
    ixx += gp[0] * (np.dot(N, np.transpose([1,:]))) ** 2 * j
    iyy += gp[0] * (np.dot(N, np.transpose([0,:]))) ** 2 * j
    ixy += (gp[0] * np.dot(N, np.transpose([0,:])) *
              np.dot(N, np.transpose([1,:])) * j)

print "ixx = {}".format(ixx)
print "iyy = {}".format(iyy)
print "iyy = {}".format(ixy)
```

The above moments of inertia have been calculated about the global coordinate system, however we are usually more interested in the moments of inertia about the centroidal axis. The global moments of inertia can be transformed to the centroidal axis using the following expressions:

$$
I_{\overline{xx}} = I_{xx} - \frac{(Q_x)^2}{A} \\
I_{\overline{yy}} = I_{yy} - \frac{(Q_y)^2}{A} \\
I_{\overline{xy}} = I_{xy} - \frac{Q_x Q_y}{A}
$$

This is easily implemented in python as follows:

```python
ixx_c = ixx - qx * qx / area
iyy_c = iyy - qy * qy / area
ixy_c = ixy - qx * qy / area
```

## Area Centroids, Radii of Gyration and Elastic Section Moduli

These properties can be directly computed from the properties we have already computed. The area centroids are defined as follows:

$$
x_c = \frac{Q_y}{A} \\
y_c = \frac{Q_x}{A}
$$

The radii of gyration are defined as follows:

$$
r_x = \sqrt{\frac{I_{\overline{xx}}}{A}} \\
r_y = \sqrt{\frac{I_{\overline{yy}}}{A}}
$$

The elastic section moduli are defined as follows:

$$
Z_{xx}^+ = \frac{I_{\overline{xx}}}{y_{max} - y_c} \\
Z_{xx}^- = \frac{I_{\overline{xx}}}{y_c - y_{min}} \\
Z_{yy}^+ = \frac{I_{\overline{yy}}}{x_{max} - x_c} \\
Z_{yy}^- = \frac{I_{\overline{yy}}}{x_c - x_{min}}
$$

The above expressions can be directly computed after executing the previous python code snippets:

```python
# calculate area centroids
x_c = qy / area
y_c = qx / area

# calculate radii of gyration
r_x = (ixx_c / area) ** 0.5
r_y = (iyy_c / area) ** 0.5

# calculate the elastic section moduli
xmax = nodes[:,0].max()
xmin = nodes[:,0].min()
ymax = nodes[:,1].max()
ymin = nodes[:,1].min()

zxx_plus = ixx_c / (ymax - x_c)
zxx_minus = ixx_c / (y_c - ymin)
zyy_plus = iyy_c / (xmax - x_c)
zyy_minus = iyy_c / (x_c - xmin)
```

## Principal Section Properties

The principal bending axes are determined by calculating the principal moments of inertia [1]:

$$
I_{11} = \frac{I_{\overline{xx}} + I_{\overline{yy}}}{2} + \Delta \\
I_{22} = \frac{I_{\overline{xx}} + I_{\overline{yy}}}{2} - \Delta
$$

where $$\Delta$$ is defined as follows:

$$
\Delta = \sqrt{\left(\frac{I_{\overline{xx}} - I_{\overline{yy}}}{2}\right)^2 + {I_{\overline{xy}}}^2}
$$

Once the principal moments of inertia are calculated, the angle between the x-axis and the axis belonging to the largest principal moment of inertia can be computed as follows:

$$
\phi = {\tan}^{-1} \left(\frac{I_{\overline{xx}} - I_{11}}{I_{\overline{xy}}}\right)
$$

The principal radii of gyration can be easily calculated using the new principal moments of inertia. To calculate the principal elastic section moduli, the distance to the extreme fibres in the principal directions needs to be determined. Firstly, a function is written to convert a coordinate in the xy-axis into a principal axis coordinate:

```python
def principal_coordinate(phi, x, y):
    """
    Determines the coordinates of the cartesian point *(x, y)* in the
    principal axis system given an axis rotation angle phi.
    """

    # convert principal axis angle to radians
    phi_rad = phi * np.pi / 180

    # form rotation matrix
    R = np.array([[np.cos(phi_rad), np.sin(phi_rad)],
                  [-np.sin(phi_rad), np.cos(phi_rad)]])

    # calculate rotated x and y coordinates
    x_rotated = R.dot(np.array([x, y]))

    return (x_rotated[0], x_rotated[1])
```

Now the extreme fibre positions can be calculated by looping through all the nodes and determining the extreme values:

```python
# loop through all points in the mesh
for (i, point) in enumerate(nodes):
    # determine the coordinate of the point wrt the principal axis
    (x1, y2) = (principal_coordinate(phi, point[0] - x_c, point[1] - y_c))

    # initialise min, max variables
    if i == 0:
        x1max = x1
        x1min = x1
        y2max = y2
        y2min = y2

    # update the min and max where necessary
    x1max = max(x1max, x1)
    x1min = min(x1min, x1)
    y2max = max(y2max, y2)
    y2min = min(y2min, y2)
```

Finally the principal elastic section moduli can be calculated:

```python
# evaluate principal elastic section moduli
z11_plus = i11_c / abs(y2max)
z11_minus = i11_c / abs(y2min)
z22_plus = i22_c / abs(x1max)
z22_minus = i22_c / abs(x1min)
```

The next blog post will focus on the calculation of the plastic section moduli, a tricky topic which deserves its own separate post.

<script type="text/javascript" src="//downloads.mailchimp.com/js/signup-forms/popup/unique-methods/embed.js" data-dojo-config="usePlainJson: true, isDebug: false"></script><script type="text/javascript">window.dojoRequire(["mojo/signup-forms/Loader"], function(L) { L.start({"baseUrl":"mc.us19.list-manage.com","uuid":"541c65ecb1b23522bcf1300db","lid":"b7a47b4e83","uniqueMethods":true}) })</script>

## References
1. W.D. Pilkey, Analysis and Design of Elastic Beams: Computational Methods, John Wiley & Sons, Inc., New York, 2002.
