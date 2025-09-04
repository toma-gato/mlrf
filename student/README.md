# BRATS18 dataset sample

This is a dataset based on a larger dataset available on the Internet.

As usual, I shuffled it to avoid cheating (as much as possible).

The dataset we provide contains normalized slices of brain MRIs for 4 modalities
(FLAIR, T1, T1ce, T2), along with the ground truth of tumoral regions to be detected.


## Student task
1. Using the training set (`train_data_x.dat` and `train_data_y.dat`),
  design and train a segmentation system which detects the tumoral regions.
2. Process the test set (`test_data_x.dat`)
  and submit your results (`test_data_ypred.dat`) for grading.


## Files
- `train_data_x.dat`: 256 normalized slices, one per patient,
  containing 240x240 images with 4 channels (1 for each modality),
  to be used for training.
- `train_data_y.dat`: 256 target segmentations, one per patient,
  containing 240x240 images with 1 channel (indicating tumor or clean region),
  to be used for training.
- `test_data_x.dat`: 29 normalized slices, one per patient (not in the training set),
  containing 240x240 images with 4 channels (1 for each modality),
  to be used for testing.
- `test_data_y.dat`: **available on teacher pack only** –
  29 target segmentations, one per patient,
  containing 240x240 images with 1 channel (indicating tumor or clean region),
  to be used for testing.
- `README.md`: this file.


## How to read and write those `.dat` files:

The `.dat` files we use here are binary dumps of NumPy arrays exactly as they are
represented in memory.
Each file contains exactly one array.

They are meant to be "[memmapped](https://en.wikipedia.org/wiki/Memory-mapped_file)"
because it is very useful for large files — and the original dataset is large —
as it avoid to load the complete array in RAM, but instead caches pages in RAM and
load content from disk on demand.

There are two drawbacks:
1. the files are not compressed;
2. we must open them in a special way.

To open them, we use the NumPy function `numpy.memmap(path, mode, dtype[, shape])`.

### Details about `train_data_x.dat`
This file contains 256 normalized slices, one per patient, containing 240x240 images
with 4 channels (1 for each modality), to be used for training.

Structure:
- The type of the elements in the array is `np.float32`.
- The shape of the array is `(256, 240, 240, 4)` which correspond to:
  - 256 slices, one per patient;
  - 240x240 images;
  - 4 modalities, in this order:
    1. "t1";
    2. "t1ce";
    3. "t2";
    4. "flair".

Sample code for loading this file:
```python
train_data_x = np.memmap("train_data_x.dat", mode="r", dtype=np.float32, shape=(256, 240, 240, 4))
```

About data values:
- Values are already normalized so that the average value of the voxels within the brain
  (for the whole scan, for each modality) is `0`, and standard deviation is `1`.
- Values code for the intensity of the acquisition for each modality.
- Value ranges are different for each modality.
- Background voxels are set to the minimal value for each slice, for each modality.

### Details about `train_data_y.dat`
This file contains 256 target segmentations, one per patient, containing 240x240 images
with 1 channel (indicating tumor or clean region), to be used for training.

Structure:
- The type of the elements in the array is `np.uint8`.
- The shape of the array is `(256, 240, 240)` which correspond to:
  - 256 slices, one per patient;
  - 240x240 images;
  - a binary indication of whether the current voxel should be classified as a tumor or not.

Sample code for loading this file:
```python
train_data_y = np.memmap("train_data_y.dat", mode="r", dtype=np.uint8, shape=(256, 240, 240))
```

About data values:
- A value of `0` indicate that the voxel falls within a clean region,
  and a value of `1` indicates a tumoral region.


### Details about `test_data_x.dat`
This file has exactly the same structure and meaning as the `train_data_x.dat` file.
There are two differences:
1. The slices are extracted from patients which are **not** in the training set.
2. There are only `29` slices (one for each patient in the test set).

Sample code for loading this file:
```python
test_data_x = np.memmap("test_data_x.dat", mode="r", dtype=np.float32, shape=(29, 240, 240, 4))
```

About data values:
- Values are normalized using exactly the same procedure as the one used to prepare the training set.


### Details about `train_data_y.dat`
**This file is only available in the teacher package and is used for grading students' results.**

This file has exactly the same structure and meaning as the `train_data_y.dat` file.
There are two differences:
1. The slices are extracted from patients which are **not** in the training set.
2. There are only `29` slices (one for each patient in the test set).

Sample code for loading this file:
```python
test_data_y = np.memmap("test_data_y.dat", mode="r", dtype=np.uint8, shape=(29, 240, 240))
```

Sample code for writing this file:
```python
# First open the file and create an array
test_data_ypred = np.memmap("test_data_ypred.dat", mode="w+", dtype=np.uint8, shape=(29, 240, 240))
# Then fill the values like with any array
test_data_ypred[0] = SOME_VALUES
# The buffer is automatically flushed on disk upon object destruction but can also be explicit
test_data_ypred.flush()
del test_data_ypred
```
