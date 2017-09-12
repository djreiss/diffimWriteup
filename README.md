# Current state of image subtraction in the LSST stack

# 1. Alard and Lupton (A&L) PSF matching and image subtraction

The A&L algorithm is used by default in `ip_diffim` to perform PSF matching and image subtraction. It performs quickly and well because it uses small regions surrounding bright, isolated stars around to compute the PSF-matching kernel, $k_i$, at various locations $i$ across the image. It uses various heuristics to pre-filter those bright stars prior to computation of the $k_i$, and once they are computed, uses PCA to estimate a smoothly spatially-varying $k$ from them.

There are a very large number of configuration parameters which affect the quality of the subtraction. In general, the defaults work well, although for images with different pixel-scales and/or PSF sizes, these parameters may need to be tuned. Many of these important parameters are buried deep in the `kernel` config parameter of the `subtract` algorithm (a reference to the `lsst.ip.diffim.ImagePsfMatchTask` task).

The image subtraction script is in `lsst.pipe.tasks.imageDifference`, which performs image subtraction and then detects and measures sources (`diaSources`) in the subtractions. This task itself very recently was refactored (DM-3704) and split into two separate tasks which now reside in `ip_diffim`: `MakeDiffimTask` and `ProcessDiffimTask`, which are called sequentially from the command-line `imageDifference.py`.

## 1.1. Pre-convolution

An un-published modification to the A&L algorithm was implemented, which accounts for the occasions when the width of the PSF of the science image is $\leq$ the width of the PSF of the template image. In this case, A&L cannot convolve the template to match that of the science image; it instead would need to deconvolve the template, which would result in ringing artifacts. Instead, the science image is "pre-convolved," or "pre-filtered" with its own PSF. If the template PSF is narrower than $\sqrt{2}\times$ that of the science image, then A&L will now work, but the resulting image subtraction will have been pre-filtered by the science image's PSF. This image then corresponds to the pre-filtered "likelihood" image, which, for detection, just needs to be thresholded. A special case is then needed to do any kind of measurement on detected sources in this image. I am not entirely sure how that works.

## 1.2. A&L Decorrelation

When the template exposure has significant noise (i.e., is not constructed from a number of coadds), then A&L will correlate the noise among neighboring pixels when it convolves the template with the PSF-matching kernel, $k$. As a result, the noise will be correlated in the image subtraction, leading to inaccurate detection and measurement (see [DMTN-006](https://dmtn-021.lsst.io/) for details). In [DMTN-021](https://dmtn-021.lsst.io/), we describe a method for "decorrelating" the A&L image subtraction, and this has been implemented in the LSST image subtraction code. See section (3) below for details about how the decorrelation is performed, accounting for spatially-varying PSFs and noise.

### 1.2.1. Decorrelation + pre-convolution = trouble

## 

# 2. Zackay, et al. (2016) (ZOGY) image subtraction

## 2.1. Variants (image-space convolutions)

## 2.3. The ZOGY $S_{corr}$ image

## 2.3. Issues, unimplemented aspects, artifacts

# 3. Spatial variations via `ImageMapReduce`

## 3.1. Implementation details

## 3.2. Known issues

# 4. Conclusions and recommendations for future work


------
1. decorrelation + preconvolution not working
2. zogy artifacts
3. imageMapReduce gridding could be optimized, right now makes the map-reduce part slow.
4. use of coaddPsf not ideal -- detection is fast but measurement is SLOW
5. issue with zogy when psfs have different dimensions (offset due to psf padding)
--- this is now fixed but points out issues when images are not properly flux-calibrated
6. additional artifacts when zogy run with image-space convolutions
7. differences between what is produced by A&L vs. ZOGY (e.g. matched template, etc.) and how to handle that with DipoleFitting. Zogy in spatially varying mode does not return the matchedTemplate, and thus it is not used for dipole fititng.
- the cross-convolved images in Zogy are not useful for dipole fitting.
8. spatially-varying decorrelation is done by computing the kernel on chunks, and then convolving it on those chunks. should consider computing on chunks, then creating smoothly spatially-varying kernel, then convolving the image w/ the spatially varying kernel
