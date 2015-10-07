
MDSrv is a simple server that enables remote access to coordinate trajectories from molecular dynamics simulations. It can be used together with the NGL Viewer (http://github.com/arose/ngl) to interactively view trajectories of molecular complexes in a web-browser, either within a local network or from anywhere over the internet.

Formats supported are:
* xtc/trr
* nc, netcdf
* dcd

Thanks to code from MDAnalysis (http://www.mdanalysis.org/) there is random access to xtc/trr trajectory files via indexing and seeking capabilities added to the libxdrfile2 library.


Table of contents
=================

* [Installation](#installation)
* [Running](#running)
* [API](#api)
* [Deployment](#deployment)
* [License](#license)


Installation
============

Flask
-----

The server is based on *Flask*, which can be installed through *pip*:

    sudo pip install Flask



Trajectory reading
------------------

For the efficient access of trajectory files some libraries need to be installed. First, make sure you have the python development files installed, e.g.

    sudo apt-get install python-dev python-numpy


Then install the NetCDF libraries

    sudo apt-get install libhdf5-serial-dev libnetcdf-dev
    sudo pip install netCDF4


In case you get "ValueError: did not find HDF5 headers" try:

    sudo su
    find / -name "libhdf5*.so*"
    # use what the above 'find' suggests to set 'HDF5_DIR'
    export HDF5_DIR=/usr/lib/i386-linux-gnu/hdf5/serial/
    pip install netCDF4


Libraries for reading xtc/trr and dcd files are included and can be installed with

    sh install.sh



Configuration file
------------------

Copy/rename the sample `app.cfg` file. It allows e.g. setting data directories that will be accessible through the web server and to define access restrictions.

    cp app.cfg.sample app.cfg



Running
=======

A local development server can be started with the python script

    python serve.py


which will use the `app.cfg` configuration file or with

    python serve.py my_conf.cfg


to use the `my_conf.cfg` configuration file.


API
===

File content
------------

To retrieve a file from `data_dir` `<root>` with file path `<filename>` call:

    /file/<root>/<filename>


Directory content description
-----------------------------

To get a JSON description of available `data_dir` directories call:

    /dir/


To get a JSON description of the directory content in `data_dir` `<root>` call:

    /dir/<root>/


To get a JSON description of the directory content in `data_dir` `<root>` at path `<path>` call:

    /dir/<root>/<path>


The JSON description is a list of file or sub-directory entries:

    [
        {
            "name": "name of sub-directory",
            "path": "path relative to `<root>`",
            "dir": "`true` if entry is a directory",
            "restricted": "`true` if access is restricted"
        },
        {
            "name": "name of the file",
            "path": "path relative to `<root>`",
            "size": "file size in bytes"
        },
        {
            ...
        }
    ]



Frame count
-----------

To get the number of frames the trajectory in `data_dir` `<root>` with file path `<filename>` has call:

    /traj/numframes/<root>/<filename>


Frame coordinates
-----------------

To get the coordinates of frame number `<frame>` of the trajectory in `data_dir` `<root>` with file path `<filename>` call:

    /traj/frame/<frame>/<root>/<filename>


The set of atoms for which coordinates should be returned can be restricted by `POST`ing an `atomIndices` parameter with the following format. A list of index ranges is defined by pairs of integers separated by semi-colons (`;`) where the two integers within a pair are separated by a comma (`,`). To select atoms with indices (starting at zero) 5 to 10 and 22 to 30 send:

    5,10;22,30


The coordinate frame is returned in binary format and also contains the frame number, the simulation time (when available) and the box vectors.

| Offset | Size |  Type | Description                  |
| -----: | ---: | ----: | :--------------------------- |
|      0 |    4 |   int | frame number                 |
|      4 |    4 | float | simulation time              |
|      8 |    4 | float | X coordinate of box vector A |
|     12 |    4 | float | Y                            |
|     16 |    4 | float | Z                            |
|     20 |    4 | float | X coordinate of box vector B |
|     24 |    4 | float | Y                            |
|     28 |    4 | float | Z                            |
|     32 |    4 | float | X coordinate of box vector C |
|     36 |    4 | float | Y                            |
|     40 |    4 | float | Z                            |
|     44 |    4 | float | X coordinate of first atom   |
|     48 |    4 | float | Y                            |
|     52 |    4 | float | Z                            |
|    ... |  ... |   ... | ...                          |


Deployment
==========

The Apache Webserver can used to run the server via `mod_wsgi`. First make sure you have everything required installed:

    sudo apt-get install git apache2 libapache2-mod-wsgi


Then you need to create a wsgi configuration file to be referenced in the Apache configuration. There is an example named `ngl.wsgi.sample` in the root directory of this package. Also, a snippet showing how the configuration for Apache should look like can be found in the `apache.config.sample` file.

Finally, to restart apache issue

    sudo /etc/init.d/apache2 restart



License
=======

Generally GPL 2, see the LICENSE file for details.