# Practical Denoising
Reference Implementation of Practical Denoising for VFX Production Using Temporal Blur

![Header Image](denoiserHeaderImage.png)

## Introduction

This is a reference implementation of the method presented in the talk Practical Denoising for VFX Production Using Temporal Blur, published in [Proceedings of SIGGRAPH ’18 Talks](https://doi.org/10.1145/3214745.3214762).

It consists of a node network for the [Gaffer]( http://www.gafferhq.org/ ) application.  This network contains a section which sets up special geometry variables, and uses them to do renders of a test scene with the offset AOVs required by our method.  It then contains another sequence of nodes which perform the image space operations that do the actual denoising.

## Running

Download Gaffer for your platform: http://www.gafferhq.org/download/ ( Linux and OS X supported ).

Download the Arnold renderer for your platform: https://www.solidangle.com/arnold/try/

( It should be a fairly small amount of work to support other renderers, in the future we may add support for the free Appleseed renderer by default ).

Set the ARNOLD_ROOT environment variable to point to the directory where Arnold is installed.

Launch Gaffer, and open the file denoiserReferenceImplementation.gfr stored in this repo.

Executing the node "ArnoldRender" will render a frame sequence suitable for denoising.  Once you've rendered a range of frames to disk, you can then select nodes in the Denoiser box to step through the steps of the denoising process, or execute the node "BeautyWriter" to write the sequence of denoised images to disk.

## Video Walkthrough

I'll be briefly presenting a video walking through this network at the SIGGRAPH Gaffer BOF.  A recording of this session will be linked at some point ...

## Caveats

In order to make an implementation usable directly in standard Gaffer without custom C++ code, a few corners have been cut.

### Renderer Integration
 - Sample Clamp - Implementing our soft sample clamping ( Equation 3 in Supplemental Material ) requires direct knowledge of the renderer's filter kernel.  To avoid this, we use standard sample claming here ( Equation 2 in Supplemental Material ).  This produces less pleasing highlights, and does not preserve additivity among AOVs when denoising multiple AOVs, but is usable.
 - Variance Computation - Implementing our more accurate pixel variance calculation is also a bit intrusive, so instead we approximate this by dividing Arnold's sample variance by the number of samples ( the square of Arnold AA_samples setting ).  This yields very slightly less accurate edges, but does not significantly affect overall quality.
 
 ### Scene Support
 The network that sets up the data for the offset AOVs should handle most scenes.
 
 This network does not currently support cameras with an animated field-of-view - in the future, we will add a method to Gaffer for querying this, and the expressions can be updated to take this into account.
 
 Because Arnold does not support Vertex interpolation for curve primvars, you would need to use a Resample node to resample the POffsetPrev and POffsetNext primvars to Varying.  In the future this will be done automatically by the Arnold renderer backend.

## Questions?

The talk's [abstract and supplemental material](denoiser2018AbstractAndSupplemental.pdf) describes this approach in a fair bit of detail.  If you have questions about the details, or are using Gaffer in production, and are have questions about using this sort of approach in production, my email address is at the top of the abstract.
