# Image-segmentation-using-clustering-algorithms

The goal is to find homogeneous regions in the images which hopefully belong to objects or object parts. Below is one of the test images and an example segmentation using K-means (In the right image, each segment is colored based on the average color within the segment).
<p align = "center">
<img width ="600" height="180", src = 
https://user-images.githubusercontent.com/45757826/57579422-c972af00-749b-11e9-99ca-8847a548e730.png>
</p>

# Table of Content
- [Data Description](#data-description)
- [Clustering Methods](#clustering-methods)
  - [K Means](#k-means)
  - [Gaussian Mixture Models](#gaussian-mixture-models)
  - [Mean Shift](#mean-shift)
  - [Modification](#modification)
- [Results](#results)
- [How To Use](#how-to-use)

# Data Description 
To segment the image, we first extract feature vectors from each pixel in the image (or a subset of pixels along a regular grid). In particular, we form a window centered around each pixel and extract a four-dimensional feature vector, <img src="https://tex.s2cms.ru/svg/%20%5Cinline%20x%20%3D%20%5Bu%3B%20v%3B%20c_%7By%7D%3B%20c_%7Bx%7D%5D%5ET%20" alt=" \inline x = [u; v; c_{y}; c_{x}]^T " />, where <img src="https://tex.s2cms.ru/svg/(u%3B%20v)" alt="(u; v)" /> are the *average chrominance values* (color without brightness) in the window, and <img src="https://tex.s2cms.ru/svg/%5Cinline%20(c_x%3B%20c_y)" alt="\inline (c_x; c_y)" /> is the *pixel location* (center of the window in x-y-coordinates). The feature values for the example image are shown below (each subplot
represents one feature dimension over the entire image). 

<p align = "center">
<img width ="600" height="450", src = 
https://user-images.githubusercontent.com/45757826/57579428-e3ac8d00-749b-11e9-8112-8cbfc3ac60fd.png>
</p>
 
# Clustering Methods

Next, three clustering algorithm is used to group the feature vectors, and cluster labels are assigned to each corresponding pixel forming the segmentation.

## K Means
Given a set of observations <img src="https://tex.s2cms.ru/svg/%20%5Cinline%20(x_1%2C%20x_2%2C%20%5Ccdots%2C%20x_%7Bn%7D)" alt=" \inline (x_1, x_2, \cdots, x_{n})" />, where each observation is a d-dimensional real vector, k-means clustering aims to partition the <img src="https://tex.s2cms.ru/svg/%20%5Cinline%20n" alt=" \inline n" /> observations into <img src="https://tex.s2cms.ru/svg/%20%5Cinline%20K%20(%E2%89%A4%20n)" alt=" \inline K (≤ n)" /> sets <img src="https://tex.s2cms.ru/svg/S%20%3D%20%7BS_1%2C%20S_2%2C%20%E2%80%A6%2C%20S_K%7D" alt="S = {S_1, S_2, …, S_K}" /> so as to minimize the within-cluster sum of squares (WCSS) (i.e. variance). Formally, the objective is to find:

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%5Cunderset%7BS%7D%7B%5Carg%20%5Cmin%7D%20%5Csum_%7Bi%7D%5E%7BK%7D%20%5Csum%20_%7Bx%20%5Cin%20S_i%7D%20%5Cleft%5CVert%20x-%5Cmu_%7Bi%7D%5Cright%20%5CVert%5E2" alt="\underset{S}{\arg \min} \sum_{i}^{K} \sum _{x \in S_i} \left\Vert x-\mu_{i}\right \Vert^2" />

</p>

Given an initial set of k means <img src="https://tex.s2cms.ru/svg/%5Cinline%20%5Cmu_1%2C%20%5Ccdots%2C%5Cmu_K" alt="\inline \mu_1, \cdots,\mu_K" /> , the algorithm proceeds by alternating between two steps:

- Assignment step: Assign each observation to the cluster whose mean has the least squared Euclidean distance, this is intuitively the "nearest" mean.[7] (Mathematically, this means partitioning the observations according to the Voronoi diagram generated by the means).

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%20S_%7Bi%7D%5E%7B(t)%7D%3D%7B%5Cbig%20%5C%7B%7Dx_%7Bp%7D%3A%7B%5Cbig%20%5C%7C%7Dx_%7Bp%7D-m_%7Bi%7D%5E%7B(t)%7D%7B%5Cbig%20%5C%7C%7D%5E%7B2%7D%5Cleq%20%7B%5Cbig%20%5C%7C%7Dx_%7Bp%7D-m_%7Bj%7D%5E%7B(t)%7D%7B%5Cbig%20%5C%7C%7D%5E%7B2%7D%5C%20%5Cforall%20j%2C1%5Cleq%20j%5Cleq%20k%7B%5Cbig%20%5C%7D%7D%2C%20" alt=" S_{i}^{(t)}={\big \{}x_{p}:{\big \|}x_{p}-m_{i}^{(t)}{\big \|}^{2}\leq {\big \|}x_{p}-m_{j}^{(t)}{\big \|}^{2}\ \forall j,1\leq j\leq k{\big \}}, " />
</p>
 
where each <img src="https://tex.s2cms.ru/svg/%5Cinline%20x_%7Bp%7D" alt="\inline x_{p}" /> is assigned to exactly one <img src="https://tex.s2cms.ru/svg/%5Cinline%20S%5E%7B(t)" alt="\inline S^{(t)" />.


- Update step: Calculate the new means (centroids) of the observations in the new clusters.

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%20%5Cmu_%7Bi%7D%5E%7B(t%2B1)%7D%3D%7B%5Cfrac%20%7B1%7D%7B%5Cleft%7CS_%7Bi%7D%5E%7B(t)%7D%5Cright%7C%7D%7D%5Csum%20_%7Bx_%7Bj%7D%5Cin%20S_%7Bi%7D%5E%7B(t)%7D%7Dx_%7Bj%7D%20" alt=" \mu_{i}^{(t+1)}={\frac {1}{\left|S_{i}^{(t)}\right|}}\sum _{x_{j}\in S_{i}^{(t)}}x_{j} " />
</p>

The algorithm has converged when the assignments no longer change. However, the algorithm does not guarantee to find the optimum.

## Gaussian Mixture Models

The GMM is weighted sum of Gaussian distributions, 

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%0A%20p(x%5Cmid%20%5Cvartheta)%3D%20%5Csum_%7Bk%3D1%7D%5E%7BK%7D%20%5Cpi_k%20%5Cmathcal%7BN%7D(x%20%5Cmid%20%5Cmu_k%2C%5CSigma_k)%2C%0A%20" alt="
 p(x\mid \vartheta)= \sum_{k=1}^{K} \pi_k \mathcal{N}(x \mid \mu_k,\Sigma_k),
 " />

</p>

where <img src="https://tex.s2cms.ru/svg/%5Cpi" alt="\pi" /> is the mixing coefficient that satisfies the probability constraints 
 
<p align = "center">

<img src="https://tex.s2cms.ru/svg/%09%5Csum_%7Bk%3D1%7D%5E%7BK%7D%5Cpi_k%3D1" alt="	\sum_{k=1}^{K}\pi_k=1" />
</p>

Now we introduce the hidden variable <img src="https://tex.s2cms.ru/svg/z%3D%5Bz_1%2C%5Ccdots%2Cz_K%5D%5ET" alt="z=[z_1,\cdots,z_K]^T" />, where <img src="https://tex.s2cms.ru/svg/z_k%3D1" alt="z_k=1" /> and the other entries equal to 0, 

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%20%09p(z)%3D%5Cpi_k%3D%5Cprod_%7Bk%3D1%7D%5E%7BK%7D%20%5Cpi_k%5E%7Bz_k%7D." alt=" 	p(z)=\pi_k=\prod_{k=1}^{K} \pi_k^{z_k}." />
</p>

Then we have the conditional probability of <img src="https://tex.s2cms.ru/svg/%20%5Cinline%20x" alt=" \inline x" /> given <img src="https://tex.s2cms.ru/svg/%20%5Cinline%20z" alt=" \inline z" /> as

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%20%09p(x%5Cmid%20z)%3D%5Cprod_%7Bk%3D1%7D%5E%7BK%7D%20%5Cmathcal%7BN%7D(x%5Cmid%5Cmu_k%2C%5CSigma_k)%5E%7Bz_k%7D." alt=" 	p(x\mid z)=\prod_{k=1}^{K} \mathcal{N}(x\mid\mu_k,\Sigma_k)^{z_k}." />
</p>

Then we rewrite it as

<p align = "center">

 <img src="https://tex.s2cms.ru/svg/%0A%20%09p(x)%3D%5Csum_%7Bz%7Dp(z)p(x%5Cmid%20z)%3D%20%5Csum_%7Bk%3D1%7D%5E%7BK%7D%20%5Cpi_k%20%5Cmathcal%7BN%7D(x%20%5Cmid%20%5Cmu_k%2C%5CSigma_k)%2C%0A%20" alt="
 	p(x)=\sum_{z}p(z)p(x\mid z)= \sum_{k=1}^{K} \pi_k \mathcal{N}(x \mid \mu_k,\Sigma_k), " /> 
</p>

The parameters <img src="https://tex.s2cms.ru/svg/%5Cinline%20%5Cvartheta%3D%20%5C%7B%5Cpi_k%2C%5Cmu_k%2C%5Csigma_k%5C%7D" alt="\inline \vartheta= \{\pi_k,\mu_k,\sigma_k\}" /> can be estimated using the expectation maximization (EM) algorithm:
 
<p align = "center">
 
<img src="https://tex.s2cms.ru/svg/%0A%20%5Ctext%7BE%20step%3A%7D%5Cquad%20A(%5Cvartheta%2C%5Cvartheta_s)%20%3D%20E_z%5B%5Clog%20p(x%2Cz%5Cmid%5Cvartheta)%5Cmid%20x%2C%20%5Cvartheta_s%5D%2C%5C%5C%0A%20%5Ctext%7BM%20step%3A%7D%20%5Cquad%20%5Cvartheta_%7Bs%2B1%7D%20%3D%20%5Carg%20%5Cunderset%7B%5Cvartheta%7D%7B%5Cmax%7D%20%5C%3A%20A(%5Cvartheta%2C%20%5Cvartheta_s)%2C%20%5C%5C%0A%20" alt="
 \text{E step:}\quad A(\vartheta,\vartheta_s) = E_z[\log p(x,z\mid\vartheta)\mid x, \vartheta_s],\\
 \text{M step:} \quad \vartheta_{s+1} = \arg \underset{\vartheta}{\max} \: A(\vartheta, \vartheta_s)," />
</p>

where 

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%09A(%5Cvartheta%2C%5Cvartheta_s)%20%3D%20%5Csum_%7Bk%3D1%7D%5E%7BK%7D%5Csum_%7Bt%3D1%7D%5E%7BN%7D%20E_z%5Bz_%7Bk%2Ct%7D%5Cmid%20x%2C%5Cvartheta_s%5D%5Cleft(%5Clog%5Cpi_%7Bk%7D%20%2B%20%5Clog%20p(x_t%20%5Cmid%20z_%7Bk%2Ct%7D%2C%20%5Cvartheta)%5Cright)%20%0A" alt="	A(\vartheta,\vartheta_s) = \sum_{k=1}^{K}\sum_{t=1}^{N} E_z[z_{k,t}\mid x,\vartheta_s]\left(\log\pi_{k} + \log p(x_t \mid z_{k,t}, \vartheta)\right) " />
</p>

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%09E_%7Bz%7D%20%5Bz_%7Bk%2Ct%7D%5Cmid%20x%2C%5Cvartheta_s%5D%26%3Dp(z_%7Bk%2Ct%7D%3D1%5Cmid%20x%2C%5Ctheta_s)%3D%5Cfrac%7B%20%5Cpi_k%20p(x_t%5Cmid%20z_%7Bk%2Ct%7D%3D1%2C%5Cvartheta_s)%20%7D%7B%20%5Csum_%7Bk%3D1%7D%5E%7BK%7D%20%5Cpi_k%20p(x_t%5Cmid%20z_%7Bk%2Ct%7D%3D1%2C%5Cvartheta_s)%7D%5Ctriangleq%20%5Ctau_%7Bk%2Ct%7D%0A" alt="	E_{z} [z_{k,t}\mid x,\vartheta_s]&amp;=p(z_{k,t}=1\mid x,\theta_s)=\frac{ \pi_k p(x_t\mid z_{k,t}=1,\vartheta_s) }{ \sum_{k=1}^{K} \pi_k p(x_t\mid z_{k,t}=1,\vartheta_s)}\triangleq \tau_{k,t}" />
</p>

Take the derivative of <img src="https://tex.s2cms.ru/svg/A" alt="A" /> with respect to <img src="https://tex.s2cms.ru/svg/%5Cmu_k" alt="\mu_k" />, we have 

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%09%5Cfrac%7B%5Cpartial%20A%7D%7B%20%5Cpartial%20%5Cmu_k%7D%20%26%3D%20%5Csum_%7Bt%3D1%7D%5E%7BN%7D%20%5Csum_%7Bk%3D1%7D%5E%7BK%7D%5Ctau_%7Bk%2Ct%7D%5Cfrac%7B%5Cpartial%20%5Clog%20p(x_t%5Cmid%20z_%7Bk%2Ct%7D%2C%5Cvartheta)%7D%7B%5Cpartial%20%5Cmu_%7Bk%7D%7D%3D0%20%5CRightarrow%0A%09%5Chat%7B%5Cmu%7D_k%3D%5Cfrac%7B%5Csum_%7Bt%3D1%7D%5E%7BN%7D%20%5Ctau_%7Bk%2Ct%7D%20x_t%7D%7B%5Csum_%7Bt%3D1%7D%5E%7BN%7D%5Ctau_%7Bk%2Ct%7D%7D%2C%5C%5C%0A%09%5Cfrac%7B%5Cpartial%20A%7D%7B%20%5Cpartial%20%5CSigma_k%7D%20%26%3D%20%5Csum_%7Bt%3D1%7D%5E%7BN%7D%5Ctau_%7Bk%2Ct%7D%5Cfrac%7B%5Cpartial%20%5Clog%20p(x_t%5Cmid%20z_%7Bk%2Ct%7D%2C%5Cvartheta)%7D%7B%5Cpartial%20%5CSigma_k%7D%3D0%20%5CRightarrow%20%5Chat%7B%5CSigma%7D_k%3D%5Cfrac%7B%5Csum_%7Bt%3D1%7D%5E%7BN%7D%20%5Ctau_%7Bk%2Ct%7D%20(x_t-%5Cmu_k)(x_t-%5Cmu_k)%5ET%7D%7B%5Csum_%7Bt%3D1%7D%5E%7BN%7D%5Ctau_%7Bk%2Ct%7D%7D%0A" alt="	\frac{\partial A}{ \partial \mu_k} &amp;= \sum_{t=1}^{N} \sum_{k=1}^{K}\tau_{k,t}\frac{\partial \log p(x_t\mid z_{k,t},\vartheta)}{\partial \mu_{k}}=0 \Rightarrow
	\hat{\mu}_k=\frac{\sum_{t=1}^{N} \tau_{k,t} x_t}{\sum_{t=1}^{N}\tau_{k,t}},\\
	\frac{\partial A}{ \partial \Sigma_k} &amp;= \sum_{t=1}^{N}\tau_{k,t}\frac{\partial \log p(x_t\mid z_{k,t},\vartheta)}{\partial \Sigma_k}=0 \Rightarrow \hat{\Sigma}_k=\frac{\sum_{t=1}^{N} \tau_{k,t} (x_t-\mu_k)(x_t-\mu_k)^T}{\sum_{t=1}^{N}\tau_{k,t}}," />
</p>

We should also consider the probability constraint, thus it is standard to use the Lagrangian multiplier as

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%09J(%5Cvartheta%2C%5Cvartheta_s)%3DA(%5Cvartheta%2C%5Cvartheta_s)%2B%5Clambda%5Cleft(1-%5Csum_%7Bk%3D1%7D%5E%7BK%7D%20%5Cpi_k%5Cright)%0A" alt="J(\vartheta,\vartheta_s)=A(\vartheta,\vartheta_s)+\lambda\left(1-\sum_{k=1}^{K} \pi_k\right)" />
</p>

Thus, to find the optimal <img src="https://tex.s2cms.ru/svg/%5Cpi_k" alt="\pi_k" />, we take the derivative to <img src="https://tex.s2cms.ru/svg/J" alt="J" /> with respect to <img src="https://tex.s2cms.ru/svg/%5Cpi_k" alt="\pi_k" />


<p align = "center">
<img src="https://tex.s2cms.ru/svg/%09%5Cfrac%7B%5Cpartial%20J%7D%7B%5Cpartial%20%5Cpi_k%7D%3D%5Csum_%7Bt%3D1%7D%5E%7BN%7D%20%5Ctau_%7Bk%2Ct%7D%5Cfrac%7B1%7D%7B%5Cpi_k%7D-%5Clambda%3D0%20%0A" alt="\frac{\partial J}{\partial \pi_k}=\sum_{t=1}^{N} \tau_{k,t}\frac{1}{\pi_k}-\lambda=0 " />
</p>

Combining the probability constraints and the above equation, we have 

<p align = "center">
<img src="https://tex.s2cms.ru/svg/%20%5Chat%7B%5Cpi%7D_k%3D%5Cfrac%7B1%7D%7BN%7D%7B%5Csum_%7Bt%3D1%7D%5E%7BN%7D%5Ctau_%7Bk%2Ct%7D%7D." alt=" \hat{\pi}_k=\frac{1}{N}{\sum_{t=1}^{N}\tau_{k,t}}." /> 
</p>

Then we iterate the EM until convergence, we can get the parameters. 


## Mean Shift

Mean shift is a procedure for locating the maxima—the modes—of a density function given discrete data sampled from that function. This is an iterative method, and we start with an initial estimate <img src="https://tex.s2cms.ru/svg/%20%5Cmu" alt=" \mu" />. Let a kernel function <img src="https://tex.s2cms.ru/svg/%20K(x_%7Bi%7D-%5Cmu)" alt=" K(x_{i}-\mu)" />  be given. This function determines the weight of nearby points for re-estimation of the mean. Typically a Gaussian kernel on the distance to the current estimate is used, <img src="https://tex.s2cms.ru/svg/%20K(x_%7Bi%7D-x)%3De%5E%7B-c%7C%7Cx_%7Bi%7D-%5Cmu%7C%7C%5E%7B2%7D%7D" alt=" K(x_{i}-x)=e^{-c||x_{i}-\mu||^{2}}" />. The weighted mean of the density in the window determined by <img src="https://tex.s2cms.ru/svg/K" alt="K" /> is

<p align = "center">

<img src="https://tex.s2cms.ru/svg/m(x)%20%3D%20%5Cfrac%7B%20%5Csum_%7Bx_i%20%5Cin%20N(x)%7D%20K(x_i%20-%20%5Cmu)%20x_i%20%7D%20%7B%5Csum_%7Bx_i%20%5Cin%20N(x)%7D%20K(x_i%20-%20%5Cmu)%7D%20" alt="m(x) = \frac{ \sum_{x_i \in N(x)} K(x_i - \mu) x_i } {\sum_{x_i \in N(x)} K(x_i - \mu)} " />
</p>

where <img src="https://tex.s2cms.ru/svg/%5Cinline%20N(x)" alt="\inline N(x)" />  is the neighborhood of <img src="https://tex.s2cms.ru/svg/%20x" alt=" x" />, a set of points for which <img src="https://tex.s2cms.ru/svg/%20K(x_%7Bi%7D)%5Cneq%200" alt=" K(x_{i})\neq 0" />.

The difference <img src="https://tex.s2cms.ru/svg/%20m(x)-%5Cmu" alt=" m(x)-\mu" /> is called mean shift in [Fukunaga and Hostetler](https://ieeexplore.ieee.org/document/1055330). The mean-shift algorithm now sets <img src="https://tex.s2cms.ru/svg/%20%5Cmu%20%5Cleftarrow%20m(x)%20" alt=" \mu \leftarrow m(x) " />, and repeats the estimation until <img src="https://tex.s2cms.ru/svg/%20m(x)" alt=" m(x)" />  converges.



## Modification
The feature vector <img src="https://tex.s2cms.ru/svg/%5Cinline%20x" alt="\inline x" /> contains two types of features that are on different scales; the chrominance values <img src="https://tex.s2cms.ru/svg/%5Cinline%20(u%3B%20v)" alt="\inline (u; v)" /> range from <img src="https://tex.s2cms.ru/svg/%5Cinline%200-255" alt="\inline 0-255" />, while the pixel locations <img src="https://tex.s2cms.ru/svg/%20%5Cinline%20(c_%7Bx%7D%3B%20c_%7By%7D)" alt=" \inline (c_{x}; c_{y})" /> range from <img src="https://tex.s2cms.ru/svg/%5Cinline%200-512" alt="\inline 0-512" />. The EM-GMM clustering algorithm can adapt to the different scales of each feature by changing the covariance matrix. On the other hand, K-means and mean-shift assume a fixed isotropic covariance matrix.
We can modify these two algorithms to allow different scaling of the features:
- For K-means, the distance to a cluster center <img src="https://tex.s2cms.ru/svg/%5Cinline%20x'" alt="\inline x'" /> is modified to apply a weighting between the features types,

<p align = "center">

<img src="https://tex.s2cms.ru/svg/%20d(x%2Cx')%20%3D%20%5Cleft%20%5CVert%20%5Cbegin%7Bbmatrix%7D%20u%5C%5C%20v%5Cend%7Bbmatrix%7D%20-%5Cbegin%7Bbmatrix%7D%20u'%5C%5C%20v'%5Cend%7Bbmatrix%7D%5Cright%5CVert%5E%7B2%7D%20%20%2B%20%5Clambda%20%5Cleft%5CVert%20%5Cbegin%7Bbmatrix%7D%20c_%7Bx%7D%5C%5C%20c_%7By%7D%5Cend%7Bbmatrix%7D%20-%5Cbegin%7Bbmatrix%7D%20c_%7Bx%7D'%5C%5C%20c_%7By%7D'%5Cend%7Bbmatrix%7D%5Cright%20%5CVert%5E2" alt=" d(x,x') = \left \Vert \begin{bmatrix} u\\ v\end{bmatrix} -\begin{bmatrix} u'\\ v'\end{bmatrix}\right\Vert^{2}  + \lambda \left\Vert \begin{bmatrix} c_{x}\\ c_{y}\end{bmatrix} -\begin{bmatrix} c_{x}'\\ c_{y}'\end{bmatrix}\right \Vert^2" />
</p>

where <img src="https://tex.s2cms.ru/svg/%5Cinline%20%5Clambda" alt="\inline \lambda" /> controls the weighting

- For mean-shift, the kernel is modified to use separate bandwidths on each feature type,

<p align = "center">

<img src="https://tex.s2cms.ru/svg/k(x%2Cx')%20%3D%5Cfrac%7B1%7D%7B2%5Cpi%20h_%7Bp%7D%5E2%20h_%7Bc%7D%5E2%7D%20exp%5Cleft%5C%7B%5Cfrac%7B1%7D%7B2%20h_%7Bc%7D%5E2%7D%5Cleft%20%5CVert%20%5Cbegin%7Bbmatrix%7D%20u%5C%5C%20v%5Cend%7Bbmatrix%7D%20-%5Cbegin%7Bbmatrix%7D%20u'%5C%5C%20v'%5Cend%7Bbmatrix%7D%5Cright%5CVert%5E%7B2%7D%20%20-%20%5Cfrac%7B1%7D%7B2%20h_%7Bp%7D%5E2%7D%20%5Cleft%5CVert%20%5Cbegin%7Bbmatrix%7D%20c_%7Bx%7D%5C%5C%20c_%7By%7D%5Cend%7Bbmatrix%7D%20-%5Cbegin%7Bbmatrix%7D%20c_%7Bx%7D'%5C%5C%20c_%7By%7D'%5Cend%7Bbmatrix%7D%5Cright%20%5CVert%5Cright%5C%7D" alt="k(x,x') =\frac{1}{2\pi h_{p}^2 h_{c}^2} exp\left\{\frac{1}{2 h_{c}^2}\left \Vert \begin{bmatrix} u\\ v\end{bmatrix} -\begin{bmatrix} u'\\ v'\end{bmatrix}\right\Vert^{2}  - \frac{1}{2 h_{p}^2} \left\Vert \begin{bmatrix} c_{x}\\ c_{y}\end{bmatrix} -\begin{bmatrix} c_{x}'\\ c_{y}'\end{bmatrix}\right \Vert\right\}" />
</p>

where <img src="https://tex.s2cms.ru/svg/%5Cinline%20h_%7Bc%7D" alt="\inline h_{c}" /> is the bandwidth for the color features, and <img src="https://tex.s2cms.ru/svg/%5Cinline%20h_%7Bp%7D" alt="\inline h_{p}" /> is the bandwidth for the locations.

# Results
The results based on raw data are not presented as they give poor segmentation. The feature location <img src="https://tex.s2cms.ru/svg/%20%5Cinline%20(c_%7Bx%7D%3B%20c_%7By%7D)" alt=" \inline (c_{x}; c_{y})" /> of the image was scaled by <img src="https://tex.s2cms.ru/svg/%20%5Cinline%20%5Cfrac%7B(c_%7Bx%7D%3B%20c_%7By%7D)%7D%7B10%7D" alt=" \inline \frac{(c_{x}; c_{y})}{10}" />. The segmentation by the clustering algorithms with different parameters is shown below.

<p align = "center">
<img src = https://user-images.githubusercontent.com/45757826/57580220-40fa0b80-74a7-11e9-9a45-55c3b6d16942.png>
</p>
     

Fig.1 K-means clustering results with <img src="https://tex.s2cms.ru/svg/%5Cinline%20K%20%3D%204" alt="\inline K = 4" />

<p align = "center">
<img src =https://user-images.githubusercontent.com/45757826/57580219-40fa0b80-74a7-11e9-923f-089491fb1c25.png>
</p>

Fig.2 GMM clustering results with <img src="https://tex.s2cms.ru/svg/%5Cinline%20K%20%3D%204" alt="\inline K = 4" />

<p align = "center">
<img width = "650" height = "150" src =https://user-images.githubusercontent.com/45757826/57580221-4192a200-74a7-11e9-84df-551224718bb6.png>
</p>

Fig.3 mean-shift clustering with <img src="https://tex.s2cms.ru/svg/%5Cinline%20h%20%3D%2010" alt="\inline h = 10" />
 
As shown in the results, K Means performs best in segmentation. 

We also studied different <img src="https://tex.s2cms.ru/svg/K" alt="K" /> for GMM and K means, and <img src="https://tex.s2cms.ru/svg/h" alt="h" /> for mean-shift (MS) clustering. GMM is less sensitive to the changes in parameters, while K means and MS is quite sensitive.


When the number of clusters is unknown, GMM and MS are better choices, since MS can adaptively converge to the cluster centers and GMM is robust to K values. 
K means is sensitive to the <img src="https://tex.s2cms.ru/svg/K" alt="K" /> value. As <img src="https://tex.s2cms.ru/svg/K" alt="K" /> increases, the segmentation becomes poor.

Means shift is sensitive to the parameter <img src="https://tex.s2cms.ru/svg/h" alt="h" />. When the <img src="https://tex.s2cms.ru/svg/h" alt="h" /> is out of a certain range, segmentation is poor (h=10, h=13, h=15).
For consideration of computational time, MS is the most computational costive, since it requires each data point converge to a cluster.  GMM has a modest computation cost and K Means is the most computationally efficient.


# How to Use
The raw images are contained in [images](#).

The repository contains MATLAB code for extracting the features and creating the segmentation image from the cluster labels:
 - [getfeatures.m](#) --MATLAB function for extracting features from an image
 - [labels2segm.m](#) --MATLAB function for generating a segmentation image from the cluster labels
 - [colorssegm.m](#) -- MATLAB function to create a nicely colored segmentation image

As an example, the following code was used to generate the above segmentation images:
```Matlab
% read the image and view it
img = imread('image/12003.jpg')
subplot(1,3,1); imagesc(img); axis imag;

% extract features (stepsize =7)
[X, L] = getfeatures(img, 7);
XX = [X(1:2,:); X(3:4,:)/10]; % downscale the coordinate features

% run, e,g. kmeans
[Y, C] = kmeans(XX',2);

% make a segmentation image from the labels

segm =labels2segm(Y, L);
subplot(1,3,2), imagesc(segm); axis image;

% color the segmentation image
csegm = colorsegm(segm, img)
subplot(1,3,3); imagesc(csegm); axis image
```







