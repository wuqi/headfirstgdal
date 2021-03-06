.. highlight:: rst

################
静态编译
################

分享一下完全的静态编译过程，栅格数据格式加入 ``hdf4`` 、 ``hdf5`` 、 ``netcdf`` 、 ``openJPEG`` 、 ``webp`` 静态编译，矢量数据格式加入编译 ``libexpat`` 和 ``libcurl`` ``spatialite``， ``gdal`` 是最新 ``svn`` 中编译的，其他库都是最新 ``release`` 版本。

这样编译出来的 ``gdal`` 只有  ``gdal`` 本身的 ``dll`` 和 ``lib`` ，不需要添加其他的 ``dll`` ，比较方便一些，且比加入其他的 ``dll`` 要小，有问题请直接在最下评论。

动态编译请参考 `GDAL源码剖析（二）之编译说明 <http://blog.csdn.net/liminlu0314/article/details/6937194>`_ 

******************
编译环境
******************
编译所需的软件： 

* `cmake <http://www.cmake.org/>`_  根据netcdf的要求是2.8.10以上版本，用于编译 ``expat hdf4 hdf5 openJPEG``
* ``vs2010`` ，一般都使用命令行编译
* `nsis <http://nsis.sourceforge.net/Download>`_  2.46版， 用于生成 ``hdf4 hdf5`` 安装文件
  
推荐编译顺序： ``libexpat  libcurl hdf4 hdf5  netcdf4 geos proj.4 libsqlite3 libwebp openJPEG gdal``

编译顺序除了注意事项中所描述的，一般可以互换。

``netcdf hdf4 hdf5 spatialite``  这几个库编译比较复杂，请注意编译条件的修改。

虽然 ``cmake`` 可以使用 ``GUI`` ，推荐都使用命令行编译，命令行开头我将以 ``>`` 标识出来,本文都是在 ``Windows`` 下编译， ``Linux`` 下没有尝试。

.. warning::

    * libexpat库静态编译必须设置 ``XML_STATIC`` 宏
    * openJPEG库静态编译后需要设置 ``OPJ_STATIC`` 宏
    * 如果需要编译netcdf，libcurl库必须在netcdf前编译
    * netcdf与hdf4如果都需要的话，编译hdf4时必须关闭netcdf设置
    * geos库最好不用svn的版本
    * 编译都假设起始目录为代码源目录
    * 不要使用 ``nsis3.0a1`` 版本，无法生成安装文件
    * hdf5是只读驱动,但是netcdf也算hdf5格式,netcdf是读写驱动

.. attention::

    我编译netcdf时没有加入hdf4设置，加入后无法通过编译，如果有人解决，请在评论中留言

******************
libexpat
******************

编译前准备
================
下载源码后，修改 ``expat-2.1.0/lib/expat.h`` 文件,在文件首加上：

.. code-block:: c++

    #define XML_STATIC

这一步也可以在最后的生成include中修改，只要能链接就可以。

打开vs2010命令行，依次输入：

.. code-block:: bash

    mkdir build
    cd build 
    cmake -G   "Visual Studio 10" -DBUILD_shared=OFF 
          -DCMAKE_INSTALL_PREFIX=E:\BUILD\expat ..
    cmake --build . --config Release --target INSTALL

生成文件将在 ``E:/BUILD/expat`` 路径下

******************   
libcurl
******************

libcurl更新比较快,1-2月更新一个版本,请选择最新的版本,可选 ``openssl`` 和 ``zlib`` 等库,最简单的编译方式如下:

.. code-block:: bash

    cd winbuild
    nmake /f Makefile.vc mode=static

生成文件将在 ``builds`` 文件夹下.

******************   
hdf4
******************

hdf4最终版本是 ``4.2.13`` ,但是自 ``4.2.12`` 版本起,编译方式变化很大,请阅读相应部分

``4.2.11`` 以及之前版本
===========================

编辑 ``/config/cmake/cacheinit.cmake`` 文件：

设置静态库,7行::

    SET (BUILD_SHARED_LIBS OFF CACHE BOOL "Build Shared Libraries" FORCE)
     
关闭 ``fortan`` 编译,15行::

    SET (HDF4_BUILD_FORTRAN OFF CACHE BOOL "Build FORTRAN support" FORCE)

关闭 ``netcdf`` 支持，如果在 ``gdal`` 中不同时使用 ``netcdf`` ，可以不修改，23行::

    SET (HDF4_ENABLE_NETCDF OFF CACHE BOOL "Build HDF4 versions of NetCDF-3 APIS" FORCE)

编译 ``zlib`` 库和 ``szip`` 库,使用svn中的代码(需要联网)，49行::

    SET (HDF4_ALLOW_EXTERNAL_SUPPORT "SVN" CACHE STRING  \
        "Allow External Library Building" FORCE)

在 ``vs2010`` 命令行工具中，依次输入：

.. code-block:: bash

    mkdir build
    cd build 
    cmake -G "Visual Studio 10"  -C ..\config\cmake\cacheinit.cmake ..
    cmake --build . --config Release
    copy /B .\bin\Release\libjpeg.lib .\bin\libjpeg.lib
    copy /B .\bin\Release\libzlib.lib .\bin\libzlib.lib
    copy /B .\bin\Release\libszip.lib .\bin\libszip.lib
    cmake --build . --config Release
    cpack -C Release CPackConfig.cmake
    HDF-4.2.9-win32.exe

