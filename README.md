# VideoSimilarity

## My experiments on similarity metrics for videos (and images)

### Color based similarity

A Faster - *GPU* based implementation of **k-means clustering** - is used for getting the dominant color.

I used Facebook’s [**faiss**](https://github.com/facebookresearch/faiss) GPU implementation and compared it with scikit-learn’s vanilla [k-means](http://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html). Below is a table of speeds.

**Useful Links:**
1. [Compares Python, C++ and Cuda implementations of k-means](http://www.goldsborough.me/c++/python/cuda/2017/09/10/20-32-46-exploring_k-means_in_python,_c++_and_cuda).

![Runtime of k-means clustering on a 200x200x3 image (seconds)](https://github.com/CoderHam/VideoSimilarity/blob/master/plots/chart.png)

As we can see, using a GPU gives a massive performance boost in this case from **13x** to nearly **500x** as the cluster size (k) increases from **5** to **50**. We see sklearn takes exponentially more time and a GPU definitely speeds up this process. Although the algorithms are not exactly the same, the performance and quality of results are comparable and the speedup is worth the minor loss in accuracy.

**PS:** _In all experiments, the runtime includes the time to load the image and cluster it._

Then we perform a similarity search using these dominant colors to find similar colored images and by concatenating dominant colors for multiple frames, we find similar colored videos.

**Explanation:**

We get such a **1x100** image for each of the images we cluster i.e. we cluster the resized image of size **200x200** and create a smaller image of size **100 (0.25%)** that contains its dominant colors in descending order of dominance.

![Bar of dominant colors ordered by cluster size](https://github.com/CoderHam/VideoSimilarity/blob/master/plots/clustered_bar.png)

The next step is to concatenate these **100** pixel images for each frame (samples at **n** fps from the video of length **L** seconds) to create a new image of size **L x n x 100** pixel image that will represent the entire video.

This compressed image representation is thereafter used to find similar images using either the **MSE** or **SSIM** similarity metric or by using the **KNN** algorithm.

PS: For the current experiments, **L** and **n** is variable but I have used a script to extract a fixed number of frames **(20)** for each video. Thus the new image representation is **20 x 100 x 3** [since there are 3 channels (RGB)].

I have extracted this image representation for the videos in [`data/videos`](https://github.com/CoderHam/VideoSimilarity/tree/master/data/videos) and will now use KNN to return k-similar neighbors.

### KNN similarity search

We cluster smaller images / features and use the faiss - *GPU* implementation instead of sklearn and store them in [`data/vid2img`](https://github.com/CoderHam/VideoSimilarity/tree/master/data/vid2img).

For **10 x 7 = 70** videos, the process took **437 s** to execute i.e. approx **6.2 s** for video. This process includes:

1. Extracting frames from videos and writing them to disk.
2. Clustering the extracted frames (20 per video).
3. Converting the histogram of clusters into an image and writing to disk.

Thereafter, I ran the KNN GPU implementation on the image representations in [`data/vid2img`](https://github.com/CoderHam/VideoSimilarity/tree/master/data/vid2img).

The dimensionality is **6000** since (*20 x 100 = 2000* pixels and RBG (*3*) values for each pixel).

The runtime for the KNN search with **k = 3** (build and run on all videos) for this subset of **70** videos is **132 ms** and we can be sure that this will scale effectively based on previous experiments.

### Extra - Using Wavelet image hash for similarity search (Not currently using):

https://fullstackml.com/wavelet-image-hash-in-python-3504fdd282b5 - Uses [imagehash](https://pypi.org/project/ImageHash/), a python library to compute 1 of 4 different hashes and use hashes for comparison
