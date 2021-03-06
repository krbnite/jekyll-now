---
title: Intel at the Edge (Installing Intel's OpenVINO on MacBook w/ 4th Generation Intel Core i7)
layout: post
tags: edge ai easi
---


When first considering installing Intel's OpenVINO on my MacBook in 
[my last post](https://krbnite.github.io/Intel-at-the-Edge-Getting-Started/), I quickly determined
that I did not meet the minimum specs required for running Intel's OpenVINO... Or so 
I thought.

You see, on their website, Intel says you will need one of the following processors:
* 6th-10th Generation Intel® Core™
* Intel® Xeon® v5 family
* Intel® Xeon® v6 family

But my MBP Mid-2015 only has a 4th Generation Core i7 (2.8 GHz).  "Sad face!" I exclaimed. "Now I will
have to order the Neural Compute Stick 2."  

Now, I don't regret ordering the NCS2 because I do plan on playing with it, and possibly even
buying a Rasberry Pi to play on as well.  That said, after I ordered the NCS2, the thought
occurred to me: "Why don't I try downloading and running everything anyway... Just to see what
happens."

So I did, and I found out that -- even with a shabby ol' 4th Gen processor -- you can 
play around with Intel's OpenVINO.  I'm sure it's not as fast or efficient of an experience 
that Intel wants you to have, but it works.   

Here are some notes on getting it up and running.  

## Download & Install
First, you will 
[download latest installation package for MacOS](https://software.intel.com/en-us/openvino-toolkit/choose-download/free-download-macos?elq_cid=6019966&erpm_id=9101489).  In this step, 
you will also have to register your downloaded software and obtain a serial number (though I found that I never had to
plug in this serial number anywhere downstream in the installation process).  

Begin the installation process in the usual way (i.e., double click on the thing you just downloaded).  When 
the first part of installation is complete, you will receive a message -- something like:
> The first part of Intel® Distribution of OpenVINO™ toolkit 2019 R3.1 for macOS* has been successfully 
> installed in `/opt/intel/openvino_2019.3.376`.

You will also get notified that there are additional steps to be taken and get redirected to 
an [Intel webpage with further instructions](https://docs.openvinotoolkit.org/2019_R3.1/_docs_install_guides_installing_openvino_macos.html#set-the-environment-variables).  

## Additional Installation Steps
After the initial installation step, there are a few things left to do:
  - Install External Software Dependencies
  - Set Environment variables 
  - Configure Model Optimizer
  - Run the Verification Scripts to Verify Installation and Compile Samples


### Environment Variables
To ensure the proper environment variables are set in your current Terminal session: 
```
source /opt/intel/openvino/bin/setupvars.sh
```

To ensure the proper environment variables are always set, plug that command into your
Bash profile. You can also wrap it in a easy-to-remember function so that the command
is only run when desired:

```
function intelopenvino {
  source /opt/intel/openvino/bin/setupvars.sh
}
```

### Model Optimizer (Ensure Prerequisites are Installed)
Now, we must make sure the model optimizer can work with the deep learning frameworks
of our choice.  First, head into the Model Optimizer section of the OpenVINO directory:

```
cd /opt/intel/openvino/deployment_tools/model_optimizer/install_prerequisites
```

Here, you can choose to make sure the Model Optimizer can work with all supported frameworks,
or just the supported frameworks you are interested in.

To configure the Model Optimizer for all available deep learning frameworks (Caffe, TensorFlow, MXNet, Kaldi*, and ONNX): 
```
sudo ./install_prerequisites.sh
```

To just configure for one framework, e.g., TensorFlow:
```
sudo ./install_prerequisites_tf.sh
```

#### Troubleshooting Notes
I ran into a few issues. 

First, I had to upgrade pip:  
```
pip install --upgrade pip
```

It is often recommended to use virtual environments for different Python use cases and setups.  One way to do
this is using Python's `virtualenv` package... Another is using conda environments, which I've been doing for the
past few years...  However, I guess I haven't created any new environments in a while because I received
notifications stating my conda software was out of date.  But when I went to `conda update conda`, I received
error messages stating that I couldn't delete `pip`... Long story short, I googled around, found I had
uninstall pip, then reinstall it with conda...and a few other things.  After all was said and done, I was
able to create a new conda environment.

```
conda create --name intel
conda activate intel
```

Finally, when I installed Model Optimizer prerequisites for all supported deep learning frameworks, 
I got some warning messages at the beginning of the screen output:
```
WARNING: The directory '/Users/kevinurban/Library/Caches/pip/http' or its parent directory is not owned by the current  user and the cache has been disabled. Please check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
WARNING: The directory '/Users/kevinurban/Library/Caches/pip' or its parent directory is not owned by the current user and caching wheels has been disabled. check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
```

And at the end, I received some as well:
```
All Model Optimizer dependencies are installed globally.
[WARNING] If you want to keep Model Optimizer in separate sandbox run install_prerequisites.sh 
[WARNING] venv {caffe|tf|mxnet|kaldi|onnx}
```

# Test Demos
The site says if you installed as root, make sure to go into root mode first (`sudo -i`).  I
thought I installed as root, but this
put me into an environment that no longer had the necessary dependences (it took me out of my
conda environment.  Note that running this without `sudo` results in errors as well.  However, by 
just using the `sudo` prefix things worked.


## Image Classification w/ SqueezeNet
```
cd /opt/intel/openvino/deployment_tools/demo
sudo ./demo_squeezenet_download_convert_run.sh
```

If everything runs smoothly, you should get some probability scores classifying an image of a car:
```
Top 10 results:

Image /opt/intel/openvino_2019.3.376/deployment_tools/demo/car.png

classid probability label
------- ----------- -----
817     0.6853030   sports car, sport car
479     0.1835197   car wheel
511     0.0917197   convertible
436     0.0200694   beach wagon, station wagon, wagon, estate car, beach waggon, station waggon, waggon
751     0.0069604   racer, race car, racing car
656     0.0044177   minivan
717     0.0024739   pickup, pickup truck
581     0.0017788   grille, radiator grille
468     0.0013083   cab, hack, taxi, taxicab
661     0.0007443   Model T
```

From Intel's website:
> "The Image Classification verification script downloads a public SqueezeNet Caffe* model and runs the Model 
> Optimizer to convert the model to .bin and .xml Intermediate Representation (IR) files. The Inference Engine requires 
> this model conversion so it can use the IR as input and achieve optimum performance on Intel hardware.
>
> This verification script creates the directory /home/<user>/inference_engine_samples/, builds the Image Classification
> Sample application and runs with the model IR and car.png image located in the demo directory. When the verification 
> script completes, you will have the label and confidence for the top-10 categories."


From the README file in the demo directory:
> The demo illustrates the general workflow of using the Intel(R) Deep Learning Deployment Toolkit and performs 
> the following:
>
>  - Downloads a public SqueezeNet model using the Model Downloader (open_model_zoo\tools\downloader\downloader.py)
>  - Installs all prerequisites required for running the Model Optimizer using the scripts from the "model_optimizer\install_prerequisites" folder
>  - Converts SqueezeNet to an IR using the Model Optimizer (model_optimizer\mo.py) via the Model Converter (open_model_zoo\tools\downloader\converter.py)
>  - Builds the Inference Engine classification_sample (inference_engine\samples\classification_sample)
>  - Runs the sample with the car.png picture located in the demo folder


If you look at the screen output when running the demo, you can see a command that is used
for running the model:
```
./classification_sample_async -d CPU -i /opt/intel/openvino_2019.3.376/deployment_tools/demo/car.png -m /Users/<userName>/openvino_models/ir/public/squeezenet1.1/FP16/squeezenet1.1.xml
```

You will find the IR files for these models in:
```
~/openvino_models/ir/public/
```


It's pretty easy to guess at what the various flags do:
* -d: which device to use
* -i: where is the input coming from
* -m: which model to use


Looking at the Bash script itself is very instructive:

```
#!/usr/bin/env bash

# Copyright (C) 2018-2019 Intel Corporation
# SPDX-License-Identifier: Apache-2.0

ROOT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

. "$ROOT_DIR/utils.sh"

usage() {
    echo "Classification demo using public SqueezeNet topology"
    echo "-d name     specify the target device to infer on; CPU, GPU, FPGA, HDDL or MYRIAD are acceptable. Sample will look for a suitable plugin for device specified"
    echo "-help            print help message"
    exit 1
}

trap 'error ${LINENO}' ERR

target="CPU"

# parse command line options
while [[ $# -gt 0 ]]
do
key="$1"

case $key in
    -h | -help | --help)
    usage
    ;;
    -d)
    target="$2"
    echo target = "${target}"
    shift
    ;;
    -sample-options)
    sampleoptions="$2 $3 $4 $5 $6"
    echo sample-options = "${sampleoptions}"
    shift
    ;;
    *)
    # unknown option
    ;;
esac
shift
done

target_precision="FP16"

printf "target_precision = ${target_precision}\n"

models_path="$HOME/openvino_models/models"
models_cache="$HOME/openvino_models/cache"
irs_path="$HOME/openvino_models/ir"

model_name="squeezenet1.1"

target_image_path="$ROOT_DIR/car.png"

run_again="Then run the script again\n\n"
dashes="\n\n###################################################\n\n"


if [ -e "$ROOT_DIR/../../bin/setupvars.sh" ]; then
    setupvars_path="$ROOT_DIR/../../bin/setupvars.sh"
else
    printf "Error: setupvars.sh is not found\n"
fi

if ! . $setupvars_path ; then
    printf "Unable to run ./setupvars.sh. Please check its presence. ${run_again}"
    exit 1
fi

# Step 1. Download the Caffe model and the prototxt of the model
printf "${dashes}"
printf "\n\nDownloading the Caffe model and the prototxt"

cur_path=$PWD

printf "\nInstalling dependencies\n"

if [[ -f /etc/centos-release ]]; then
    DISTRO="centos"
elif [[ -f /etc/lsb-release ]]; then
    DISTRO="ubuntu"
fi

if [[ $DISTRO == "centos" ]]; then
    sudo -E yum install -y centos-release-scl epel-release
    sudo -E yum install -y gcc gcc-c++ make glibc-static glibc-devel libstdc++-static libstdc++-devel libstdc++ libgcc \
                           glibc-static.i686 glibc-devel.i686 libstdc++-static.i686 libstdc++.i686 libgcc.i686 cmake

    sudo -E rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm || true
    sudo -E yum install -y epel-release
    sudo -E yum install -y cmake ffmpeg gstreamer1 gstreamer1-plugins-base libusbx-devel

    # check installed Python version
    if command -v python3.5 >/dev/null 2>&1; then
        python_binary=python3.5
        pip_binary=pip3.5
    fi
    if command -v python3.6 >/dev/null 2>&1; then
        python_binary=python3.6
        pip_binary=pip3.6
    fi
    if [ -z "$python_binary" ]; then
        sudo -E yum install -y rh-python36 || true
        . scl_source enable rh-python36
        python_binary=python3.6
        pip_binary=pip3.6
    fi
elif [[ $DISTRO == "ubuntu" ]]; then
    sudo -E apt update
    print_and_run sudo -E apt -y install build-essential python3-pip virtualenv cmake libcairo2-dev libpango1.0-dev libglib2.0-dev libgtk2.0-dev libswscale-dev libavcodec-dev libavformat-dev libgstreamer1.0-0 gstreamer1.0-plugins-base
    python_binary=python3
    pip_binary=pip3

    system_ver=`cat /etc/lsb-release | grep -i "DISTRIB_RELEASE" | cut -d "=" -f2`
    if [ $system_ver = "18.04" ]; then
        sudo -E apt-get install -y libpng-dev
    else
        sudo -E apt-get install -y libpng12-dev
    fi
elif [[ "$OSTYPE" == "darwin"* ]]; then
    # check installed Python version
    if command -v python3.7 >/dev/null 2>&1; then
        python_binary=python3.7
        pip_binary=pip3.7
    elif command -v python3.6 >/dev/null 2>&1; then
        python_binary=python3.6
        pip_binary=pip3.6
    elif command -v python3.5 >/dev/null 2>&1; then
        python_binary=python3.5
        pip_binary=pip3.5
    else
        python_binary=python3
        pip_binary=pip3
    fi
fi

if ! command -v $python_binary &>/dev/null; then
    printf "\n\nPython 3.5 (x64) or higher is not installed. It is required to run Model Optimizer, please install it. ${run_again}"
    exit 1
fi

if [[ "$OSTYPE" == "darwin"* ]]; then
    $pip_binary install -r $ROOT_DIR/../open_model_zoo/tools/downloader/requirements.in
else
    sudo -E $pip_binary install -r $ROOT_DIR/../open_model_zoo/tools/downloader/requirements.in
fi

downloader_dir="${INTEL_OPENVINO_DIR}/deployment_tools/open_model_zoo/tools/downloader"

model_dir=$("$python_binary" "$downloader_dir/info_dumper.py" --name "$model_name" |
    "$python_binary" -c 'import sys, json; print(json.load(sys.stdin)[0]["subdirectory"])')

downloader_path="$downloader_dir/downloader.py"

print_and_run "$python_binary" "$downloader_path" --name "$model_name" --output_dir "${models_path}" --cache_dir "${models_cache}"

ir_dir="${irs_path}/${model_dir}/${target_precision}"

if [ ! -e "$ir_dir" ]; then
    # Step 2. Configure Model Optimizer
    printf "${dashes}"
    printf "Install Model Optimizer dependencies\n\n"
    cd "${INTEL_OPENVINO_DIR}/deployment_tools/model_optimizer/install_prerequisites"
    . ./install_prerequisites.sh caffe
    cd $cur_path

    # Step 3. Convert a model with Model Optimizer
    printf "${dashes}"
    printf "Convert a model with Model Optimizer\n\n"

    mo_path="${INTEL_OPENVINO_DIR}/deployment_tools/model_optimizer/mo.py"

    export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=cpp
    print_and_run "$python_binary" "$downloader_dir/converter.py" --mo "$mo_path" --name "$model_name" -d "$models_path" -o "$irs_path" --precisions "$target_precision"
else
    printf "\n\nTarget folder ${ir_dir} already exists. Skipping IR generation  with Model Optimizer."
    printf "If you want to convert a model again, remove the entire ${ir_dir} folder. ${run_again}"
fi

# Step 4. Build samples
printf "${dashes}"
printf "Build Inference Engine samples\n\n"

OS_PATH=$(uname -m)
NUM_THREADS="-j2"

if [ $OS_PATH == "x86_64" ]; then
  OS_PATH="intel64"
  NUM_THREADS="-j8"
fi

samples_path="${INTEL_OPENVINO_DIR}/deployment_tools/inference_engine/samples"
build_dir="$HOME/inference_engine_samples_build"
binaries_dir="${build_dir}/${OS_PATH}/Release"

if [ -e $build_dir/CMakeCache.txt ]; then
	rm -rf $build_dir/CMakeCache.txt
fi
mkdir -p $build_dir
cd $build_dir
cmake -DCMAKE_BUILD_TYPE=Release $samples_path

make $NUM_THREADS classification_sample_async

# Step 5. Run samples
printf "${dashes}"
printf "Run Inference Engine classification sample\n\n"

cd $binaries_dir

cp -f $ROOT_DIR/${model_name}.labels ${ir_dir}/

print_and_run ./classification_sample_async -d "$target" -i "$target_image_path" -m "${ir_dir}/${model_name}.xml" ${sampleoptions}

printf "${dashes}"

printf "Demo completed successfully.\n\n"
```

### Messing Around
By importing the `car.png` file into Python, I found the image is 787x259.  
```python
from PIL import Image
Image.open('car.png')
  <PIL.PngImagePlugin.PngImageFile image mode=RGBA size=787x259 at 0x1186926A0>
```

I then randomly downloaded some car images from Google Images, with varying size and file
type (i.e., I used jpg files as well as png).  For example:

```python
Image.open('car2.jpg')
  <PIL.JpegImagePlugin.JpegImageFile image mode=RGB size=600x360 at 0x10C4C6CC0>
```

Without altering the files at all, the SqueezeNet demo was able to work.  However, I suspect
to do so, it is cropping images in some default way.  I say this b/c the results weren't always
great, e.g., for a very obvious picture of a gray car front-and-center, here are the following
probabilities:

```
Image /opt/intel/openvino_2019.3.376/deployment_tools/demo/car2.jpg

classid probability label
------- ----------- -----
581     0.3110719   grille, radiator grille
817     0.2064950   sports car, sport car
479     0.1889743   car wheel
717     0.0811309   pickup, pickup truck
751     0.0761435   racer, race car, racing car
511     0.0455605   convertible
436     0.0277504   beach wagon, station wagon, wagon, estate car, beach waggon, station waggon, waggon
656     0.0193125   minivan
609     0.0133630   jeep, landrover
864     0.0076768   tow truck, tow car, wrecker
```

Still pretty good though!

For other cars, it was great, e.g., for a 480x240 JPG of an orange, sporty car:

```
classid probability label
------- ----------- -----
817     0.9660867   sports car, sport car
511     0.0110451   convertible
751     0.0084816   racer, race car, racing car
479     0.0076236   car wheel
581     0.0024839   grille, radiator grille
436     0.0016106   beach wagon, station wagon, wagon, estate car, beach waggon, station waggon, waggon
518     0.0012010   crash helmet
656     0.0010051   minivan
468     0.0002406   cab, hack, taxi, taxicab
779     0.0000357   school bus
```

For a 318x159 JPG of a gray cat whose face was in focus front-and-center, but whose body was 
blurred out of focus:

```
Image /opt/intel/openvino_2019.3.376/deployment_tools/demo/cat.jpg

classid probability label
------- ----------- -----
281     0.3908087   tabby, tabby cat
285     0.3239622   Egyptian cat
282     0.2236193   tiger cat
287     0.0523816   lynx, catamount
286     0.0035395   cougar, puma, catamount, mountain lion, painter, panther, Felis concolor
728     0.0030023   plastic bag
280     0.0007048   grey fox, gray fox, Urocyon cinereoargenteus
289     0.0005830   snow leopard, ounce, Panthera uncia
278     0.0003320   kit fox, Vulpes macrotis
761     0.0002517   remote control, remote
```

So, overall -- very cool! :-)




## The Inference Pipeline Verification Script

```
cd /opt/intel/openvino/deployment_tools/demo/
sudo ./demo_security_barrier_camera.sh
```


Looking over the screen output from running this file is instructive.  For example, you will see that
Intel's Model Downloader was used multiple times to download a few models:
* license-plate-recognition-barrier-0001
* vehicle-attributes-recognition-barrier-0039
* vehicle-license-plate-detection-barrier-0106

These are downloaded using commands like the following:
```
python3.7 /opt/intel//openvino_2019.3.376/deployment_tools/open_model_zoo/tools/downloader/downloader.py --name vehicle-license-plate-detection-barrier-0106 --output_dir /Users/kevinurban/openvino_models/ir --cache_dir /Users/kevinurban/openvino_models/cache
```

You will find the IR files for these models in:
```
~/openvino_models/ir/intel/
```

You'll also get a note where the build files are located:
```
Build files have been written to: /Users/<userName>/inference_engine_demos_build
```

How are these models used?  From the website:
> "First, an object is identified as a vehicle. This identification is used as input to the next model, which identifies 
> specific vehicle attributes, including the license plate. Finally, the attributes identified as the license plate are used 
> as input to the third model, which recognizes specific characters in the license plate."

From the README:
```
The demo illustrates using the Inference Engine with pre-trained models to perform vehicle detection, vehicle 
attributes and license-plate recognition tasks. 

As the sample produces visual output, it should be run in GUI mode.  

The demo script does the following:

- Builds the Inference Engine security barrier camera sample (inference_engine\samples\security_barrier_camera_sample)
- Runs the sample with the car_1.bmp located in the demo folder

The sample application displays the resulting frame with detections rendered as bounding boxes and text.
```

In the screen output when running this demo, you can see how to string together a few 
models:

```
./security_barrier_camera_demo -d CPU -d_va CPU -d_lpr CPU \
	-i /opt/intel/openvino_2019.3.376/deployment_tools/demo/car_1.bmp \
	-m /Users/<userName>/openvino_models/ir/intel/vehicle-license-plate-detection-barrier-0106/FP16/vehicle-license-plate-detection-barrier-0106.xml \
	-m_lpr /Users/<userName>/openvino_models/ir/intel/license-plate-recognition-barrier-0001/FP16/license-plate-recognition-barrier-0001.xml \
	-m_va /Users/<userName>/openvino_models/ir/intel/vehicle-attributes-recognition-barrier-0039/FP16/vehicle-attributes-recognition-barrier-0039.xml
```

And, of course, looking over the bash script itself is super helpful in learning
how to issue commands yourself.

```
#!/usr/bin/env bash

# Copyright (C) 2018-2019 Intel Corporation
# SPDX-License-Identifier: Apache-2.0

ROOT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

. "$ROOT_DIR/utils.sh"

usage() {
    echo "Security barrier camera demo that showcases three models coming with the product"
    echo "-d name     specify the target device to infer on; CPU, GPU, FPGA, HDDL or MYRIAD are acceptable. Sample will look for a suitable plugin for device specified"
    echo "-help            print help message"
    exit 1
}

trap 'error ${LINENO}' ERR

target="CPU"

# parse command line options
while [[ $# -gt 0 ]]
do
key="$1"

case $key in
    -h | -help | --help)
    usage
    ;;
    -d)
    target="$2"
    echo target = "${target}"
    shift
    ;;
    -sample-options)
    sampleoptions="$2 $3 $4 $5 $6"
    echo sample-options = "${sampleoptions}"
    shift
    ;;
    *)
    # unknown option
    ;;
esac
shift
done


target_image_path="$ROOT_DIR/car_1.bmp"

run_again="Then run the script again\n\n"
dashes="\n\n###################################################\n\n"

if [[ -f /etc/centos-release ]]; then
    DISTRO="centos"
elif [[ -f /etc/lsb-release ]]; then
    DISTRO="ubuntu"
elif [[ "$OSTYPE" == "darwin"* ]]; then
    DISTRO="macos"
fi

if [[ $DISTRO == "centos" ]]; then
    sudo -E yum install -y centos-release-scl epel-release
    sudo -E yum install -y gcc gcc-c++ make glibc-static glibc-devel libstdc++-static libstdc++-devel libstdc++ libgcc \
                           glibc-static.i686 glibc-devel.i686 libstdc++-static.i686 libstdc++.i686 libgcc.i686 cmake

    sudo -E rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm || true
    sudo -E yum install -y epel-release
    sudo -E yum install -y cmake ffmpeg gstreamer1 gstreamer1-plugins-base libusbx-devel

    # check installed Python version
    if command -v python3.5 >/dev/null 2>&1; then
        python_binary=python3.5
        pip_binary=pip3.5
    fi
    if command -v python3.6 >/dev/null 2>&1; then
        python_binary=python3.6
        pip_binary=pip3.6
    fi
    if [ -z "$python_binary" ]; then
        sudo -E yum install -y rh-python36 || true
        . scl_source enable rh-python36
        python_binary=python3.6
        pip_binary=pip3.6
    fi
elif [[ $DISTRO == "ubuntu" ]]; then
    sudo -E apt update
    print_and_run sudo -E apt -y install build-essential python3-pip virtualenv cmake libcairo2-dev libpango1.0-dev libglib2.0-dev libgtk2.0-dev libswscale-dev libavcodec-dev libavformat-dev libgstreamer1.0-0 gstreamer1.0-plugins-base
    python_binary=python3
    pip_binary=pip3

    system_ver=`cat /etc/lsb-release | grep -i "DISTRIB_RELEASE" | cut -d "=" -f2`
    if [ $system_ver = "18.04" ]; then
        sudo -E apt-get install -y libpng-dev
    else
        sudo -E apt-get install -y libpng12-dev
    fi
elif [[ "$OSTYPE" == "darwin"* ]]; then
    # check installed Python version
    if command -v python3.7 >/dev/null 2>&1; then
        python_binary=python3.7
        pip_binary=pip3.7
    elif command -v python3.6 >/dev/null 2>&1; then
        python_binary=python3.6
        pip_binary=pip3.6
    elif command -v python3.5 >/dev/null 2>&1; then
        python_binary=python3.5
        pip_binary=pip3.5
    else
        python_binary=python3
        pip_binary=pip3
    fi
fi

if ! command -v $python_binary &>/dev/null; then
    printf "\n\nPython 3.5 (x64) or higher is not installed. It is required to run Model Optimizer, please install it. ${run_again}"
    exit 1
fi

if [[ $DISTRO == "macos" ]]; then
    $pip_binary install -r $ROOT_DIR/../open_model_zoo/tools/downloader/requirements.in
else
    sudo -E $pip_binary install -r $ROOT_DIR/../open_model_zoo/tools/downloader/requirements.in
fi

if [ -e "$ROOT_DIR/../../bin/setupvars.sh" ]; then
    setupvars_path="$ROOT_DIR/../../bin/setupvars.sh"
else
    printf "Error: setupvars.sh is not found\n"
fi
if ! . $setupvars_path ; then
    printf "Unable to run ./setupvars.sh. Please check its presence. ${run_again}"
    exit 1
fi

# Step 1. Downloading Intel models
printf "${dashes}"
printf "Downloading Intel models\n\n"


target_precision="FP16"

printf "target_precision = ${target_precision}\n"

downloader_dir="${INTEL_OPENVINO_DIR}/deployment_tools/open_model_zoo/tools/downloader"

downloader_path="$downloader_dir/downloader.py"
models_path="$HOME/openvino_models/ir"
models_cache="$HOME/openvino_models/cache"

declare -a model_args

while read -r model_opt model_name; do
    model_subdir=$("$python_binary" "$downloader_dir/info_dumper.py" --name "$model_name" |
        "$python_binary" -c 'import sys, json; print(json.load(sys.stdin)[0]["subdirectory"])')

    model_path="$models_path/$model_subdir/$target_precision/$model_name"

    print_and_run "$python_binary" "$downloader_path" --name "$model_name" --output_dir "$models_path" --cache_dir "$models_cache"

    model_args+=("$model_opt" "${model_path}.xml")
done < "$ROOT_DIR/demo_security_barrier_camera.conf"

# Step 2. Build samples
printf "${dashes}"
printf "Build Inference Engine demos\n\n"

demos_path="${INTEL_OPENVINO_DIR}/deployment_tools/inference_engine/demos"

if ! command -v cmake &>/dev/null; then
    printf "\n\nCMAKE is not installed. It is required to build Inference Engine demos. Please install it. ${run_again}"
    exit 1
fi

OS_PATH=$(uname -m)
NUM_THREADS="-j2"

if [ $OS_PATH == "x86_64" ]; then
  OS_PATH="intel64"
  NUM_THREADS="-j8"
fi

build_dir="$HOME/inference_engine_demos_build"
if [ -e $build_dir/CMakeCache.txt ]; then
	rm -rf $build_dir/CMakeCache.txt
fi
mkdir -p $build_dir
cd $build_dir
cmake -DCMAKE_BUILD_TYPE=Release $demos_path
make $NUM_THREADS security_barrier_camera_demo

# Step 3. Run samples
printf "${dashes}"
printf "Run Inference Engine security_barrier_camera demo\n\n"

binaries_dir="${build_dir}/${OS_PATH}/Release"
cd $binaries_dir

print_and_run ./security_barrier_camera_demo -d "$target" -d_va "$target" -d_lpr "$target" -i "$target_image_path" "${model_args[@]}" ${sampleoptions}

printf "${dashes}"
printf "Demo completed successfully.\n\n"
```

### Messing Around
I was able to successfully mess around with the image classification demo, so I figured
I'd do the same here.  

I downloaded some JPEGs of cars with visible license plates...and the script ran without error, however
it did not detect a license plate (so did not read a license plate either).

I even tried converting the JPEGs to BitMap using ImageMagick (`convert img.jpg img.bmp`), but 
license plates were still not detected...


