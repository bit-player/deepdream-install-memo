## Installing the Deep Dreaming software stack

In June 2015 Alexander Mordvintsev, Christopher Olah, and Mike Tyka posted a batch of their "Deep Dreaming" images on the [Google Research Blog](http://googleresearch.blogspot.com/2015/06/inceptionism-going-deeper-into-neural.html). I was pretty amazed, and when they [released their software](http://googleresearch.blogspot.com/2015/07/deepdream-code-example-for-visualizing.html) a few weeks later, I knew I would have to give it a try. I've now got the program running, and so I can make weird-and-wonderful pictures like this:

![Tent Rocks National Monument, New Mexico](tentrocks-6oct-oscale14-iter15-inception_4b-output-1.jpeg)

But it wasn't easy getting here. The program released by Mordvintsev, Olah, and Tyka is an IPython notebook with about 100 lines of code, available in this [GitHub repository](https://github.com/google/deepdream). However, the program depends on the Caffe neural network framework, which in turn has some other major dependencies, such as the Boost C++ library and the OpenCV library for computer vision. Getting all the pieces properly installed and communicating with one another was a bit of a *bad* dream. I made several attempts to get it running on my everyday computer, then switched to a different machine that had a little less cruft in its dark corners. That effort also failed. Annoyed and chagrined, I decided to try again on a completely fresh disk partition, with a newly installed operating system. Success at last.

This memo documents the steps I followed in the final successful install. I am recording it in detail in case I ever need to do it again, and making it public in case anyone else might benefit. Note that this is a record of what worked for me; it's not necessarily the best way or the only way. Listed below are several other installation guides. I have taken bits and pieces from several of them.

### Other guides to installation and troubleshooting

* [Google installation instructions](https://github.com/google/deepdream/blob/master/dream.ipynb)
* [Official Caffe installation page](http://caffe.berkeleyvision.org/installation.html)
* [Official Caffe installation page for OS X](http://caffe.berkeleyvision.org/install_osx.html)
* [Installing Caffe the right way](http://installing-caffe-the-right-way.wikidot.com/start)
* [How to install Caffe on Mac OS X 10.10 for dummies (like me)](http://hoondy.com/2015/04/03/how-to-install-caffe-on-mac-os-x-10-10-for-dummies-like-me/)
* [robertsdionne Deepdream installation gist](https://gist.github.com/robertsdionne/f58a5fc6e5d1d5d2f798)
* [kylemcdonald Theory of Building Caffe on OS X](https://gist.github.com/kylemcdonald/0698c7749e483cd43a0e)
* [playittodeath How to install Caffe on Mac (OS X Yosemite 10.10.4)](http://playittodeath.ru/how-to-install-caffe-on-mac-os-x-yosemite-10-10-4/)
* [ryankennedy Running deep dream on Windows and OSX](http://ryankennedy.io/running-the-deep-dream/)
* [VISIONAI Deepdreaming in the clouds: A Dockerized deepdream Guide](https://github.com/VISIONAI/clouddream)
* [jcjohnson cnn-vis](https://github.com/jcjohnson/cnn-vis)

### Hardware environment

Apple MacBook Pro 17-inch, late 2011. 2.4 GhZ Core i7, 16GB RAM. The GPU is an AMD Radeon rather than an Nvidia part, so there's no hope of running Cuda, and I did not install the Cuda interface for Caffe.

New, empty 300 GB partition on a 750 GB disk.

### Set up the software environment

From the App Store, install OS X Yosemite 10.10.5 (14F27). After reboot onto the new partition, create admin account.

From the App Store download and install Xcode (Version 6.4 (6E35b)). Also install the Xcode command-line tools:

    xcode-select --install

Install [Homebrew](http://brew.sh/), the package manager:

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

Set the PATH so we can find executables in usr/local/bin:

    export PATH=/usr/local/bin:/usr/bin:/usr/sbin:/bin:/sbin:
    
Make sure Homebrew is working:

    brew doctor

(I got a warning about an outdated XQuartz, which I ignored.)

Install Python 2.7 and pip via Homebrew:

    brew install python pip

Homebrew installs several dependencies ()pkg-config, readline, sqlite, gdbm, openssl), then Python 2.7.10_2 and the pip utility for managing more Python packages.

Note that OS X comes with a preinstalled Python. As a matter of fact, it has four of them, versions 2.3 through 2.7. And I ordinarily use a different Python distribution, from Anaconda. But I suspect most of the trouble I was having in earlier failed attempts could be traced back to stray linkages to packages from the wrong Python version. It seemed safest and simplest for this experiment to run the Homebrew install.

### Preparing to install Caffe

Following the [installation instructions](http://caffe.berkeleyvision.org/install_osx.html) for Caffe, we start with some dependencies:

    brew install -vd snappy leveldb gflags glog szip lmdb
    
Snappy is a compression library; LevelDB is a key-value store; Gflags is a utility for setting command-line flags; glog is a logging facility; szip does lossless compression of HDF5 files; lmdb (the Lightning Memory-Mapped Database) is another key-value store. (Why do we need two key-value databases? I'm sure there's a perfectly good reason.)

    brew tap homebrew/science

[Homebrew/science](https://github.com/Homebrew/homebrew-science/blob/master/opencv.rb) is a repository of Homebrew install scripts for about 500 packages of interest in scientific computing. This command downloads the scripts, making the packages available for installation. Below we'll be installing two of them, hdf5 and opencv.

    brew install hdf5
    
[HDF (Hierarchical Data Format)](https://www.hdfgroup.org/HDF5/) is a library and file format for large and complex data sets.

    brew install opencv

[OpenCV](http://opencv.org/about.html) is the Open Source Computer Vision Library. It's large and complex, and it has dependencies that are also large and complex. One of those dependencies is a patched version of gcc, the Gnu C compiler, which has to be compiled from source. The whole install process takes about an hour.

    brew install --build-from-source -with-python -vd protobuf

"[Protocol buffers](https://developers.google.com/protocol-buffers/?hl=en) are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data — think XML, but smaller, faster, and simpler."

As the command indicates, this is another compile-from-source project. The '-vd' flags request verbose output and debugging information, which might be useful if anything goes wrong. I found the output was indeed verbose but not all that helpful: 2,200 lines, most of them having to do with localization. I'm relieved to know I've got zoneinfo set for any of the 12 places in Antarctica I might be visiting.

    # brew install --build-from-source -vd boost boost-python
    
    brew install --build-from-source -vd \
         https://raw.githubusercontent.com/Homebrew/homebrew/6fd6a9b6b2f56139a44dd689d30b7168ac13effb/Library/Formula/boost.rb \
         https://raw.githubusercontent.com/Homebrew/homebrew/3141234b3473717e87f3958d4916fe0ada0baba9/Library/Formula/boost-python.rb

The commented version of this command installs the current version of the boost library and its python wrapper. At the time I was doing this work, the current version was 1.58, which some sources indicated was incompatible with Caffe. The alternative version of the command with full URLs installs version 1.57. (I scarfed this command from [robertsdionne](https://gist.github.com/robertsdionne/f58a5fc6e5d1d5d2f798).)

In this case I would really suggest omitting the '-vd' option flags. I got more than 115,000 lines of output. I didn't read it all.

### Install and build the Caffe framework

Finally we can get started with [Caffe](http://caffe.berkeleyvision.org/) itself.

    git clone https://github.com/BVLC/caffe.git
    cd caffe
    
But hold on. We've got still more requirements to attend to.

    cat python/requirements.txt 
    Cython>=0.19.2
    numpy>=1.7.1
    scipy>=0.13.2
    scikit-image>=0.9.3
    matplotlib>=1.3.1
    ipython>=3.0.0
    h5py>=2.2.0
    leveldb>=0.191
    networkx>=1.8.1
    nose>=1.3.0
    pandas>=0.12.0
    python-dateutil>=1.4,<2
    protobuf>=2.5.0
    python-gflags>=2.0
    pyyaml>=3.10
    Pillow>=2.3.0
    six>=1.1.0folio:python bph$ 

Some of these were already installed above. Nevertheless, it's easiest to hand the whole list to pip:

    pip install --requirement python/requirements.txt
    
### Customize the Caffe makefile

Now we need to start mucking about with the Caffe makefile. First, make a renamed copy of the example file distributed with the package.

    cp Makefile.config.example Makefile.config

The next steps call for editing Makefile.config. You can do this in a terminal window using nano or another line editor, or else open the file with a freestanding editor. Here are the four changes:

Line 8. Uncomment CPU_ONLY := 1, since I have no GPU that will run Cuda.

    # CPU_ONLY := 1  ==>  CPU_ONLY := 1

Lines 51-52. Caffe needs access to some of the Python header files. Make sure it finds the right ones, namely those for the Homebrewed Python, not those of the System Python.

    remove or comment out:
    
    PYTHON_INCLUDE := /usr/include/python2.7 \
		/usr/lib/python2.7/dist-packages/numpy/core/include

	replace with:
	
	PYTHON_INCLUDE := /usr/local/lib/python2.7/site-packages/numpy/core/include/ \
       /usr/local/Cellar/python/2.7.10_2/Frameworks/Python.framework/Versions/2.7/include/python2.7/

Line 61. Likewise Caffe wants to know where to look for the Python dynamically loaded library. 

    remove or comment out:
    
    PYTHON_LIB := /usr/lib

	replace with:
	
	PYTHON_LIB := /usr/local/lib/python2.7 \
       /usr/local/Cellar/python/2.7.10_2/Frameworks/Python.framework/Versions/2.7/lib/ \
       /usr/local/lib/python2.7/site-packages/numpy/lib

Line 79. Finally, make sure the library path doesn't include the dreaded usr/lib, where the System Python libraries are found.

    remove or comment out:
    
    LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib

	replace with:
	
	LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib
	
Save the file. 

### Compile and test Caffe

Now we're ready to get Caffe running.

    make clean
    make all -j8

The `make clean` command shouldn't actually be needed if this is your first try at compiling the program, but it does no harm. The '-j8' option specifies the number of cores to use in running the job. You may see warnings about "argument unused during compilation: '-pthread'"; they are ignorable.

The test suite gets built separately:

    make test

Then yet another makefile takes care of running the tests:

    make runtest

At this point you should see a long list of test results, hopefully concluding with something like `[  PASSED  ] 880 tests.` Congratulations. Caffe is now installed and working. But we're not done yet. The next chore is to build the Python interface for Caffe.

    make pycaffe -j8

Then we move the essential binaries and runtime components out of the '.build_release' directory into a 'distribute' directory.

    make distribute

A couple more PATH additions, and we should be done with Caffe:

    export PYTHONPATH=/Users/bph/caffe/distribute/python
    export DYLD_FALLBACK_LIBRARY_PATH=/usr/local/lib:/Users/bph/caffe/distribute/lib:$DYLD_FALLBACK_LIBRARY_PATH

### Get the Deep Dream IPython notebook

When we were setting up the Python environment, we neglected to include the components for IPython notebook. So:

    pip install ipython[notebook]

Now get the Deep Dream notebook and its ancilary files from the Google repo.

    git clone https://github.com/google/deepdream.git
    
At long last, we can now launch that IPython notebook:

    cd deepdream
	ipython notebook

The Jupyter (a.k.a IPython) home page will open in a browser window, with a file listing of the deepdream directory. Click on `dream.ipynb`, and that notebook will open in another browser tab or window.

### Get the trained Caffe models

We're so close, but we're not quite finished yet. We have the Caffe neural-network software, and everything needed to run it, but we don't have a trained neural network model. The models are fairly large (about 250 MB), and so they are not distributed with Caffe itself. We can install the GoogLeNet model as follows:

	cd ..
	pwd   #make sure you're in the caffe directory
	scripts/download_model_binary.py models/bvlc_googlenet

(A few more models are also available in the [Model Zoo](http://caffe.berkeleyvision.org/model_zoo.html) and can be installed in the same way. See also the [Model Zoo wiki](https://github.com/BVLC/caffe/wiki/Model-Zoo) for other models, which have to be manually installed.)

Final step: In the second code cell of the `dream` notebook, under the heading "Loading DNN model", insert the path to the `bvlc_googlenet` model you've just downloaded.

Final final step: Have fun!