.. attention::

    * ``cmake --build . --config Release`` 运行了两次，因为hdf库的cmake写的有些问题，需要把 ``libjpeg`` 等库先拷贝到上一层才能完成全部的编译。
    * 如果需要编译64位的话，第三行需要修改为： ``cmake -G "Visual Studio 10 Win64"  -C ../config/cmake/cacheinit.cmake ..``

``4.2.12`` 之后版本
===========================

.. attention::
	
	* 请注意, ``4.2.12`` 之后版本不需要下载原始sourcecode,下载 CMake版本代码直接编译!!
	
参考cmake build页面: `cmakebuild <https://support.hdfgroup.org/release4/cmakebuild.html>`_

下载页面中的: `Contains files to build HDF4 with CMake on Windows <https://support.hdfgroup.org/ftp/HDF/HDF_Current/src/CMake-hdf-4.2.13.zip>`_ ,下载后的压缩包中包含了szip和zlib等所需库,不需要额外再下载.

修改其中的  ``HDF4options.cmake`` 文件,文件末尾添加  ``set(ADD_BUILD_OPTIONS "${ADD_BUILD_OPTIONS} -DHDF4_ENABLE_NETCDF:BOOL=OFF")`` ,防止与 ``netcdf`` 库冲突

下载完成后,根据Visual Studio版本,运行相应的 ``build-VS20xx-32.bat`` 或者 ``build-VS20xx-64.bat`` 文件,会自动新建build文件夹,最终在 ``build`` 文件夹下生成zip文件.

老版本或者新版本的Visual Studio可直接编辑 ``HDF4config.cmake`` 文件,仿照其他bat文件写脚本.

******************   
hdf5
******************

``hdf5`` 与 ``hdf4`` 类似,新版本直接使用脚本文件调用 ``CMake``

``5.1.8.17`` 之前版本
===========================

编辑 ``/config/cmake/cacheinit.cmake`` 文件：

编译静态库，7行::

    SET (BUILD_SHARED_LIBS OFF CACHE BOOL "Build Shared Libraries" FORCE)
     
关闭 ``fortran`` 库编译，17行::

    SET (HDF5_BUILD_FORTRAN OFF CACHE BOOL "Build FORTRAN support" FORCE)

``zlib`` 和 ``szip`` 库支持，63行::

    SET (HDF5_ALLOW_EXTERNAL_SUPPORT "SVN" CACHE STRING \
        "Allow External Library Building" FORCE)

.. attention::

    * 网络不好的情况下,可以在 `HDF5官网 <http://www.hdfgroup.org/HDF5/release/cmakebuild.html>`_ 下载szip和zlib库,放在hdf5文件夹下,然后修改第63行左右为:

            SET (HDF5_ALLOW_EXTERNAL_SUPPORT "TGZ" CACHE STRING "Allow External Library Building" FORCE)

在 ``vs2010`` 命令行工具中，依次输入：

.. code-block:: bash

    mkdir build
    cd build 
    cmake -G "Visual Studio 10"  -C ../config/cmake/cacheinit.cmake ..
    cmake --build . --config Release
    copy /B .\bin\Release\libzlib.lib .\bin\libzlib.lib
    copy /B .\bin\Release\libszip.lib .\bin\libszip.lib
    cmake --build . --config Release
    cpack -C Release CPackConfig.cmake

.. attention::

    * 注意 ``cmake --build . --config Release`` 运行了两次，因为hdf库的cmake写的有些问题，需要把 ``libzlib`` 等库先拷贝到上一层才能完成全部的编译。
    * 如果需要编译64位的话，第三行需要修改为： ``cmake -G "Visual Studio 10 Win64"  -C ../config/cmake/cacheinit.cmake ..``

.. warning::

    * ``hdf5.1.8.13`` 版本静态编译有问题,没有特殊需求不要使用,若使用,请删除或注释 ``/hdf5-1.8.13/src/H5.c`` 第841行以下的部分

``5.1.8.17`` 之后的老版本
===========================

与 ``HDF4`` 类似,参考cmake build页面: `cmakebuild <https://support.hdfgroup.org/HDF5/release/cmakebuild518.html>`_ ,下载后的压缩包中包含了szip和zlib等所需库,不需要额外再下载.

下载页面中的: `Contains files to build HDF5 with CMake on Windows <https://support.hdfgroup.org/ftp/HDF5/current18/src/CMake-hdf5-1.8.19.zip>`_

下载完成后,根据Visual Studio版本,运行相应的 ``build-VS20xx-32.bat`` 或者 ``build-VS20xx-64.bat`` 文件.

老版本或者新版本的Visual Studio可直接编辑 ``HDF5config.cmake`` 文件,仿照其他bat文件写脚本.

``5.1.10`` 版本
===========================

.. attention::

	``5.1.10`` 是新的大版本,可以读老文件,写文件与老版本不兼容.

与 ``HDF4`` 类似,参考cmake build页面: `cmakebuild <https://support.hdfgroup.org/HDF5/release/cmakebuild.html>`_ ,下载后的压缩包中包含了szip和zlib等所需库,不需要额外再下载.

下载页面中的: `Contains files to build HDF5 with CMake on Windows <https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.1/src/CMake-hdf5-1.10.1.zip>`_

下载完成后,根据Visual Studio版本,运行相应的 ``build-VS20xx-32.bat`` 或者 ``build-VS20xx-64.bat`` 文件.

老版本或者新版本的Visual Studio可直接编辑 ``HDF5config.cmake`` 文件,仿照其他bat文件写脚本.


******************   
netcdf
******************

netcdf4.3.0
=================

