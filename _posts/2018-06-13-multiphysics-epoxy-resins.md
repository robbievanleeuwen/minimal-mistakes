---
title: "a multiphysics numerical framework for epoxy resins"
excerpt: "a short overview of my masters thesis."
category : Finite Element Analysis
tags : [programming, computational mechanics]
header:
  overlay_image: /assets/images/posts/2018-06-13-multiphysics-epoxy-resins/turbine-header.jpg
  overlay_filter: 0.3 # same as adding an opacity of 0.5 to a black background
  image_description:
  caption: # "" # for photo credit
  teaser: /assets/images/posts/2018-06-13-multiphysics-epoxy-resins/turbine-teaser.jpg
toc: true
toc_label: Table of Contents
toc_sticky: true
---

>This post presents an overview of my Masters thesis in Computational Mechanics which I completed in June 2018. If you are interested in reading my thesis, you can find it on the [TU Delft repository](https://repository.tudelft.nl/islandora/object/uuid%3A86d469f3-0c44-4f45-9396-ec296d87031f?collection=education) or you can download the entire report [here](/assets/Thesis_RJvanLeeuwen.pdf).

## Overview

Epoxy resins are increasingly used in critical structural components with widespread applications in the transportation, construction and energy industries. One such application is in wind turbine blades, where the structural components are predominantly constructed from glass/epoxy composites, or fibreglass. In offshore installations, these blades are subject to a wide range of environmental conditions, most notably large variations in humidity and temperature. Both moisture and increased temperatures have been observed to have a significant impact on the stiffness and strength of epoxy resins. These environmental effects, coupled with complex time dependent mechanical behaviour, means that the accurate prediction of the structural performance of epoxy resins has not yet been fully described.

<figure>
<div align="center">
  <a href="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/offshore-wind.jpg">
    <img src="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/offshore-wind.jpg" alt="Offshore wind turbine." style="width:80%;height:80%;">
  </a>
</div>
  <figcaption>Wind turbine blades in offshore installations are subjected to significant dynamic fatigue loading, large variations in temperature and increases in moisture content.</figcaption>
</figure>

In my thesis I presented a multiphysics framework for the simulation of hygrothermal ageing in epoxy resins. The constitutive model consisted of a non-linear viscoelastic and viscoplastic mechanical model, physically coupled with a Fourier heat conduction model and a Fickian diffusion model. Material degradation based on a glass transition surface was implemented to describe the multi-state behaviour of epoxy resins. A number of numerical benchmark tests and case studies were performed using a finite element implementation of the numerical framework and it was shown that the multiphysics framework can capture the characteristic mechanical and hygrothermal ageing behaviour exhibited by epoxy resins. In this post I will highlight the key features of each component of the multiphysics model and present some of the interesting results.

<figure>
<div align="center">
  <a href="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/multiphys.png">
    <img src="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/multiphys.png" alt="Multiphysics Framework." style="width:100%;height:80%;">
  </a>
</div>
  <figcaption>Schematic diagram of the multiphysics model. A one way coupling was used between the physical models and the energy dissipated through mechanical deformation passed to the heat model in the proceeding time increment.</figcaption>
</figure>

## Multiphysics Numerical Framework

### Heat Conduction and Moisture Diffusion

Both head conduction and moisture diffusion follow a similar formulation based on the governing laws of Fourier heat conduction and Fickian diffusion. Fourier heat conduction which states that heat flux is proportional to the negative gradient of the temperature:

$$
q = -\kappa \nabla T
$$

Fickian diffusion is analogous to Fourier heat conduction in that the moisture flux is proportional to the negative gradient of the moisture concentration:

$$
j = -D_\omega \nabla \omega
$$

When combined with conservation of energy, the strong formulation of the transport problem can be obtained. Inline with finite element theory, the weak form can be obtained with consideration of the boundary conditions. The transient formulation of the transport processes produces the assembly of element level matrices and vectors. In this blog post, the finite element formulation of the heat conduction model is presented. For a full derivation of both formulations, the reader is referred to the full report referred to at the top of this page. The finite element formulation of the heat conduction model is:

$$
\mathcal{A}_e \, \bigg\{\left(\textbf{k}_{\text{h},e} + \textbf{h}_{\text{h},e} \right) T_e + \textbf{c}_{\text{h},e} \dot{T}_e = \textbf{r}_h + \textbf{r}_q + \textbf{r}_Q \bigg\}
$$

where $$\mathcal{A}_e$$ is the element assembly operator, $$T_e$$ is the element temperature, and the element level matrices and vectors are given by:

$$
\begin{align}
	\begin{aligned}
   		\textbf{k}_{\text{h},e} &= \int_{\Omega_e} \textbf{B}^{\mathrm{T}} \kappa \textbf{B} \, \mathrm{d} \Omega \\       
		\textbf{h}_{\text{h},e} &= \int_{\Gamma_{h,e}} \textbf{N}^{\mathrm{T}} h \textbf{N} \, \mathrm{d} \Gamma \\
		\textbf{c}_{\text{h},e} &= \int_{\Omega_e} \textbf{N}^{\mathrm{T}} \rho c \textbf{N} \, \mathrm{d} \Omega
  	\end{aligned}
  	&&
  	\begin{aligned}
   		\textbf{r}_h &= \int_{\Gamma_{h,e}} \textbf{N}^{\mathrm{T}} h T_\text{f} \, \mathrm{d} \Gamma \\
		\textbf{r}_q &= \int_{\Gamma_{q,e}} \textbf{N}^{\mathrm{T}} q_i \, \mathrm{d}\Gamma \\
		\textbf{r}_Q &= \int_{\Omega_e} \textbf{N}^{\mathrm{T}} Q_\text{int} \, \mathrm{d} \Omega
  	\end{aligned}
\end{align}
$$

### Mechanical Model

The mechanical model follows from the classical continuum small strain formulation, resulting in the following finite element formulation:

$$
\mathcal{A}_e \, \bigg\{\textbf{k}_{\text{m},e} \vec{u_e} = \textbf{f}_b + \textbf{f}_\text{t}\bigg\}
$$

where $$\vec{u_e}$$ is the element displacement vector, and the element level matrices and vectors are given by:

$$
\begin{align}
	\textbf{k}_{\text{m},e} &= \int_{\Omega_e} \textbf{B}^{\mathrm{T}} \textbf{D} \textbf{B} \, \mathrm{d} \Omega & \textbf{f}_b &= \int_{\Omega_e} \textbf{N}^{\mathrm{T}} \vec{b} \, \mathrm{d} \Omega & \textbf{f}_\text{t} &= \int_{\Gamma_{\text{t},e}} \textbf{N}^{\mathrm{T}} \vec{t_h} \, \mathrm{d}\Gamma
\end{align}
$$

Of main interest in this thesis is the determination of the material constitutive matrix for epoxy resins $$\textbf{D}$$ and the resulting stress vector $$\vec{\sigma}$$. The ingredients that contribute to this constitutive attempt to characterise the mechanical behaviour of epoxy resins and in this thesis consists of the following:

- Non-linear viscoelasticity
- Viscoplasticity
- Glass transition surface
- Material degradation

#### Non-linear Viscoelasticity

The non-linear viscoelasticity model was implemented to capture time dependent elastic behaviour as well as the non-linear recoverable elastic behaviour exhibited by epoxy resins. In this model the elastic stress is decomposed into a long term elastic component and a viscoelastic component:

$$
\vec{\sigma^\text{e}} (t) = \textbf{D}_{\infty} (t) \vec{\varepsilon^\text{e}} (t) + \vec{\sigma^\text{ve}} (t)
$$

The expression for the viscoelastic stress can be further decomposed into a volumetric component $$ p^\text{ve} (t)$$ and a deviatoric component $$\textbf{S}^\text{ve} (t)$$:

$$
\begin{align}
	p^\text{ve} (t) &= \sum_{i=1}^N \int_0^t \exp\left(-\frac{t-\tau}{\lambda_{i,k}}\right) \frac{d}{d \tau} \left[\frac{K_i(t) \varepsilon^\text{e}_\text{v}(t)}{g(\sigma,\zeta,\varepsilon_\text{eq}^\text{p})}\right] \, d \tau \\
	\textbf{S}^\text{ve} (t) &= \sum_{i=1}^N \int_0^t \exp\left(-\frac{t-\tau}{\lambda_{i,g}}\right) \frac{d}{d \tau} \left[\frac{2 G_i(t) \vec{\varepsilon^\text{e}_\text{d}}(t)}{g(\sigma,\zeta,\varepsilon_\text{eq}^\text{p})}\right] \, d \tau
\end{align}
$$

In the above equations the function $$g$$ introducing non-linearity into the viscoelastic model. The finite element implementation of the above formulation is further elaborated in the full report. Following from the finite element formulation of the stress vector, the tangent stiffness $$D$$ can also be derived.

#### Viscoplasticity

A viscoplasticity model was implemented to capture the time dependent plastic behaviour of epoxy resins. The formulation used in my thesis followed from the model used by Rocha et. al. [24]. In order to combine a non-linear viscoelasticity model with a viscoplasticity model, a coupled return mapping scheme was derived and implemented in the context of the Newton-Raphson method to ensure robust and efficient convergence. The essence behind this implementation is that the degree of non-linear viscoelasticity $$g$$ and the degree of plasticity, defined by the plastic multiplier $$\Delta \gamma$$, are determined using the coupled Jacobians of the iterative functions:

$$
\begin{equation}
	\begin{bmatrix}
		g^{k+1} \\ \Delta \gamma^{k+1}
	\end{bmatrix} = \begin{bmatrix}
		g^{k} \\ \Delta \gamma^{k}
	\end{bmatrix} - \begin{bmatrix}
		\dfrac{\partial \Gamma}{\partial g} & \dfrac{\partial \Gamma}{\partial \Delta \gamma} \\
		\dfrac{\partial \Phi}{\partial g} & \dfrac{\partial \Phi}{\partial \Delta \gamma}
	\end{bmatrix}^{-1} \begin{bmatrix}
		\Gamma(g^k, \Delta \gamma^k) \\ \Phi(g^k, \Delta \gamma^k)
	\end{bmatrix}
\end{equation}
$$

#### Glass Transition Surface

The idea of the glass transition surface is to relate both the temperature and moisture content of the epoxy resin to its material state. Conceptually an epoxy resin, or in general a polymer, can exist in three different material states: glassy, rubbery and mixed glassy-rubbery. Glassy behaviour occurs at lower temperatures and moisture contents and is characterised by stiffer and stronger material behaviour. On the other hand, rubbery  behaviour occurs at higher temperatures and moisture contents and is characterised by softer and weaker material behaviour. In this thesis the material state is related to a scalar parameter $$\zeta$$ which characterises the degree of glass transition:

- Glassy behaviour: $$\zeta = 0$$
- Mixed glassy-rubbery behaviour: $$0 < \zeta < 1$$
- Rubbery behaviour: $$\zeta = 1$$

The simplicity of the glass transition model is that the state of the material can be described by the temperature and moisture content of the epoxy resin and can easily be calibrated to a specific epoxy resin system.

<figure>
<div align="center">
  <a href="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/transition-diagram.png">
    <img src="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/transition-diagram.png" alt="Glass Transition Diagram." style="width:80%;height:80%;">
  </a>
</div>
  <figcaption>Glass transition diagram in which the material state is derived from the temperature and moisture content of the epoxy resin.</figcaption>
</figure>

#### Material Degradation

The purpose of the degradation model is to relate the current state of the epoxy resin to its current material properties. The relevant material properties of the epoxy resin are assumed to be constant in the glassy and rubbery states, and are linearly interpolated based on the degree of glass transition $$\zeta$$ in the mixed state. The state dependent material properties is generalised as follows:

$$
\Theta (\zeta) = (1-\zeta) \Theta^\text{gla} + \zeta \Theta^\text{rub}
$$

While this method simplifies the material behaviour by imposing a constant property when the epoxy resin is in a glassy or rubbery state and a linearly interpolated property for the mixed state, this assumption has been found to give reasonably accurate results.

<figure>
<div align="center">
  <a href="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/model-graph.png">
    <img src="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/model-graph.png" alt="Degradation Model." style="width:80%;height:80%;">
  </a>
</div>
  <figcaption>Implementation of the material state dependent degradation model.</figcaption>
</figure>

## Multiphysics Fatigue Analysis

Using a finite element implementation of the above multiphysics model, a fatigue test was numerically performed on an epoxy resin sample. A dogbone sample, initially at room temperature, was surrounded by air at room temperature. A cyclic load was applied to the end of the specimen at a high strain rate in order to significantly increase the temperature of the epoxy resin. The multiphysics model formulated in my thesis is able to capture the competing processes of internal heat generation from viscous mechanical deformation and dissipative cooling by the surrounding fluid.

<figure>
<div align="center">
  <a href="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/dogbone3D.png">
    <img src="/assets/images/posts/2018-06-13-multiphysics-epoxy-resins/dogbone3D.png" alt="Fatigue Analysis." style="width:80%;height:80%;">
  </a>
</div>
  <figcaption>Mesh and boundary conditions used for the fatigue analysis.</figcaption>
</figure>

The simulation is summarised in the animation below, in which the temperature of the epoxy resin, the longitudinal stress distribution and the material state are plotted within the specimen. Also presented is the load displacement diagram and a plot of the temperature and degree of glass transition.

{% include video id="CrThw3o0iCI" provider="youtube" %}

The multiphysics framework is able to capture the glass transition that occurs due to the heat generated by mechanical deformation. This glass transition occurs first at the edges of the speciment where the stress concentration is the highest. A corresponding redistribution of stress towards the centre of the element accompanies this material state transition as the load moves away from the softening material at the edges. Eventually the entire central section of specimen transitions to a rubbery state and a uniform stress distribution is obtained. This characteristic cyclic softening is further highlighted in the load displacement plot in which a softening of the material response, entirely induced by cyclic loading, occurs after several thousand cycles.

## Conclusions

A numerical multiphysics model using the finite element framework was formulated for epoxy resins in order to capture complex time dependent mechanical and hygrothermal behaviour. The fatigue test illustrated that the model was able to capture many aspects of the multiphysical behaviour of epoxy resins, such as time dependent mechanical effects, cyclic relaxation and softening, and deformation induced glass transition.

Not mentioned in this blog, but covered in the full report, are further topics related to numerically describing the behaviour of epoxy resins:

- Assessment of the non-linear viscoelasticity model
- Experiments (DMA and creep tests) conducted on epoxy resin samples
- Mesh sensitivity study on a continuum damage model for epoxy resins
- Recommendations for future work related to the ideas presented in my thesis
