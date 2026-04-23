3.1 DICOM Image Acquisition and Pre-processing
CT scans in DICOM format were uploaded and read using the pydicom library. Pixel
intensities were extracted from the DICOM object and converted to 32-bit floating-point
format. The original image was visualized to assess contrast, noise level, and anatomical
coverage.
To ensure consistency across neural network inputs, each image underwent the following
pre-processing steps: Intensity normalization, Conversion to NumPy format ,Preparation
for downstream denoising and CNN inference
3.2 Noise Reduction using Non-Local Means Denoising
To evaluate how noise influences deep neural networks, the original image was denoised
using the Non-Local Means (NLM) algorithm from scikit-image. The process involved:
● Normalizing image intensities to the range [0,1]
● Estimating noise level using estimate_sigma
● Applying NLM with patch-based similarity filtering
● Restoring intensities back to the original dynamic range
This generated a pair of images for every scan:
Original (noisy) CT image, Denoised CT image.These were used for all subsequent
comparisons.To evaluate denoising performance, pixel-level metrics such as MSE,
PSNR, and SNR-estimate were computed by comparing the original and denoised
versions (treating the denoised output as an approximation of the underlying clean signal
3.3 Image Preparation for CNN Processing
Deep neural networks require fixed-size, multi-channel inputs. Therefore, both original
and denoised images were converted into model-compatible tensors through the
following steps:
● Resizing to 224 × 224 using OpenCV
● Re-normalizing intensities to [0,1]
● Replicating the single-channel CT slice into 3 channels
● Applying ImageNet mean–std normalization
● Converting to PyTorch tensors and batching
This ensured compatibility with standard pretrained CNN architectures.
3.4 CNN Model: ResNet-18 Pretrained on ImageNet
A pretrained ResNet-18 was selected as a feature extractor to study how a deep CNN responds to
noisy versus denoised CT images. The model was loaded with ImageNet weights and used strictly
in inference mode, without any fine-tuning.
A forward hook was registered on the final convolutional layer of ResNet-18 to extract:
Final convolutional feature maps (512 × 7 × 7)
Model logits and predicted class
This allowed deeper analysis of the internal CNN representations for both image conditions.
3.5 Model Output Comparison: Logit Difference and
Feature Maps
The model’s raw output logits were computed for both original and denoised inputs. The final
class score (logit) was extracted for the predicted class and used to quantify prediction confidence
shift:
ΔConfidence =Logit denoised − Logit original
A negative shift indicates the model became less confident after noise removal, suggesting that
noise contributed to its decision-making process.
Feature maps captured through the forward hook were compared to assess structural differences in
CNN activation patterns between the two conditions.
3.6 Explainability using Grad-CAM
To understand how noise influences CNN attention, Grad-CAM was applied to both
image versions. The method computes the gradient of the output logit with respect to the
final convolution feature maps, producing a class-specific heatmap of regions most
responsible for the prediction.
Grad-CAM outputs include:
● Original image + Grad-CAM heatmap
● Denoised image + Grad-CAM heatmap
This enabled a direct visual comparison of how the model’s focus shifts after denoising.
Typically:
● Original images showed wider, noisier attention regions
● Denoised images showed tighter, anatomically relevant focus
3.7 Experimental Summary
The complete methodology allowed us to evaluate:
● How denoising modifies pixel-level image quality
● How CNN prediction confidence changes after noise removal
● How feature maps differ between the two conditions
● How Grad-CAM attention patterns shift, confirming whether CNNs rely on image
noise
This multi-stage analysis provides both quantitative and qualitative evidence to support the
research question on noise-dependent AI behavior in medical imaging systems.