netcdf4.3.0直接按照说明文件可以编译通过,如下所述:

使用 ``cmake`` 编译，自己根据情况修改库和 ``include`` , ``-D`` 表示配置

注意 ``cmake`` 命令是一整行

.. code-block:: bash

    mkdir build
    
    cd build
         
    cmake -G "Visual Studio 10" -DCMAKE_INSTALL_PREFIX=e:/build/netcdf 
        -DENABLE_NETCDF_4=ON -D"CURL_LIBRARY=E:/BUILD/libcurl/lib/libcurl_a.lib" 
        -D"CURL_INCLUDE_DIR=E:/BUILD/libcurl/include"  -D"BUILD_SHARED_LIBS=OFF" 
        -D"HDF5_LIB=E:/BUILD/hdf/1.8.11/lib/libhdf5.lib" 
        -D"HDF5_HL_LIB=E:/BUILD/hdf/1.8.11/lib/libhdf5_hl.lib" 
        -D"HDF5_INCLUDE_DIR=E:/BUILD/hdf/1.8.11/include" 
        -D"ZLIB_LIBRARY=E:/BUILD/hdf/1.8.11/lib/libzlib.lib;
        E:/BUILD/hdf/1.8.11/lib/libszip.lib"
        -D"ZLIB_INCLUDE_DIR=E:/BUILD/hdf5-1.8.11/build/ZLIB-prefix/src/ZLIB" 
        -D"SZIP_INCLUDE_DIR=E:/BUILD/hdf5-1.8.11/build/SZIP-prefix/src/SZIP/src" 
        -D"SZIP_DIR=E:\BUILD\hdf5-1.8.11\build\SZIP-prefix\src\SZIP-build" 
        -D"USE_SZIP=ON"   ..

     cmake --build . --config Release --target INSTALL

完整的命令解释：

.. code-block:: bash

    cmake -G "Visual Studio 10"                                  #使用vs2010编译
          -DCMAKE_INSTALL_PREFIX=e:/build/netcdf                  #安装路径 
          -DENABLE_NETCDF_4=ON                                    #编译netcdf4#   
          -D"CURL_LIBRARY=E:/BUILD/libcurl/lib/libcurl_a.lib"     #curl库路径
          -D"CURL_INCLUDE_DIR=E:/BUILD/libcurl/include"           #curl库头文件路径
          -D"BUILD_SHARED_LIBS=OFF"                               #静态库
          -D"HDF5_LIB=E:/BUILD/hdf/1.8.11/lib/libhdf5.lib"        #hdf5库
          -D"HDF5_HL_LIB=E:/BUILD/hdf/1.8.11/lib/libhdf5_hl.lib"  #hdf5库
          -D"HDF5_INCLUDE_DIR=E:/BUILD/hdf/1.8.11/include"        #hdf5库头文件
                                                                  #zlib库和szip库文件
          -D"ZLIB_LIBRARY=E:/BUILD/hdf/1.8.11/lib/libzlib.lib;
            E:/BUILD/hdf/1.8.11/lib/libszip.lib" 
                                                                  #zlib库头文件
          -D"ZLIB_INCLUDE_DIR=E:/BUILD/hdf5-1.8.11/build/ZLIB-prefix/src/ZLIB"   
                                                                  #szip库头文件  
          -D"SZIP_INCLUDE_DIR=E:/BUILD/hdf5-1.8.11/build/SZIP-prefix/src/SZIP/src"
                                                                  #szip源文件 
          -D"SZIP_DIR=E:\BUILD\hdf5-1.8.11\build\SZIP-prefix\src\SZIP-build" 
          -D"USE_SZIP=ON"                                         #使用sizp库
          ..                           #编译目录中cmakelist.ext文件，在上级目录中。

.. attention::

    * 如果需要编译64位的话，第一行需要修改为： ``cmake -G "Visual Studio 10 Win64"``

netcdf4.4.0
=================

.. important::

    * netcdf4.4.0 windows的cmake build有问题,需要修改cmake文件,详细介绍如下:

netcdf4.4.0 的 `cmake` windows版本有问题,详细信息可参见  `github issue #222 <https://github.com/Unidata/netcdf-c/issues/222>`_  

先需要修改CMakeLists.txt,在498行 ``ELSE`` 前面,加入 ``INCLUDE_DIRECTORIES(${HDF5_INCLUDE_DIR})``

再修改编译命令,将 ``HDF5_HL_LIB`` 修改为 ``HDF5_HL_LIBRARY`` , ``HDF5_LIB`` 改为 ``HDF5_C_LIBRARY`` ,具体如下:

.. code-block:: bash

    mkdir build
    
    cd build
         
    cmake -G "Visual Studio 10" -DCMAKE_INSTALL_PREFIX=d:/GDAL/netcdf4.4.0 
       -D"HDF5_DIR=E:/lib/hdf5-1.8.16/build"
       -D"ZLIB_LIBRARY=D:/GDAL/HDF5-1.8.16-win32/lib/libzlib.lib;
        D:/GDAL/HDF5-1.8.16-win32/lib/libszip.lib" 
       -D"ZLIB_INCLUDE_DIR=E:/BUILD/hdf5-1.8.16/build/ZLIB-prefix/src/ZLIB"    
       -D"SZIP_INCLUDE_DIR=E:/BUILD/hdf5-1.8.16/build/SZIP-prefix/src/SZIP/src" 
       -DENABLE_NETCDF_4=ON 
       -D"CURL_LIBRARY=D:/GDAL/libcurl-vc-x86-7.47.1/lib/libcurl_a.lib"    
       -D"CURL_INCLUDE_DIR=D:/GDAL/libcurl-vc-x86-7.47.1/include"    
       -D"HDF5_C_LIBRARY=D:/GDAL/HDF5-1.8.16-win32/lib/hdf5.lib"    
       -D"HDF5_HL_LIBRARY=D:/GDAL/HDF5-1.8.16-win32/lib/hdf5_hl.lib" 
       -D"HDF5_INCLUDE_DIR=D:/GDAL/HDF5-1.8.16-win32/include"  
       -D"BUILD_SHARED_LIBS=OFF"   ..
    
