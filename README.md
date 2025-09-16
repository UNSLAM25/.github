**UNSLAM** is a real-time *hybrid* Visual SLAM system, which runs partly on a cell phone and partly on a server PC. It was developed at the University of La Matanza (UNLAM), and is based on the **stella_vslam** Visual SLAM system, which in turn is based on **OpenVSLAM**, which is based on **ORB-SLAM2**.

Visual SLAM systems are heavy to run in real time on most cell phones. The hybrid approach divides the computation between the cell phone and a PC connected by WIFI.

This GitHub organization consists of several repositories with the complete code and documentation to compile and install UNSLAM, and even to modify it. For your convenience, a Docker image with the ready-to-use system is provided.

All software is provided as is and without responsibility for its use. The development of UNLAM is provided under a free license, third-party parts are governed by their own licenses.

The following sections describe the system, address different use cases, and describe the UNSLAM repositories.


# UNSLAM System
The UNSLAM system consists of a server conveniently developed in Python, easy to understand and modify. This server fulfills two main functions:
- web server accessed with the cell phone's browser
- executes the core of the Visual SLAM system

The cell phone part runs on the web page served by the UNSLAM system, so it does not require any installation on the mobile device. The server, via Python bindings, uses **stella_vslam** system slightly modified to work in hybrid mode.

The parts of the system are:
- vslam backend
  - Python application that implements a web server, a UDP messaging system, and the connection with the Visual SLAM system
  - runs in the container on PC
- stella_vslam (modified)
  - hybrid Visual SLAM system, based on stella_vslam
  - this system is installed in the container
  - it is executed by vslam_backend which uses it through Python bindings
- web preprocessor
  - this web application served by vslam_backend runs in the cell phone browser
  - opens the camera, preprocesses the image and sends an image descriptor to the server via WIFI
- wasm preprocessor
  - central component of web preprocessor
written in C++ and compiled with emscripten, incorporates OpenCV to process the image with high performance


# Ready-to-use Container
You can download the [Docker container image for the UNSLAM system from this link](https://drive.google.com/file/d/1bbYczytUE1hg_rWUzoSy8ynT70jD-tuJ/view?usp=drive_link) (2.78GB, Aug 2025).


*Instructions...*


# UNSLAM Repositories
Brief descriptions of this organization repositories:
- vslam-backend
  - Python server
complete system, includes files produced or obtained from other repositories, such as the wasm preprocessor and orb_vocab.fbow
  - requires stellavslam.cpython-310-x86_64-linux-gnu.so (Python bindings), libstella_vslam.so, and libpangolin_viewer.so installed in the system
- stella_vslam
  - modified stella_vslam fork, incorporates the image descriptor unpacker
  - note: compiling and installing a Visual SLAM system is often a long and complex process, which is why UNSLAM is provided in a ready-to-use container
- StellaVSLAM-Python-bindings
  - code to generate the bindings module that connects the Python backend with the stella_vslam.so library
  - the functions of stella_vslam accessible from Python can be seen in stellavslam_bindings.cpp
- wasm-preprocessor
  - code to generate the WASM module that preprocesses the image in the browser and produces the image descriptor
  - the produced WASM module is provided for convenience, so there is no need to regenerate it unless you want to test modifications
- web-preprocessor
  - website that opens the camera, captures images, preprocesses them with wasm-preprocessor, and sends them to the server
  
  The complete project has only these compilable parts: stella_vslam, wasm-preprocessor, StellaVSLAM-Python-bindings. All are provided already compiled for convenience within a ready-to-use Docker image.

# Modifying the code
The container image is a minimal Linux image with stella_vslam already installed. It contains vslam-backend in the /stella_vslam/vslam-backend/ folder.

Through parameters, the docker command line allows changing files in the image when running the container. These changes do not persist, they are not permanent, they last for the duration of the container. With this mechanism, changes to completeSystem.py and any other system file can be tested.

For example, the command

    sudo docker run -it --rm --privileged --name unslam-cont -e DISPLAY=$DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix:ro -v myFolder/completeSystem.py:/stella_vslam/vslam-backend/completeSystem.py unslam

replaces completeSystem.py in the image with a local version.

# Local Installation
Instead of using the container, the complete application can be run locally on Linux, ideal for studying it in detail and making deep changes. Here are the general steps; detailed instructions are presented in each repository:
- clone all UNSLAM repositories locally
- install stella_vslam from the UNSLAM repository
  - this is the most complex and demanding step
  - the result is the libstella_vslam.so and libpangolin_viewer.so libraries
- compile the Python bindings in the StellaVSLAM-Python-bindings repository
  - a library with a name like this is produced:
  - stellavslam.cpython-310-x86_64-linux-gnu.so
  - copy that file to the vslam-backend folder
- compile wasm-preprocessor
  - move the produced files to the vslam-backend/web/wasm folder
- the system starts by running completeSystem.py in vslam-backend
  - in the console, it will show the URL to navigate to with the cell phone
  - according to the instructions in web-preprocessor, configure that URL in the cell phone browser to treat it as secure, as the server does not have a certificate or use https
  - navigate to that URL and start slowly mapping the environment.