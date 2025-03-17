# sigma_compare - a float image comparison tool
sigma_compare is described in [Gary Demos "A Quality Metric For High Dynamic Range", SMPTE 2014 Annual Technical Conference and Exhibition](https://github.com/michaeldsmith/sigma_compare/blob/main/resources/references/QualityMetricHDR.pdf)

## Installation Prerequisites ##

### Required ###

__CMake__

CMake can be downloaded directly from www.cmake.org or using using package managers using the commands below:

* Ubuntu

        sudo apt-get install cmake

* Redhat

        yum install cmake

* OS X

        brew install cmake
  
__OpenEXR__

OpenEXR can be downloaded directly and built from source available from www.openexr.org or using using package managers using the commands below:

* Ubuntu

        sudo apt-get install libopenexr-dev

* Redhat

        yum install OpenEXR-devel

* OS X
  
        brew install openexr                

## Compilation ##

from the root source directory:

        cd build
        cmake ..
        make

  to run the optional unit tests:

        ctest

## Example usage: ##

        ./sigma-compare ../resources/test/exr/AllHalfValues.NONE.exr ../resources/test/exr/AllHalfValues.DWAA_45.exr
        
        