netcdf4.5.0
=================

使用 ``cmake`` 编译，自己根据情况修改库和 ``include`` , ``-D`` 表示配置，需要注意，此版本中 ``HAVE_HDF5_H`` 和 ``SZIP`` 需要单独配置，否则编译失败

注意 ``cmake`` 命令是一整行

.. code-block:: bash

    mkdir build
    
    cd build
         
    cmake -G "Visual Studio 10"  -DUSE_SZIP=ON -DUSE_HDF5=ON -DENABLE_DAP=ON 
      -D"BUILD_SHARED_LIBS=OFF" -DCMAKE_INSTALL_PREFIX=F:/BUILD/netcdf4.5
      -D“SZIP=F:/BUILD/HDF5-1.8.19-win32/lib/libszip.lib”
      -D"ZLIB_INCLUDE_DIR=F:/GDAL_BUILD/CMake-HDF5-1.8.19/build_x86/ZLIB-prefix/src/ZLIB" 
      -D"ZLIB_LIBRARY=F:/BUILD/HDF5-1.8.19-win32/lib/libzlib.lib"
      -DENABLE_NETCDF_4=ON -D"CURL_LIBRARY=F:/BUILD/libcurl-7.56.1-x84/lib/libcurl_a.lib"
      -D"CURL_INCLUDE_DIR=F:/BUILD/libcurl-7.56.1-x86/include"
      -D"HAVE_HDF5_H=F:/BUILD/HDF5-1.8.19-win32/include"
      -D"HDF5_INCLUDE_DIR=F:/BUILD/HDF5-1.8.19-win32/include"
      -D"HDF5_C_LIBRARY=F:/BUILD/HDF5-1.8.19-win32/lib/libhdf5.lib"
      -D"HDF5_HL_LIBRARY=F:/BUILD/HDF5-1.8.19-win32/lib/libhdf5_hl.lib" ..

     cmake --build . --config Release --target INSTALL

******************   
geos
******************
请直接下最新的 ``release`` 编译， ``svn`` 中部分存在问题，编译不过， ``geos`` 直接采用 ``nmake`` 可以生成静态库和动态库，在 ``src`` 子目录下。

.. code-block:: bash

    nmake /f Makefile.vc

******************   
proj.4
******************
修改 ``nmake.opt`` 文件中32、33行::

    # Uncomment the first for linking exes against DLL or second for static
    #EXE_PROJ =    proj_i.lib
    EXE_PROJ =    proj.lib

以及安装目录 ``INSTDIR`` ,然后开始编译即可
    
.. code-block:: bash

    nmake /f makefile.vc
    nmake /f makefile.vc install-all

*******************
libsqlite3
*******************

* 下载 ``sqlite3`` 源码，放入 ``src`` 目录中。
* 下载 `sqliteCmake <https://github.com/snikulov/sqlite.cmake.build>`_ ,放在 ``src`` 目录上一层
* 使用cmake编译

如下所示，设置安装路径和静态库即可

.. code-block:: bash

    mkdir build
    cd build
    cmake -G "Visual Studio 10" -DCMAKE_INSTALL_PREFIX=f:/gdal/sqlite3  ..
    cmake --build . --config Release --target INSTALL

.. attention::

    * 如果需要编译64位的话，第三行需要修改为： ``cmake -G "Visual Studio 10 Win64" ..``
    
******************
libwebp
******************
静态库，输出在 ``output/release-static/x86`` 中。完成后，拷贝 ``src/webp`` 到 ``include`` 文件夹中。

.. code-block:: bash

    nmake /f Makefile.vc CFG=release-static RTLIBCFG=static OBJDIR=output

.. attention::

    * 如果需要支持WINDOWS XP 的话,libwebp版本不能超过0.4.4,从libwep 0.5开始,不支持windows xp系统


*******************
openJPEG
*******************

.. code-block:: bash

    mkdir build
    cd build
    cmake -G   "Visual Studio 10" .. -DBUILD_SHARED_LIBS=OFF 
                                      -DCMAKE_INSTALL_PREFIX=f:/gdal/openjpeg 
                                      -DBUILD_THIRDPARTY=ON ..
    cmake --build . --config Release --target INSTALL

完成后，添加 ``#define OPJ_STATIC 1`` 到输出的  ``inlcude/openjpeg-2.0/openjpeg.h``  里

*******************
pcre
*******************

.. code-block:: bash

    mkdir build
    cd build
    cmake -G   "Visual Studio 10" -DCMAKE_INSTALL_PREFIX=f:/BUILd/pcre8.41.0_x86 .. 
    cmake --build . --config Release --target INSTALL

完成后需要修改 ``pcre.h`` ,添加 ``#define PCRE_STATIC`` 

******************
libKML
******************
``LibKML`` 需要从 ``github`` 中下最新代码, 找到其中 ``msvc`` 文件夹,打开 ``libkml.sln`` 工程,然后编译即可

