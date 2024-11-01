### Manifold
### Connections
##### Why is connection important --> bigger picture
> In differential geometry, the notion of “parallel” is defined by parallel vector fields. Parallel vector fields are then used to define and construct the idea of parallel transport.
1. Parallel Vector Fields
	$$\nabla_{\dot{\gamma}(t)} X(t)=0\ \qquad \forall$$
		- M be a smooth manifold and $\nabla$ be a connection on M
		- $\dot{\gamma}(t)$ is a vector that is tangent to the curve $\gamma(t)$
		- in differential geometry, we can only define the term “parallel” when we define it along a curve.
2. Parallel Transport
	- Parallel transport calculates the parallel vector field along the curve $\gamma(t)$ and then selects the element of the vector field that lies at the final point $\gamma(s)$
	- The parallel transport of a vector $\vec{v}$ along $\gamma$ from point $\gamma(a)$ to point $\gamma(s)$ is denoted by
![[Pasted image 20240830204151.png]]

### [What is Manifold Learning (Perpetual Enigma)](https://prateekvjoshi.com/2014/06/21/what-is-manifold-learning/#comments) 

**Dimensionality and Dimentionality Reduction**
- Dimensionality refers to the minimum number of coordinates needed to specify any point within a space or an object

**Linear Dimensionality Reduction**
- PCA is one of the methods where it finds the directions along which the data has maximum variance in addition to the relative importance of these directions. It will return 2 vectors (positive weights) in plane and 3rd orthogonal vector (weight of zero)
- PCA is useful for linear sub-spaces. But why?
	- PCA projects the data onto some low-dimensional surface. But this is restrictive in the sense that those surfaces are all linear. What if the the best representation lies in some weirdly shaped surface? PCA will totally miss that.

**What is Manifold Learning**
- mathematical understanding: [[#Manifold]] 
- Manifold allows us to representing Data on non-planar surface, thereby enabling us to describe high-dimensional data on low-dimensional manifold
