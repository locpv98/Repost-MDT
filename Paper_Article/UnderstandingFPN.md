# Understanding Feature Pyramid Networks for object detection (FPN)
Detecting objects in different scales (quy mô) is challenging in particular (cụ thể) for small objects. We can use a pyramid of the same image at different scale to detect objects (the left diagram below). However, processing multiple scale (tỉ lệ) images is time consuming (tiêu thụ/mất) and the memory demand (nhu cầu) is too high to be trained end-to-end (đầu đến cuối) simultaneously (đồng thời). Hence (vì thế), we may only use it in inference (suy luận) to push accuracy as high as possible, in particular for competitions (cuộc thi), when speed is not a concern (mối quan tâm). Alternatively (ngoài ra), we create a pyramid of feature and use them for object detection (the right diagram). However, feature maps closer to the image layer composed of low-level structures that are not effective (có hiệu lực) for accurate object detection.

![Figure 1.](https://miro.medium.com/max/700/1*UtfPTLB53cR8EathGBOT2Q.jpeg)  

**Feature Pyramid Network (FPN)** is a feature extractor designed for such pyramid concept with accuracy and speed in mind. It replaces the feature extractor of detectors like Faster R-CNN and generates multiple feature map layers (**multi-scale feature maps**) with better quality information than the regular (thông thường) feature pyramid for object detection.  

## Data Flow

![Figure 2.](https://miro.medium.com/max/500/1*aMRoAN7CtD1gdzTaZIT5gA.png)  

FPN composes (sáng tác) of **a bottom-up** and **a top-down** pathway. The bottom-up pathway is the usual (bthg) convolutional network for feature extraction. As we go up, the spatial (không gian) resolution (sự phân giải) decreases. With more high-level structures detected, the **semantic value** (giá trị ngữ nghĩa) for each layer increases.  

![Figure 3.](https://miro.medium.com/max/470/1*_kxgFskpRJ6bsxEjh9CH6g.jpeg)  

SSD makes detection from multiple feature maps. However, the bottom layers are not selected for object detection. They are in high resolution but the semantic value is not high enough (đủ) to (để) justify (biện minh) its use as the speed slow-down (chậm lại) is significant (có ý nghĩa). So SSD only uses upper (trên) layers for detection and therefore performs much (nhiều) worse (kém/tệ hơn) for small objects. 

![image](https://user-images.githubusercontent.com/80739312/112411233-28b7d080-8d4f-11eb-848d-20c06dd746cb.png)  

FPN provides a top-down pathway to construct higher resolution layers from a semantic (ngữ nghĩa0 rich (phong phú) layer.

![Reconstruct spatial resolution in the top-down pathway](https://user-images.githubusercontent.com/80739312/112411416-76ccd400-8d4f-11eb-97a0-44ff8e7740ca.png) 

While the reconstructed (tái tạo) layers are semantic strong but the locations of objects are not precise (tóm lược) after all the downsampling (lấy mẫu xuống) and upsampling. We add lateral connections between reconstructed layers and the corresponding (tương ứng) feature maps to help the detector to predict the location betters. It also acts as skip connections to make training easier (similar to what ResNet does).  

![image](https://user-images.githubusercontent.com/80739312/112411766-07a3af80-8d50-11eb-8cd3-291eab703cf9.png)  

## Bottom-up pathway
The bottom-up pathway uses ResNet to construct the bottom-up pathway. It composes of many convolution modules (convi for i equals (=) 1 to 5) each has many convolution layers. As we move up, the spatial (không gian) dimension (kích thước) is reduced (giảm) by 1/2 (i.e. double the stride). The output of each convolution module is labeled as Ci and later used in the top-down pathway.

![image](https://user-images.githubusercontent.com/80739312/112412247-cbbd1a00-8d50-11eb-855c-ef8687410d0f.png)  

## Top-down pathway
We apply a 1 × 1 convolution filter to reduce C5 channel depth to 256-d to create M5. This becomes the first feature map layer used for object prediction.

As we go down the top-down path, we upsample (lấy mẫu) the previous (trước) layer by 2 using nearest neighbors upsampling. We again apply a 1 × 1 convolution to the corresponding feature maps in the bottom-up pathway. Then we add them element-wise. We apply a 3 × 3 convolution to all merged layers. This filter reduces the aliasing (răng cưa) effect when merged (hợp nhất) with the upsampled layer.

![image](https://user-images.githubusercontent.com/80739312/112412306-e4c5cb00-8d50-11eb-81ad-4c44648f7912.png)  

We repeat the same process for P3 and P2. However, we stop at P2 because the spatial dimension of C1 is too large. Otherwise (nếu không thì), it will slow down the process too much. Because we share the same classifier and box regressor (hồi quy) of every output feature maps, all pyramid feature maps (P5, P4, P3 and P2) have 256-d output channels.

## PN with RPN (Region Proposal Network)
> _FPN is not an object detector by itself. It is a feature extractor that works with object detectors._

FPN extracts feature maps and later feeds into a detector, says RPN, for object detection. RPN applies a sliding window over the feature maps to make predictions on the objectness (has an object r not) and the object boundary box at each location.

![image](https://user-images.githubusercontent.com/80739312/112412564-4e45d980-8d51-11eb-81d2-bed59ec922c5.png)  

In the FPN framework, for each scale level (say P4), a 3 × 3 convolution filter is applied over the feature maps followed by separate 1 × 1 convolution for objectness predictions and boundary box regression. These 3 × 3 and 1 × 1 convolutional layers are called the RPN **head**. The same head is applied to all different scale levels of feature maps.  

![image](https://user-images.githubusercontent.com/80739312/112412686-851bef80-8d51-11eb-9249-e41c47f366ae.png)  

## FPN with Fast R-CNN or Faster R-CNN
Let’s take a quick look at the Fast R-CNN and Faster R-CNN data flow below. It works with one feature map layer to create ROIs. We use the ROIs and the feature map layer to create feature patches (vá) to be _fed into_ (đưa vào) the ROI pooling.

![image](https://user-images.githubusercontent.com/80739312/112412747-9bc24680-8d51-11eb-92ae-80de398ae90e.png)  

In FPN, we generate a pyramid of feature maps. We apply the RPN (described in the previous section) to generate ROIs. Based on the size of the ROI, we select the feature map layer in the most proper (thích hợp) scale to extract the feature patches.  

![image](https://user-images.githubusercontent.com/80739312/112412812-b5638e00-8d51-11eb-95b0-97f4931255c6.png)  

The formula (công thức) to pick the feature maps is based on the width w and height h of the ROI.

![image](https://user-images.githubusercontent.com/80739312/112412891-d3c98980-8d51-11eb-9243-8e2e39fd4811.png)  

So if k = 3, we select P3 as our feature maps. We apply the ROI pooling and feed the result to the Fast R-CNN head (Fast R-CNN and Faster R-CNN have the same head) to finish the prediction.

## Segmentation (Phân đoạn)
Just like Mask R-CNN, FPN is also good at extracting masks for image segmentation. Using MLP, a 5 × 5 window is slide over the feature maps to generate an object segment (bộ phận) of dimension 14 × 14 segments. Later, we merge masks at a different scale to form our final mask predictions.

![image](https://user-images.githubusercontent.com/80739312/112413003-0a070900-8d52-11eb-883d-bfa97b7ecce5.png)  

## Resultst
Placing (đặt) PN in RPN improves (cải thiện) R (average recall: the ability to capture objects) to 56.3, an 8.0 points improvement over the RPN baseline. The performance on small objects is increased by a large margin of 12.9 points.

![image](https://user-images.githubusercontent.com/80739312/112413058-24d97d80-8d52-11eb-9746-297412401b0b.png)  

FPN is very competitive (cạnh tranh) with state-of-the-art detectors. In fact, it beats the winners for the COCO 2016 and 2015 challenge.

![image](https://user-images.githubusercontent.com/80739312/112413118-39b61100-8d52-11eb-811e-a5b99f8f7884.png)

## Lessons learned
Here are some lessons learned from the experimental (thực nghiệm) data.
* Adding more anchors on a single high-resolution feature map layer is not sufficient (đủ/đầy đủ) to improve accuracy.
* Top-down pathway restores resolution with rich semantic information.
* But we need lateral connections to add more precise object spatial information back./
* Top-down pathway plus lateral connections improve accuracy by 8 points on COCO dataset. For small objects, it improves 12.9 points.