第三方库在 ``third_party`` 文件夹中,主要需要编译的只有 ``uriparser-0.7.5.win32``  ``zlib-1.2.3`` 找到其中sln工程编译即可

******************
spatialite
******************

``spatialite`` 库可以让ogr中使用更多sql函数，方便矢量操作，但是编译比较复杂，依赖很多，需要依次编译 ``libiconv FreeXL libxml2 zlib sqlite3 geos PROJ.4`` ，后四个库 ``zlib sqlite3 geos PROJ.4`` 已经在前面编译完成，可以直接使用，其中 ``zlib`` 在 ``hdf5`` 中编译，然后依次按顺序编译。

libiconv
================
首先需要编译 ``libiconv`` 库， 2017年更新的1.15版应该可以在windows下编译通过，老的1.11版本也应该可以编译通过，1.15版vs2010工程链接：http://pan.baidu.com/s/1c2pmVxY 密码：lypx

直接使用 `iconv for windows <https://github.com/holy-shit/iconv-for-windows>`_  ，里面有 ``vs2010`` 工程文件，也可以直接编译1.14版。

编译完成后，将 ``libiconv.lib``重命名为 ``iconv.lib`` ，  ``iconv.obj`` 文件拷至 ``iconv`` 的库目录中,并重命名为 ``lib.obj`` ,后续过程中需要使用

FreeXL
================
下载 ``FreeXL`` 库，修改其中的 ``makefile.vc`` 文件中 ``iconv`` 的 ``include`` 和 ``lib`` 路径

修改 ``src/freexl.c`` 第93-109行，注释掉

.. code-block:: cpp

	#if defined(_WIN32) && !defined(__MINGW32__)
	/* MSVC compiler doesn't support lround() at all */
	/*
	static double
	__declspec(dllexport) round (double num)
	{
		double integer = ceil (num);
		if (num > 0)
		return integer - num > 0.5 ? integer - 1.0 : integer;
		return integer - num >= 0.5 ? integer - 1.0 : integer;
	}
	 
	static long
	__declspec(dllexport) lround (double num)
	{
		long integer = (long) round (num);
		return integer;
	}*/
	#endif

然后再vs命令行中编译

.. code-block:: bash

    nmake /f Makefile.vc
    nmake /f makefile.vc install

libxml2
================
下载 ``libxml2`` 库，转到目录 ``win32/VC10`` 下,使用 ``libxml2.sln`` 工程编译

* 编译前,先进入 ``vs2010`` 命令行工具中，在 ``win32`` 目录下,运行  ``cscript configure.js help``  
* 按照提示,添加iconv的include目录和iconv目录，运行 ``cscript configure.js  static=yes compiler=msvc prefix=c:\libxml2 include=c:\iconv\include lib=c:\iconv\lib`` ，此时在命令行中是无法编译通过的(不做这一步后面sln工程可能无法编译通过)
* 进入 ``win32/VC10`` 目录中,在 vs x86命令行工具下运行 ``nmake /f Makefile.msvc install``

spatialite
================
下载 ``spatialite`` 库，编辑 ``nmake.opt`` 文件和 ``makefile.vc`` 文件中各个库的路径和头文件路径,然后在   ``vs2010`` 命令行工具中输入:

.. code-block:: bash

    nmake /f Makefile.vc
    nmake /f makefile.vc install

编译完成后，将 ``sqlite`` 的头文件拷贝至 ``include/spatialite`` 文件夹中，后续编译将会使用。

.. attention::

    * ``freexl`` 和 ``spatialite`` 的动态库应该会编译出错,可以忽略掉继续下面步骤

******************
gdal
******************
修改nmake.opt文件,注意按照自己实际情况修改

