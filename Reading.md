# Paper and Codes Reading #
This Repositry is a clone of the official implementations of the paper 'Deep Animation Video Interpolation in the Wild'(CVPR21).

I added some code annotations in Chinese and there were something inaccurate in codes which will be illustrated later.

## About Paper ##

### Introduction ###


### Contributions ###


### Limits ###
- Not end-to-end 


### Whole Framework with Correspondent Codes ###
#### Dataset ####
[ATD-12K](https://drive.google.com/file/d/1XBDuiEgdd6c0S4OXLF4QvgSn_XNPwc-g/view) Dataset with triplets of animation frames from videos in the wild. It has been splited into 10k training samples and 2k test samples. 

Specific annotations are in a .json file, include:

- difficulty levels: 0 : “Easy”, 1 : “Medium”, 2 :  “Hard”.
- motion RoI(Region of Interest): x, y, width, height.
- general\_motion\_type: "translation", "rotation", "scaling", "deformation".
- behavior: "speaking", "walking", "eating", "sporting",
"fetching", "others".

#### Segment-Guided Matching ####
**1. Color Piece Segmentation**

Laplacian filter to extract contours of animation frames[1].
[**./gen\_labelmap.py/dline\_of**].

 “Trapped-
ball” algorithm to fill the contours then generate color pieces[1]. [**./linefiller & gen\_labelmap.py/trapped\_ball\_processed**]

A segmentation map where pixels
of each color piece is labeled by an identity number. [**./linefiller/trappedball\_fill.py/build\_fill\_map**]

**2. Feature Collection**

Extract features of relu1\_2, relu2\_2, relu3\_4 and relu4\_4 layers from pretrained VGG-19 model[2]. [**./my\_models.py/create\_VGGFeatNet**]

Assemble the features belonging to one segment by
the super-pixel pooling[3]. [**gen\_sgm.py/superpixel\_pooling**]

**3. Color Piece Matching**

Compute an affinity metric $\mathcal{A}$ [**./gen\_sgm.py** line 553], the distance penalty $\mathcal{L}\_{dist}$ [**./gen\_sgm.py** line 559], the size penalty $\mathcal{L}\_{size}$ [**./gen\_sgm.py** line 564], the matching map $\mathcal{M}$ [**./gen\_sgm.py/mutual\_matching**].


**4. Flow Generation**

Compute flow f [**./gen\_sgm.py/get\_guidance\_flow**]


#### Recurrent Flow Refinement Network ####

Inspired by [4], design a transformer-like architecture to recurrently refine the piece-wise flow.

- 3-layer Conv [**./rfr\_new.py/ErrorAttention**]
- Feature Net [**./extractor.py/BasicEncoder**]
- ConvGRU[5] [**./update.py/SepConvGRU**]
- Correlation [**./corr.py/CorrBlock**]


#### Frame Warping and Synthesis ####

Generate the intermediate frame by using the splatting and synthesis strategy of Soft-Splat[6].

All features and input frames are softmax splatted via forward warping. [**./softsplat.py/ModuleSoftsplat**]

All warped frames and features are fed into a GridNet[7] to synthesize the target frame. [**./GridNet.py/GridNet**]

## Inaccurate Parts ##

#### test\_anime\_sequence\_one\_by\_one.py ####
about line 38

    # source
	revmean = [-x for x in config.mean]

	# suggest to change as
	revmean = [-x for x in config.mean]

Normalize\_1: (X - Mean) / Std = Y. To reverse this normalize\_1 by another normalize\_2, it should be: (Y - (-Mean/Std)) / (1/Std) = X. So the 'revmean' above should be changed to 

### References ###

[1] Zhang, Song-Hai, et al. "Vectorizing cartoon animations." IEEE Transactions on Visualization and Computer Graphics 15.4 (2009): 618-629.

[2] Simonyan, Karen, and Andrew Zisserman. ["Very deep convolutional networks for large-scale image recognition."](https://arxiv.org/pdf/1409.1556.pdf%E3%80%82) arXiv preprint arXiv:1409.1556 (2014).

[3] Liu, Fayao, et al. ["Learning depth from single monocular images using deep convolutional neural fields."](https://arxiv.org/pdf/1502.07411) IEEE transactions on pattern analysis and machine intelligence 38.10 (2015): 2024-2039.

[4] Teed, Zachary, and Jia Deng. ["Raft: Recurrent all-pairs field transforms for optical flow."](https://arxiv.org/pdf/2003.12039) European conference on computer vision. Springer, Cham, 2020.

[5] Cho, Kyunghyun, et al. ["On the properties of neural machine translation: Encoder-decoder approaches."](https://arxiv.org/pdf/1409.1259.pdf?ref=https://githubhelp.com) arXiv preprint arXiv:1409.1259 (2014).

[6] Niklaus, Simon, and Feng Liu. ["Softmax splatting for video frame interpolation."](http://openaccess.thecvf.com/content_CVPR_2020/papers/Niklaus_Softmax_Splatting_for_Video_Frame_Interpolation_CVPR_2020_paper.pdf) Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2020.

[7] Fourure, Damien, et al. ["Residual conv-deconv grid network for semantic segmentation."](https://arxiv.org/pdf/1707.07958.pdf?ref=https://githubhelp.com) arXiv preprint arXiv:1707.07958 (2017).

