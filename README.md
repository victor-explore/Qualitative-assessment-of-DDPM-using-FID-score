# FID for DDPM

This repository contains the implementation of Fréchet Inception Distance (FID) calculation for Denoising Diffusion Probabilistic Models (DDPM). The FID score is a widely used metric to evaluate the quality of generated images by comparing the statistics of generated images to real images.

## FID Calculation

The FID calculation involves the following steps:

1. **Preprocessing**: Images are resized to 299x299 and normalized using the same statistics as the original ImageNet dataset used to train the Inception v3 model.

    ```python
    preprocess = transforms.Compose([
        transforms.Resize((299, 299)),
        transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
    ])
    
    def prepare_inception_input(images):
        return preprocess(images)
    ```

2. **Loading Inception v3 Model**: A pre-trained Inception v3 model is loaded with the final classification layer removed to extract features from the images.

    ```python
    from torchvision.models import inception_v3
    
    def load_inception_model():
        model = inception_v3(pretrained=True, transform_input=False)
        model.fc = torch.nn.Identity()
        model.eval()
        return model.to(device)
    
    inception_model = load_inception_model()
    ```

3. **Feature Extraction**: Features are extracted from both generated and real images using the Inception v3 model.

    ```python
    def extract_inception_features(image_paths):
        all_features = []
        transform = transforms.Compose([
            transforms.ToTensor(),
            transforms.Resize((299, 299)),
            transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
        ])
    
        with torch.no_grad():
            for img_path in image_paths:
                img = Image.open(img_path).convert('RGB')
                img_tensor = transform(img).unsqueeze(0).to(device)
                features = inception_model(img_tensor)
                all_features.append(features)
    
        return torch.cat(all_features, dim=0)
    ```

4. **Calculating Statistics**: The mean and covariance of the extracted features are calculated for both generated and real images.

    ```python
    generated_mean = torch.mean(generated_features, dim=0)
    generated_cov = torch.cov(generated_features.T)
    
    real_mean = torch.mean(real_features, dim=0)
    real_cov = torch.cov(real_features.T)
    ```

5. **Computing FID**: The FID score is computed using the mean and covariance of the features.

    ```python
    def calculate_frechet_inception_distance(real_mean, real_cov, generated_mean, generated_cov):
        real_mean_np = real_mean.cpu().numpy()
        real_cov_np = real_cov.cpu().numpy()
        generated_mean_np = generated_mean.cpu().numpy()
        generated_cov_np = generated_cov.cpu().numpy()
    
        mean_diff = np.sum((real_mean_np - generated_mean_np) ** 2)
        covmean = scipy.linalg.sqrtm(real_cov_np.dot(generated_cov_np))
    
        if np.iscomplexobj(covmean):
            covmean = covmean.real
    
        trace_term = np.trace(real_cov_np + generated_cov_np - 2 * covmean)
        fid = mean_diff + trace_term
    
        return fid
    
    fid_score = calculate_frechet_inception_distance(real_mean, real_cov, generated_mean, generated_cov)
    print(f"Fréchet Inception Distance: {fid_score:.4f}")
    ```

## Usage

1. Clone the repository.
2. Ensure you have the required dependencies installed.
3. Run the notebook `FID for DDPM.ipynb` to generate images and calculate the FID score.

This implementation provides a comprehensive approach to evaluate the quality of images generated by DDPM using the FID metric.