.. code-block:: bash

	//291行
	# Uncomment out the following lines to enable LibKML support.
	#LIBKML_DIR = F:\GDAL_BUILD\libkml-master
	#LIBKML_INCLUDE = -IF:\GDAL_BUILD\libkml-master\src -IF:\GDAL_BUILD\libkml-master\src\kml -I$(LIBKML_DIR)/third_party/boost_1_34_1
	#LIBKML_LIBRARY = $(LIBKML_DIR)/msvc/x64/Release
	#LIBKML_LIBS =	$(LIBKML_LIBRARY)/libkmlbase.lib \
	#		$(LIBKML_LIBRARY)/libkmlconvenience.lib \
	#		$(LIBKML_LIBRARY)/libkmldom.lib \
	#		$(LIBKML_LIBRARY)/libkmlengine.lib \
	#		$(LIBKML_LIBRARY)/libkmlregionator.lib \
	#		$(LIBKML_LIBRARY)/libkmlxsd.lib \
	#		F:\GDAL_BUILD\libkml-master\third_party\zlib-1.2.3\contrib\minizip\x64\Release/minizip_static.lib \
	#		F:\GDAL_BUILD\libkml-master\third_party\uriparser-0.7.5\win32\Visual_Studio_2005\x64\Release/uriparser.lib 
	#		$(LIBKML_DIR)/third_party\zlib-1.2.3.win32/lib/minizip.lib \
	#		$(LIBKML_DIR)/third_party\zlib-1.2.3.win32/lib/zlib.lib
	#		$(LIBKML_DIR)/third_party\expat.win32/libexpat.lib 
	//改为,最下面三个不需要,否则会哟重复引用
	# Uncomment out the following lines to enable LibKML support.
	LIBKML_DIR = F:\GDAL_BUILD\libkml-master
	LIBKML_INCLUDE = -IF:\GDAL_BUILD\libkml-master\src -IF:\GDAL_BUILD\libkml-master\src\kml -I$(LIBKML_DIR)/third_party/boost_1_34_1
	LIBKML_LIBRARY = $(LIBKML_DIR)/msvc/Release
	LIBKML_LIBS =	$(LIBKML_LIBRARY)/libkmlbase.lib \
			$(LIBKML_LIBRARY)/libkmlconvenience.lib \
			$(LIBKML_LIBRARY)/libkmldom.lib \
			$(LIBKML_LIBRARY)/libkmlengine.lib \
			$(LIBKML_LIBRARY)/libkmlregionator.lib \
			$(LIBKML_LIBRARY)/libkmlxsd.lib \
			F:\GDAL_BUILD\libkml-master\third_party\zlib-1.2.3\contrib\minizip\Release/minizip_static.lib \
			F:\GDAL_BUILD\libkml-master\third_party\uriparser-0.7.5\win32\Visual_Studio_2005\Release/uriparser.lib 
	#		$(LIBKML_DIR)/third_party\zlib-1.2.3.win32/lib/minizip.lib \
	#		$(LIBKML_DIR)/third_party\zlib-1.2.3.win32/lib/zlib.lib
	#		$(LIBKML_DIR)/third_party\expat.win32/libexpat.lib 
	
    //303行
    # Uncomment for Expat support (required for KML, GPX and GeoRSS read support).
    #EXPAT_DIR = "C:\Program Files\Expat 2.0.1"
    #EXPAT_INCLUDE = -I$(EXPAT_DIR)/source/lib
    #EXPAT_LIB = $(EXPAT_DIR)/bin/libexpat.lib
    //改为
    # Uncomment for Expat support (required for KML, GPX and GeoRSS read support).
    EXPAT_DIR =E:\BUILD\lib\expat
    EXPAT_INCLUDE = -I$(EXPAT_DIR)/include
    EXPAT_LIB = E:\BUILD\lib\expat\lib\expat.lib

    ----------------------------------------------------------------------------------
   
    //331行
    # Uncomment the following and update to enable NCSA HDF Release 4 support.
    #HDF4_PLUGIN = NO
    #HDF4_DIR =    D:\warmerda\HDF41r5
    #HDF4_LIB =    /LIBPATH:$(HDF4_DIR)\lib Ws2_32.lib
    //改为
    # Uncomment the following and update to enable NCSA HDF Release 4 support.
    HDF4_PLUGIN = NO
    HDF4_DIR =    E:\BUILD\lib\4.2.9
    HDF4_LIB =    $(HDF4_DIR)\lib\libhdf.lib $(HDF4_DIR)\lib\libmfhdf.lib  \
          $(HDF4_DIR)\lib\libxdr.lib $(HDF4_DIR)\lib\libjpeg.lib  Ws2_32.lib

    ----------------------------------------------------------------------------------

    //336行
    # Uncomment the following and update to enable NCSA HDF Release 5 support.
    #HDF5_PLUGIN = NO
    #HDF5_DIR =    c:\warmerda\supportlibs\hdf5\5-164-win
    #HDF5_LIB =    $(HDF5_DIR)\dll\hdf5dll.lib 
    //改为
    # Uncomment the following and update to enable NCSA HDF Release 5 support.
    HDF5_PLUGIN = NO
    HDF5_DIR =    E:\BUILD\lib\1.8.11
    HDF5_LIB =    $(HDF5_DIR)\lib\libhdf5.lib $(HDF5_DIR)\lib\libhdf5_hl.lib \
         $(HDF5_DIR)\lib\libszip.lib $(HDF5_DIR)\lib\libzlib.lib 

    ----------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------
    
    //387行
    # SQLite Libraries
    #SQLITE_INC=-IN:\pkg\sqlite-win32
    #SQLITE_LIB=N:\pkg\sqlite-win32\sqlite3_i.lib
    //改为
    # SQLite Libraries
    SQLITE_INC=-If:\GDAL\libsqlite3\include
    SQLITE_LIB=f:\GDAL\libsqlite3\lib\sqlite3.lib
    
    //如果是使用spatialite,那么需要修改上面的注释
    # SQLite Libraries
    #SQLITE_INC=-IN:\pkg\sqlite-win32
    #SQLITE_LIB=N:\pkg\sqlite-win32\sqlite3_i.lib
    # For spatialite support, try this instead (assuming you grab the \
        libspatialite-amalgamation-2.3.1 and installed it in osgeo4w):
    # The -DSPATIALITE_AMALGAMATION, which cause "spatialite/sqlite3.h" \
        to be included instead of "sqlite3.h" might not be necessary
    # depending on the layout of the include directories. In case of compilation errors,\
        remove it.
    #SQLITE_INC=-IC:\osgeo4w\include -DHAVE_SPATIALITE -DSPATIALITE_AMALGAMATION
    #SQLITE_LIB=C:\osgeo4w\lib\spatialite_i.lib
    # Uncomment following line if libsqlite3 has been compiled with \
        SQLITE_HAS_COLUMN_METADATA=yes
    #SQLITE_HAS_COLUMN_METADATA=yes
    # Uncomment following line if spatialite is 4.1.2 or later
    #SPATIALITE_412_OR_LATER=yes
    //改为
    # SQLite Libraries
    #SQLITE_INC=-ID:\GDAL\sqlite3.13\include
    #SQLITE_LIB=D:\GDAL\sqlite3.13\lib\sqlite3-static.lib
    # For spatialite support, try this instead
    # (assuming you grab the libspatialite-amalgamation-2.3.1 and installed it in osgeo4w):
    # The -DSPATIALITE_AMALGAMATION, which cause "spatialite/sqlite3.h"  \
        to be included instead of "sqlite3.h" might not be necessary
    # depending on the layout of the include directories. In case of compilation errors,\
        remove it.
    SQLITE_INC=-IC:\OSGeo4w\libspatialite\include \
                -DHAVE_SPATIALITE -DSPATIALITE_AMALGAMATION
    SQLITE_LIB=C:\OSGeo4w\libspatialite\lib\spatialite.lib \
        "C:\OSGeo4w\libspatialite\lib\sqlite3-static.lib"  \
        "C:\OSGeo4w\libspatialite\lib\libxml2.lib" \
        "C:\OSGeo4w\libspatialite\lib\iconv.lib" 
    # Uncomment following line if libsqlite3 has been compiled \
        with SQLITE_HAS_COLUMN_METADATA=yes
    #SQLITE_HAS_COLUMN_METADATA=yes
    # Uncomment following line if spatialite is 4.1.2 or later
    SPATIALITE_412_OR_LATER=yes
    
    ----------------------------------------------------------------------------------
    ----------------------------------------------------------------------------------
    
    //401行
    # PCRE Library (REGEXP support for SQLite) for example from \
        http://sourceforge.net/projects/gnuwin32/files/pcre/7.0/pcre-7.0.exe/download
    #PCRE_INC=-I"C:\Program Files\GNUWin32\include" -DHAVE_PCRE
    #PCRE_LIB="C:\Program Files\GNUWin32\lib\pcre.lib"
    //改为
    # PCRE Library (REGEXP support for SQLite) for example from \
        http://sourceforge.net/projects/gnuwin32/files/pcre/7.0/pcre-7.0.exe/download
    PCRE_INC=-I"e:\BUILD\lib\PCRE\include" -DHAVE_PCRE
    PCRE_LIB="e:\BUILD\lib\PCRE\lib\pcre.lib"
   
    ----------------------------------------------------------------------------------
    //420行
    # Uncomment the following to enable NetCDF format.
    #NETCDF_PLUGIN = NO
    #NETCDF_SETTING=yes
    #NETCDF_LIB=C:\Software\netcdf\lib\netcdf.lib
    #NETCDF_INC_DIR=C:\Software\netcdf\include

    # Uncomment the following to add NC4 and HDF4 support
    #NETCDF_HAS_NC4 = yes
    #NETCDF_HAS_HDF4 = yes

    # PROJ.4 stuff
    # Uncomment the following lines to link PROJ.4 library statically. Otherwise
    # it will be linked dynamically during runtime.
    #PROJ_FLAGS = -DPROJ_STATIC
    #PROJ_INCLUDE = -Id:\projects\proj.4\src
    #PROJ_LIBRARY = d:\projects\proj.4\src\proj_i.lib
    //改为
    # Uncomment the following to enable NetCDF format.
    NETCDF_PLUGIN = NO
    NETCDF_SETTING=yes
    NETCDF_LIB=E:\BUILD\netcdf\lib\netcdf.lib
    NETCDF_INC_DIR=E:\BUILD\netcdf\include

    # Uncomment the following to add NC4 and HDF4 support
    NETCDF_HAS_NC4 = yes
    #NETCDF_HAS_HDF4 = yes

    # PROJ.4 stuff
    # Uncomment the following lines to link PROJ.4 library statically. Otherwise
    # it will be linked dynamically during runtime.
    PROJ_FLAGS = -DPROJ_STATIC
    PROJ_INCLUDE = -IC:\PROJ\include
    PROJ_LIBRARY = c:\PROJ\lib\proj.lib

    ----------------------------------------------------------------------------------

    //479行
    # Uncomment to use libcurl (DLL by default)
    # The cURL library is used for WCS, WMS, GeoJSON, SRS call importFromUrl(),\
        WFS, GFT, CouchDB, /vsicurl/ etc.
    #CURL_DIR=C:\curl-7.15.0
    #CURL_INC = -I$(CURL_DIR)/include
    # Uncoment following line to use libcurl as dynamic library
    #CURL_LIB = $(CURL_DIR)/libcurl_imp.lib wsock32.lib wldap32.lib winmm.lib
    # Uncoment following two lines to use libcurl as static library
    #CURL_LIB = $(CURL_DIR)/libcurl.lib wsock32.lib wldap32.lib winmm.lib
    #CURL_CFLAGS = -DCURL_STATICLIB
    //改为
    # Uncomment to use libcurl (DLL by default)
    # The cURL library is used for WCS, WMS, GeoJSON, SRS call importFromUrl(),\
        WFS, GFT, CouchDB, /vsicurl/ etc.
    CURL_DIR=E:\BUILD\lib\libcurl
    CURL_INC = -I$(CURL_DIR)/include
    # Uncoment following line to use libcurl as dynamic library
    #CURL_LIB = $(CURL_DIR)/libcurl_imp.lib wsock32.lib wldap32.lib winmm.lib
    # Uncoment following two lines to use libcurl as static library
    CURL_LIB = $(CURL_DIR)/lib/libcurl_a.lib wsock32.lib wldap32.lib winmm.lib
    CURL_CFLAGS = -DCURL_STATICLIB

    ----------------------------------------------------------------------------------

    //495行
    # Uncomment for GEOS support (GEOS >= 3.1.0 required)
    #GEOS_DIR=C:/warmerda/geos
    #GEOS_CFLAGS = -I$(GEOS_DIR)/capi -I$(GEOS_DIR)/source/headers -DHAVE_GEOS
    #GEOS_LIB     = $(GEOS_DIR)/source/geos_c_i.lib

    //改为
    # Uncomment for GEOS support (GEOS >= 3.1.0 required)
    GEOS_DIR=e:\BUILD\geos-3.4.2
    GEOS_CFLAGS = -I$(GEOS_DIR)/capi -I$(GEOS_DIR)/include -DHAVE_GEOS
    GEOS_LIB     = $(GEOS_DIR)/src/geos.lib
    
    ----------------------------------------------------------------------------------

    //505行
    # Uncomment for OpenJpeg (release v2.0.0) support
    #OPENJPEG_ENABLED = YES
    #OPENJPEG_CFLAGS = -IC:\openjpeg\include
    #OPENJPEG_LIB = C:\openjpeg\lib\openjpeg.lib

    //改为
    # Uncomment for OpenJpeg (release v2.0.0) support
    OPENJPEG_ENABLED = YES
    OPENJPEG_CFLAGS = -If:\GDAL\openjpeg\include
    OPENJPEG_LIB = f:\GDAL\openjpeg\lib\openjp2.lib
    
    ----------------------------------------------------------------------------------

    //530行
    # Uncomment for WEBP support
    #WEBP_ENABLED = YES
    #WEBP_CFLAGS = -IE:/libwebp-0.1-windows/dev/Include
    #WEBP_LIBS = e:/libwebp-0.1-windows/dev/lib/libwebp_a.lib

    //改为
    # Uncomment for WEBP support
    WEBP_ENABLED = YES
    WEBP_CFLAGS = -IF:\GDAL\libwebp-0.3.1\src
    WEBP_LIBS = f:\GDAL\libwebp\lib\libwebp.lib

    ----------------------------------------------------------------------------------

    //591行
    LINKER_FLAGS = $(EXTRA_LINKER_FLAGS) $(MSVC_VLD_LIB) $(LDEBUG)

    //改为,防止openjepg库链接出错
    LINKER_FLAGS = $(EXTRA_LINKER_FLAGS) $(MSVC_VLD_LIB) $(LDEBUG) /NODEFAULTLIB:LIBCMT

