---
layout: docs
title:  3.0 Developing with an IBM Streams install
description: To get started with the Python Application API, you'll use the example of reading data from a temperature sensor and printing the output to the screen.
weight:  30
published: true
tag: py16
prev:
  file: python-appapi-devguide-2
  title: 2.0 Developing with the IBM Streaming Analytics service
next:
  file: python-appapi-devguide-4
  title: 4.0 Common Streams operations
---

Streaming analytics applications are intended to run indefinitely because they meet the need for real-time data processing. (Unlike applications created for the Apache Hadoop framework, which are intended to terminate when a batch of data is successfully processed.) For example, consider a company whose product scans temperature sensors across the world to determine weather patterns and trends. Because there is always a temperature, there is a perpetual need to process data. The application that processes the data must be able to run for an indefinite amount of time.

The application must also be scalable. If the number of temperature sensors doubles, the application must double the speed at which it processes data to ensure that analysis is available in a timely manner. With the Python Application API, you can develop a streaming analytics application natively in Python.

To get started with the Python Application API, you'll use the example of reading data from a temperature sensor and printing the output to the screen.

## 3.1 Setting up your environment
Before you can create your first Python application with the Python Application API and a local version of IBM Streams, you must complete the following setup tasks:

1. Install version 4.0.1 (or later) of IBM Streams or IBM Streams Quick Start Edition:

    * [IBM Streams Version 4.2.0 installation documentation](http://www.ibm.com/support/knowledgecenter/SSCRJU_4.2.0/com.ibm.streams.install.doc/doc/installstreams-container.html)

    * [IBM Streams Quick Start Edition Version 4.2.0 installation documentation](http://www.ibm.com/support/knowledgecenter/SSCRJU_4.2.0/com.ibm.streams.qse.doc/doc/installtrial-container.html)

1. Ensure that you configure the IBM Streams product environment variable by entering the following command:

        source product-installation-root-directory/4.n.n.n/bin/streamsprofile.sh

    **Tip:** Add the source command to your `home-directory/.bashrc` shell initialization file. Otherwise, you must enter the command every time you start IBM Streams. For example, if the product is installed in the `/home/streamsadmin/InfoSphere_Streams/4.2.0.0` directory, add the following line to your `.bashrc` file:

        source /home/streamsadmin/InfoSphere_Streams/4.2.0.0/bin/streamsprofile.sh


1. If you are using IBM Streams 4.2 or later, you can skip this step because the Python Application API is included at* `$STREAMS_INSTALL/toolkits/com.ibm.streamsx.topology`. <br><br>If you are using an earlier version of IBM Streams, you must:
    1. Download the latest version of the IBM Streams Topology toolkit from the IBMStreams organization on GitHub from the streamsx.topology [Releases page](https://github.com/Ibmstreams/streamsx.topology/releases).
    1. After the toolkit downloads, extract it to your file system.
    

1. Ensure that you have installed a supported version of Python: 

   * Python 2.7 and Python 3.5 are supported.
   
      **Important:** Python 3.5 is required to build application bundles with the Python Application API that can be submitted to your IBM Streaming Analytics service.

   You can install a supported version of Python from:
   * *Recommended* Anaconda [https://www.continuum.io/downloads](https://www.continuum.io/downloads)

   * CPython [https://www.python.org](https://www.python.org)




1. Include the fully qualified path of the `com.ibm.streamsx.topology/opt/python/packages` directory in the PYTHONPATH environment variable. For example:

        export PYTHONPATH=/home/myuser/download/com.ibm.streamsx.topology/opt/python/packages:$PYTHONPATH


## 3.2 Creating a topology object
The first component of your application is a `Topology` object.

Include the following code in the temperature_sensor.py file (the main module):

~~~~~~
from streamsx.topology.topology import Topology
topo = Topology("temperature_sensor")
~~~~~~

A streaming analytics application is a directed flow graph that specifies how data is generated and processed. The `Topology` object contains information about the structure of the directed flow graph.


## 3.3 Defining a data source
The `Topology` object also includes functions that enable you to define your data sources. In this application, the data source is the temperature sensor.

In this example, simulate temperature sensor readings by defining a Python generator function that returns an iterator of random numbers.

**Important:** Callable inputs to functions, such as the definition of the `readings()` function, cannot be defined in the `'_main_'` module. The inputs must be defined in a separate module.

Include the following code in the temperature_sensor_functions.py file:

~~~~~~
import random
def readings():
    while True:
        yield random.gauss(0.0, 1.0)
~~~~~~

The `Topology.source()` function takes as input a zero-argument callable object, such as a function or an instance of a callable class, that returns an iterable of tuples. In this example, the input to `source` is the `readings()` function.  The `source` function calls the `readings()` function, which returns a generator object.  The `source` function gets the iterator from the generator object and repeatedly calls the `next()` function on the iterator to get the next tuple, which returns a new random temperature reading each time.

In this example, data is obtained by calling the `random.gauss()` function. However, you can use a live data source instead of the `random.gauss()` function.


## 3.4 Creating a Stream
The `Topology.source()` function produces a `Stream` object, which is a potentially infinite flow of tuples in an application. Because a streaming analytics application can run indefinitely, there is no upper limit to the number of tuples that can flow over a `Stream`.  

Tuples flow over a `Stream` one at a time and are processed by subsequent data **operations**. Operations are discussed in more detail in the [Common Streams operations](../python-appapi-devguide-4/) section of this guide.

A tuple can be any Python object that is serializable by using the pickle module.

Include the following code in the temperature_sensor.py file:

~~~~~~
source = topo.source(temperature_sensor_functions.readings)
~~~~~~


## 3.5 Printing to output
After obtaining the data, you print it to standard output using the `sink` operation, which terminates the stream.

Include the following code in the temperature_sensor.py file:

~~~~~~
source.sink(print)
~~~~~~

The `Stream.sink()` operation takes as input a callable object that takes a single tuple as an argument and returns no value. The callable object is invoked with each tuple. In this example, the `sink` operation calls the built-in `print()` function with the tuple as its argument.  

**Tip:** The `print()` function is useful, but if your application needs to output to a file, you need to implement a custom sink operator.


## 3.6 Submitting the application
After you define the application, you can submit it by using `streamsx.topology.context` module. When you submit the application, use the `submit()` function from the `streamsx.topology.context` module and pass the context type and the topology graph object as parameters to the function.

Include the following code in the temperature_sensor.py file:

~~~~~~
streamsx.topology.context.submit("STANDALONE", topo)
~~~~~~

**Remember:** You can run your application in the following ways:

* As a **Streams distributed application** (DISTRIBUTED). When running in this mode, the application produced will be deployed automatically on your IBM Streams instance.
* As a **Streams Application Bundle file** (BUNDLE). When running in this mode, the application produces a SAB file that you can then deploy on your IBM Streams instance by using the `streamtool submitjob` command or by using the application console.
* As a **stand-alone application** (STANDALONE).  When running in this mode, the application produces a Streams Application Bundle file (SAB file), but rather than submitting the SAB file to an instance, the bundle is executed. The bundle runs within a single process and can be terminated with Ctrl-C interrupts.
* As a **Streaming Analytics service running on IBM Bluemix cloud platform** (STREAMING_ANALYTICS_SERVICE). In this mode,  the application will run in the cloud in a Streaming Analytics service.


## 3.7 The complete application
Your complete application should look like this:

The following code should be in the temperature_sensor.py file:

~~~~~~
from streamsx.topology.topology import Topology
import streamsx.topology.context
import temperature_sensor_functions

def main():
    topo = Topology("temperature_sensor")
    source = topo.source(temperature_sensor_functions.readings)
    source.sink(print)
    streamsx.topology.context.submit("STANDALONE", topo)

if __name__ == '__main__':
    main()
~~~~~~


The following code should be in the temperature_sensor_functions.py file:

~~~~~~
import random

def readings():
    while True:
        yield random.gauss(0.0, 1.0)

~~~~~~

## 3.8 Running the application
To run the sample application, enter the following command:

~~~~~~
python3 temperature_sensor.py
~~~~~~

Enter `Ctrl-C` to stop the application.

## 3.9 Sample output
The contents of your output should look something like this:

~~~~~~
...
1.6191338426594375
-0.3088492294198733
0.43973191574979087
-1.0249371132740133
-0.3151212021333815
-0.6787283449628287
-0.11907886745291935
-0.24096558784475972
...
~~~~~~
