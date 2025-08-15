# Inference Scripts for Adversarial MNIST

This directory contains scripts to run inference on adversarial MNIST datasets using pre-trained topological autoencoder models.

## Files

- `inference_adversarial_mnist.py` - Standard inference script for command line use
- `kaggle_inference_adversarial_mnist.py` - Kaggle-compatible version for notebooks
- `README_inference.md` - This file

## Prerequisites

1. **Trained MNIST Model**: You need a pre-trained MNIST topological autoencoder model (`.pth` file)
2. **Adversarial Dataset**: Your adversarial MNIST data stored as `.pt` files
3. **Dependencies**: The topological autoencoders codebase must be available

## Dataset Structure

Your adversarial dataset should be organized as follows:
```
DATASETS/
└── purification/
    └── medmnist/
        └── mnist/
            ├── cw strong/
            │   ├── batch_0.pt
            │   ├── batch_1.pt
            │   └── ...
            ├── fgsm strong/
            │   ├── batch_0.pt
            │   ├── batch_1.pt
            │   └── ...
            ├── fgsm weak/
            │   └── ...
            ├── pgd strong/
            │   └── ...
            └── pgd weak/
                └── ...
```

## Usage

### Option 1: Command Line (Standard Script)

```bash
# Modify the paths in the script first
python scripts/inference_adversarial_mnist.py
```

### Option 2: Kaggle Notebook

```python
# Import the script
from scripts.kaggle_inference_adversarial_mnist import process_all_attacks, extract_latents_on_adversarial_dataset

# Set your paths
model_path = "/kaggle/input/your-mnist-model/mnist_model.pth"
base_data_dir = "/kaggle/input/your-adversarial-dataset/DATASETS/purification/medmnist/mnist"
output_dir = "/kaggle/working/adversarial_mnist_results"

# Process all attack types
process_all_attacks(model_path, base_data_dir, output_dir)

# Or process a specific attack type
extract_latents_on_adversarial_dataset(
    model_path, 
    base_data_dir, 
    output_dir, 
    attack_type='cw strong'
)
```

## Configuration

The scripts use the default MNIST model configuration:
- **Model**: `TopologicallyRegularizedAutoencoder`
- **Autoencoder**: `DeepAE`
- **Lambda**: `0.5002972000959738`
- **Topology**: `match_edges: "symmetric"`
- **Batch Size**: `126`

## Output

For each attack type, you'll get:
- `adversarial_mnist_{attack_type}_latents.csv` - CSV with 2 latent dimensions + labels
- `adversarial_mnist_{attack_type}_complete.npz` - Complete data including:
  - `latents`: 2D latent representations
  - `labels`: Class labels
  - `original_images`: Original adversarial images
  - `reconstructed_images`: Reconstructed images from the autoencoder

Example CSV format:
```csv
0,1,labels
-0.123,0.456,7
0.789,-0.321,2
...
```

Example NPZ data structure:
```python
# Load the complete data
data = np.load('adversarial_mnist_cw_strong_complete.npz')

# Access different components
latents = data['latents']           # Shape: (n_samples, 2)
labels = data['labels']             # Shape: (n_samples,)
original = data['original_images']   # Shape: (n_samples, 1, 28, 28)
reconstructed = data['reconstructed_images']  # Shape: (n_samples, 1, 28, 28)
```

## Features

- **Automatic batch loading**: Combines multiple `.pt` files automatically
- **Flexible data formats**: Handles dict, tuple, or direct tensor formats
- **Data preprocessing**: Automatically normalizes and reshapes data for MNIST model
- **Error handling**: Continues processing even if one attack type fails
- **Kaggle compatible**: Automatically creates output directories
- **Complete data extraction**: Saves both latent representations AND reconstructed images

## Troubleshooting

### Import Errors
Make sure the `src` directory is in your Python path. The Kaggle script automatically tries to find it.

### Data Format Issues
The script handles common data formats, but if you get errors, check:
1. Your `.pt` files contain the expected data structure
2. Data dimensions are compatible with MNIST (28x28 images)
3. Labels are properly formatted

### Memory Issues
If you have very large datasets, consider:
1. Processing one attack type at a time
2. Reducing batch size
3. Using GPU if available

## Example Output

```
Found 106 .pt files
Loading batch_0.pt...
Loading batch_1.pt...
...
Final data shape: (10600, 1, 28, 28)
Final labels shape: (10600,)
Data range: [-1.000, 1.000]
Unique labels: [0 1 2 3 4 5 6 7 8 9]

Processing cw strong...
==================================================
Loading pre-trained MNIST Topological Autoencoder...
Loading model weights from /path/to/model.pth
Creating dataset from /path/to/data/cw strong...
Extracting latent representations...
Latent space shape: (10600, 2)
Labels shape: (10600,)
Saving results...
Saved to /path/to/output/adversarial_mnist_cw_strong_latents.csv and /path/to/output/adversarial_mnist_cw_strong_latents.npz
Successfully processed cw strong: 10600 samples
``` 