需要中文路径支持，请参看 `关于GDAL180中文路径不能打开的问题分析与解决 <http://blog.csdn.net/liminlu0314/article/details/6610069>`_

我选择的是方案2,
修改 ``GDAL_HOME\frmts\gdalallregister.cpp`` 文件73行左右， ``GDALAllRegister()`` 函数，以及 ``GDAL_HOME\ogr\ogrsf_frmts\generic\ogrregisterall.cpp`` 38行左右， ``OGRRegisterAll()`` 函数，在函数最前面添加

.. code-block:: c++

     CPLSetConfigOption("GDAL_FILENAME_IS_UTF8","NO");

然后使用nmake即可，需要debug的话，加上参数 ``debug=1``

.. code-block:: bash

    nmake /f makefile.vc
    nmake /f makefile.vc devinstall

如果需要64位，请修改153行左右， ``#WIN64=YES`` 为 ``WIN64=YES``

``GDAL 2.0`` 的版本中,某些gtiff的文件投影默认读不出来,需要添加 ``GDAL_DATA`` 环境变量,也可以删除 ``\frmts\gtiff\gt_wkt_srs.cpp`` 中 716-733行, ``GDAL dev`` 中已经修复, `Ticket 6210 <https://trac.osgeo.org/gdal/ticket/6210>`_  :

