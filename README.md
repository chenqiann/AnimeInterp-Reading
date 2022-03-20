# Paper and Codes Reading #
This repositry is a clone of the official implementations of the paper 'Deep Animation Video Interpolation in the Wild'(CVPR21). The sourse [README.md](https://github.com/lisiyao21/AnimeInterp#readme) file please see [AnimeInterp](https://github.com/lisiyao21/AnimeInterp) repositry.

I added some code annotations in Chinese and there were something inaccurate in codes which will be illustrated later.

## About Paper ##

### Introduction ###

Different from natural video interpolation, animation video has unique characteristics:

- Texture Insufficiency: Cartoons comprise lines and smooth color pieces. The smooth areas lack textures and make it difficult to estimate accurate motions on animation videos.
- Large Motions: Cartoons express stories via exaggeration. Some of the motions are non-linear and extremely large.

Along with original challenges of natural video interpolation, like occlussion handling, video interpolation in animations remains a challenging task.

This paper propsed an effective framework, AnimeInterp[[8]](#references), with two dedicated modules, [SGM](#segment-guided-matching) and [RFR](#recurrent-flow-refinement-network), in a coarse-to-fine manner.

---------------
### Contributions ###

- Formally define and study the animation video interpolation problem for the first time.
- Propose an effective animation interpolation framework named AnimeInterp with two dedicated modules to resolve the “lack of textures” and “non-linear and extremely large motion” challenges, which outperforms existing state-of-the-art methods both quantitatively and qualitatively.
- Build a large-scale cartoon triplet dataset called ATD-12K with large content diversity representing many types of animations to test animation video interpolation methods.

---------------
### Limits ###
- Not mentioned 

---------------
### Framework with Dataset and Correspondent Codes ###
###### Framework ######

![framework](https://github.com/chenqiann/AnimeInterp-Reading/blob/main/figs/framework.png)

#### -Dataset ####
[ATD-12K](https://drive.google.com/file/d/1XBDuiEgdd6c0S4OXLF4QvgSn_XNPwc-g/view) Dataset[[8]](#references) with triplets of animation frames from videos in the wild. It has been splited into 10k training samples and 2k test samples. 

Specific annotations are in a .json file, include:

- difficulty levels: 0 : “Easy”, 1 : “Medium”, 2 :  “Hard”.
- motion RoI(Region of Interest): x, y, width, height.
- general\_motion\_type: "translation", "rotation", "scaling", "deformation".
- behavior: "speaking", "walking", "eating", "sporting",
"fetching", "others".

#### -Segment-Guided Matching ####

![sgm](https://github.com/chenqiann/AnimeInterp-Reading/blob/main/figs/sgm_module.png)

input: $I_{0}$, $I_{1}$ - input images

output: $f_{0\rightarrow1}$, $f_{1\rightarrow0}$ - coarse optical flow


**1. Color Piece Segmentation**

Laplacian filter to extract contours of animation frames[[1]](#references).
[**./gen\_labelmap.py/dline\_of**].

 “Trapped-
ball” algorithm to fill the contours then generate color pieces[[1]](#references). [**./linefiller & gen\_labelmap.py/trapped\_ball\_processed**]

A segmentation map where pixels
of each color piece is labeled by an identity number. [**./linefiller/trappedball\_fill.py/build\_fill\_map**]

**2. Feature Collection**

Extract features of relu1\_2, relu2\_2, relu3\_4 and relu4\_4 layers from pretrained VGG-19 model[[2]](#references). [**./my\_models.py/create\_VGGFeatNet**]

Assemble the features belonging to one segment by
the super-pixel pooling[[3]](#references). [**gen\_sgm.py/superpixel\_pooling**]

**3. Color Piece Matching**

Compute an affinity metric $\mathcal{A}$ [**./gen\_sgm.py** line 553], the distance penalty $\mathcal{L}\_{dist}$ [**./gen\_sgm.py** line 559], the size penalty $\mathcal{L}\_{size}$ [**./gen\_sgm.py** line 564], the matching map $\mathcal{M}$ [**./gen\_sgm.py/mutual\_matching**].


**4. Flow Generation**

Compute flow f [**./gen\_sgm.py/get\_guidance\_flow**]


#### -Recurrent Flow Refinement Network ####

<img src="https://github.com/chenqiann/AnimeInterp-Reading/blob/main/figs/rfr_module.png" alt="rfr" width="363" height="456" align="bottom" />

input: $I_{0}$, $I_{1}$, $f_{0\rightarrow1}$, $f_{1\rightarrow0}$ - input images and coarse optical flow computed by SGM module

output: $f^{'}\_{0\rightarrow1}$, $f^{’}\_{1\rightarrow0}$ - fine flow

Inspired by [[4]](#references), design a transformer-like architecture to recurrently refine the piece-wise flow.

- 3-layer Conv [**./rfr\_new.py/ErrorAttention**]
- Feature Net [**./extractor.py/BasicEncoder**]
- ConvGRU[[5]](#references) [**./update.py/SepConvGRU**]
- Correlation [**./corr.py/CorrBlock**]


#### -Frame Warping and Synthesis ####

input: $I_{0}$, $I_{1}$, $f^{'}\_{0\rightarrow1}$, $f^{'}\_{1\rightarrow0}$ - input images and fine flow computed by RFR module

output: $\hat{I}_{1/2}$ - interpolated image

Generate the intermediate frame by using the splatting and synthesis strategy of Soft-Splat[[6]](#references).

All features and input frames are softmax splatted via forward warping. [**./softsplat.py/ModuleSoftsplat**]

All warped frames and features are fed into a GridNet[[7]](#references) to synthesize the target frame. [**./GridNet.py/GridNet**]


## Inaccurate Parts ##

#### test\_anime\_sequence\_one\_by\_one.py ####
about line 38

    # source
	revmean = [-x for x in config.mean]

	# suggest to change as
	revmean = [-x for x in config.mean]
	revmean = [revmean/std for std in (fill variable of Std here)]

Set normalize\_1: (X - Mean) / Std = Y. To reverse this normalization by another normalize\_2, it should be: (Y - (-Mean/Std)) / (1/Std) = X. So the 'revmean' above should be adjusted if Std not equals 1.

### References ###

[1] Zhang, Song-Hai, et al. "Vectorizing cartoon animations." IEEE Transactions on Visualization and Computer Graphics 15.4 (2009): 618-629.

[2] Simonyan, Karen, and Andrew Zisserman. ["Very deep convolutional networks for large-scale image recognition."](https://arxiv.org/pdf/1409.1556.pdf%E3%80%82) arXiv preprint arXiv:1409.1556 (2014).

[3] Liu, Fayao, et al. ["Learning depth from single monocular images using deep convolutional neural fields."](https://arxiv.org/pdf/1502.07411) IEEE transactions on pattern analysis and machine intelligence 38.10 (2015): 2024-2039.

[4] Teed, Zachary, and Jia Deng. ["Raft: Recurrent all-pairs field transforms for optical flow."](https://arxiv.org/pdf/2003.12039) European conference on computer vision. Springer, Cham, 2020.

[5] Cho, Kyunghyun, et al. ["On the properties of neural machine translation: Encoder-decoder approaches."](https://arxiv.org/pdf/1409.1259.pdf?ref=https://githubhelp.com) arXiv preprint arXiv:1409.1259 (2014).

[6] Niklaus, Simon, and Feng Liu. ["Softmax splatting for video frame interpolation."](http://openaccess.thecvf.com/content_CVPR_2020/papers/Niklaus_Softmax_Splatting_for_Video_Frame_Interpolation_CVPR_2020_paper.pdf) Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2020.

[7] Fourure, Damien, et al. ["Residual conv-deconv grid network for semantic segmentation."](https://arxiv.org/pdf/1707.07958.pdf?ref=https://githubhelp.com) arXiv preprint arXiv:1707.07958 (2017).

[8] Siyao, Li, et al. ["Deep animation video interpolation in the wild."](http://openaccess.thecvf.com/content/CVPR2021/papers/Siyao_Deep_Animation_Video_Interpolation_in_the_Wild_CVPR_2021_paper.pdf) Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2021.