.. code-block:: c++

    if( psDefn->Model == ModelTypeProjected &&
        psDefn->PCS != KvUserDefined &&
        GDALGTIFKeyGetSHORT(hGTIF, ProjectionGeoKey, &tmp, 0, 1  ) == 0 &&
        GDALGTIFKeyGetSHORT(hGTIF, ProjCoordTransGeoKey, &tmp, 0, 1  ) == 0 &&
        GDALGTIFKeyGetSHORT(hGTIF, GeographicTypeGeoKey, &tmp, 0, 1  ) == 0 &&
        GDALGTIFKeyGetSHORT(hGTIF, GeogGeodeticDatumGeoKey, &tmp, 0, 1  ) == 0 &&
        GDALGTIFKeyGetSHORT(hGTIF, GeogEllipsoidGeoKey, &tmp, 0, 1  ) == 0 &&
        CSLTestBoolean(CPLGetConfigOption("GTIFF_IMPORT_FROM_EPSG", "YES")) )
    {
        // Save error state as importFromEPSGA() will call CPLReset()
        int errNo = CPLGetLastErrorNo();
        CPLErr eErr = CPLGetLastErrorType();
        const char* pszTmp = CPLGetLastErrorMsg();
        char* pszLastErrorMsg = CPLStrdup(pszTmp ? pszTmp : "");
        CPLPushErrorHandler(CPLQuietErrorHandler);
        OGRErr eImportErr = oSRS.importFromEPSG(psDefn->PCS);
        CPLPopErrorHandler();
        // Restore error state
        CPLErrorSetState( eErr, errNo, pszLastErrorMsg);
        CPLFree(pszLastErrorMsg);
        bGotFromEPSG = (eImportErr == OGRERR_NONE);
    }

******************
编译结果下载
******************
放在百度网盘里，有hdf4、hdf5、curl、expat、gdal等，会更新，需要者自取。

.. note::

    `编译完成的库 <http://pan.baidu.com/s/13R6i5>`_