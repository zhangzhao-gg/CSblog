现在业务上有需求，需要使用go重构python写的一个微服务。
这其中会用到Faiss进行向量检索,Faiss天然对 python极友好，而对go的支持就非常有限，笔者查了一下，找到一个go封装的faiss库，不过最近一次更新都是四年前了。不确定能不能用，打算先进行编译下来，然后cgo调用。cgo调用请看下一篇。
该go-faiss项目地址为：https://github.com/DataIntelligenceCrew/go-faiss

1 git clone下来faiss库
`git clone https://github.com/facebookresearch/faiss.git`

2安装编译工具
`brew install cmake openblas`

3进行编译，
`cmake -B build -DFAISS_ENABLE_GPU=OFF -DFAISS_ENABLE_C_API=ON -DBUILD_SHARED_LIBS=ON .`

报错
```
(base) zhangzhao@zhangzhaodeMac-mini faiss % cmake -B build -DFAISS_ENABLE_GPU=OFF -DFAISS_ENABLE_C_API=ON -DBUILD_SHARED_LIBS=ON .
-- The CXX compiler identification is AppleClang 17.0.0.17000013
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
CMake Error at /opt/homebrew/share/cmake/Modules/FindPackageHandleStandardArgs.cmake:227 (message):
  Could NOT find OpenMP_CXX (missing: OpenMP_CXX_FLAGS OpenMP_CXX_LIB_NAMES)
Call Stack (most recent call first):
  /opt/homebrew/share/cmake/Modules/FindPackageHandleStandardArgs.cmake:591 (_FPHSA_FAILURE_MESSAGE)
  /opt/homebrew/share/cmake/Modules/FindOpenMP.cmake:651 (find_package_handle_standard_args)
  faiss/CMakeLists.txt:391 (find_package)
```

原因是有库没装
` brew install libomp`
然后换下面更详细的编译命令：

```
cmake -B build \
  -DFAISS_ENABLE_GPU=OFF \
  -DFAISS_ENABLE_C_API=ON \
  -DBUILD_SHARED_LIBS=ON \
  -DFAISS_ENABLE_PYTHON=OFF \
  -DOpenMP_CXX_FLAGS="-Xpreprocessor -fopenmp" \
  -DOpenMP_C_FLAGS="-Xpreprocessor -fopenmp" \
  -DOpenMP_CXX_LIB_NAMES="omp" \
  -DOpenMP_C_LIB_NAMES="omp" \
  -DOpenMP_omp_LIBRARY="/opt/homebrew/opt/libomp/lib/libomp.dylib" \
```

成功了一半，但是还报错：

```
CMake Error at perf_tests/CMakeLists.txt:21 (find_package):
  By not providing "Findgflags.cmake" in CMAKE_MODULE_PATH this project has
  asked CMake to find a package configuration file provided by "gflags", but
  CMake did not find one.

  Could not find a package configuration file provided by "gflags" with any
  of the following names:

    gflagsConfig.cmake
    gflags-config.cmake

  Add the installation prefix of "gflags" to CMAKE_PREFIX_PATH or set
  "gflags_DIR" to a directory containing one of the above files.  If "gflags"
  provides a separate development package or SDK, be sure it has been
  installed.
```

问题在于还有库没装

`brew install gflags`

然后再执行上面的指令，编译成功！




下面开始构建->
构建又遇到了问题
```
(base) zhangzhao@zhangzhaodeMac-mini faiss % make -C build
[  0%] Building CXX object faiss/CMakeFiles/faiss.dir/AutoTune.cpp.o
In file included from /Users/zhangzhao/Desktop/workspace/faiss/faiss/AutoTune.cpp:22:
In file included from /Users/zhangzhao/Desktop/workspace/faiss/faiss/IndexHNSW.h:18:
/Users/zhangzhao/Desktop/workspace/faiss/faiss/impl/HNSW.h:14:10: fatal error: 'omp.h' file not found
   14 | #include <omp.h>
      |          ^~~~~~~
1 error generated.
make[2]: *** [faiss/CMakeFiles/faiss.dir/AutoTune.cpp.o] Error 1
make[1]: *** [faiss/CMakeFiles/faiss.dir/all] Error 2
make: *** [all] Error 2

```
这个问题的原因是虽然 CMake 找到了 OpenMP 库，但在编译时没有正确地将头文件路径（-I/path/to/include）传递给编译器。我们需要明确指定 -I${OMP_PREFIX}/include 参数。
方法为：
```
rm -rf build
OMP_PREFIX=$(brew --prefix libomp)
cmake -B build \
  -DFAISS_ENABLE_GPU=OFF \
  -DFAISS_ENABLE_C_API=ON \
  -DBUILD_SHARED_LIBS=ON \
  -DFAISS_ENABLE_PYTHON=OFF \
  -DFAISS_BUILD_PERF_TESTS=OFF \
  -DOpenMP_CXX_FLAGS="-Xpreprocessor -fopenmp -I${OMP_PREFIX}/include" \
  -DOpenMP_C_FLAGS="-Xpreprocessor -fopenmp -I${OMP_PREFIX}/include" \
  -DOpenMP_CXX_LIB_NAMES="omp" \
  -DOpenMP_C_LIB_NAMES="omp" \
  -DOpenMP_omp_LIBRARY="${OMP_PREFIX}/lib/libomp.dylib" \
```
编译成功后再进行
`make -C build -j$(sysctl -n hw.logicalcpu)`

又遇到了新问题
这次报错是
```
/Users/zhangzhao/Desktop/workspace/faiss/tests/test_hamming.cpp:304:24: error: cannot initialize a member subobject of type 'TI *' (aka 'long long *') with an rvalue of type 'value_type *' (aka 'long *')
  304 |                 na, k, ids_gen.data(), dist_gen.data()};
      |                        ^~~~~~~~~~~~~~
/Users/zhangzhao/Desktop/workspace/faiss/tests/test_hamming.cpp:313:13: error: no viable overloaded '='
  313 |         res = {na, k, ids_ham_knn.data(), dist_ham_knn.data()};
      |         ~~~ ^ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/Users/zhangzhao/Desktop/workspace/faiss/faiss/utils/Heap.h:478:8: note: candidate function (the implicit copy assignment operator) not viable: cannot convert initializer list argument to 'const HeapArray<CMax<int, long long>>'
  478 | struct HeapArray {
      |        ^~~~~~~~~
/Users/zhangzhao/Desktop/workspace/faiss/faiss/utils/Heap.h:478:8: note: candidate function (the implicit move assignment operator) not viable: cannot convert initializer list argument to 'HeapArray<CMax<int, long long>>'
  478 | struct HeapArray {
      |        ^~~~~~~~~
/Users/zhangzhao/Desktop/workspace/faiss/tests/test_mmap.cpp:109:32: warning: 'tmpnam' is deprecated: This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of tmpnam(3), it is highly recommended that you use mkstemp(3) instead. [-Wdeprecated-declarations]
  109 |     std::string tmpname = std::tmpnam(nullptr);
      |                                ^
/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/_stdio.h:287:1: note: 'tmpnam' has been explicitly marked deprecated here
  287 | __deprecated_msg("This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of tmpnam(3), it is highly recommended that you use mkstemp(3) instead.")
      | ^
/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/sys/cdefs.h:218:48: note: expanded from macro '__deprecated_msg'
  218 |         #define __deprecated_msg(_msg) __attribute__((__deprecated__(_msg)))
      |                                                       ^
/Users/zhangzhao/Desktop/workspace/faiss/tests/test_mmap.cpp:216:32: warning: 'tmpnam' is deprecated: This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of tmpnam(3), it is highly recommended that you use mkstemp(3) instead. [-Wdeprecated-declarations]
  216 |     std::string tmpname = std::tmpnam(nullptr);
      |                                ^
/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/_stdio.h:287:1: note: 'tmpnam' has been explicitly marked deprecated here
  287 | __deprecated_msg("This function is provided for compatibility reasons only.  Due to security concerns inherent in the design of tmpnam(3), it is highly recommended that you use mkstemp(3) instead.")
      | ^
/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/sys/cdefs.h:218:48: note: expanded from macro '__deprecated_msg'
  218 |         #define __deprecated_msg(_msg) __attribute__((__deprecated__(_msg)))
      |                                                       ^
2 errors generated.
make[2]: *** [tests/CMakeFiles/faiss_test.dir/test_hamming.cpp.o] Error 1
make[2]: *** Waiting for unfinished jobs....
2 warnings generated.
make[1]: *** [tests/CMakeFiles/faiss_test.dir/all] Error 2
make: *** [all] Error 2
```

现在的问题是测试代码中的类型不匹配错误。这些是测试代码的问题，不影响核心库的编译。我们可以禁用测试来避免这些问题。
反正我们在go里也是运行核心代码

```
rm -rf build

OMP_PREFIX=$(brew --prefix libomp)

cmake -B build \
  -DFAISS_ENABLE_GPU=OFF \
  -DFAISS_ENABLE_C_API=ON \
  -DBUILD_SHARED_LIBS=ON \
  -DFAISS_ENABLE_PYTHON=OFF \
  -DFAISS_BUILD_PERF_TESTS=OFF \
  -DFAISS_BUILD_TESTS=OFF \
  -DFAISS_BUILD_BENCHMARK=OFF \
  -DFAISS_OPT_LEVEL=generic \
  -DOpenMP_CXX_FLAGS="-Xpreprocessor -fopenmp -I${OMP_PREFIX}/include" \
  -DOpenMP_C_FLAGS="-Xpreprocessor -fopenmp -I${OMP_PREFIX}/include" \
  -DOpenMP_CXX_LIB_NAMES="omp" \
  -DOpenMP_C_LIB_NAMES="omp" \
  -DOpenMP_omp_LIBRARY="${OMP_PREFIX}/lib/libomp.dylib" \
  -DCMAKE_BUILD_TYPE=Release \
  .

make -C build -j$(sysctl -n hw.logicalcpu) faiss faiss_c
```
ok这回真的成功了！！
```
[ 88%] Building CXX object c_api/CMakeFiles/faiss_c.dir/IndexBinary_c.cpp.o
[ 94%] Building CXX object c_api/CMakeFiles/faiss_c.dir/IndexBinaryIVF_c.cpp.o
[ 94%] Building CXX object c_api/CMakeFiles/faiss_c.dir/IndexScalarQuantizer_c.cpp.o
[ 94%] Building CXX object c_api/CMakeFiles/faiss_c.dir/MetaIndexes_c.cpp.o
[ 94%] Building CXX object c_api/CMakeFiles/faiss_c.dir/clone_index_c.cpp.o
[ 94%] Building CXX object c_api/CMakeFiles/faiss_c.dir/error_impl.cpp.o
[ 94%] Building CXX object c_api/CMakeFiles/faiss_c.dir/index_factory_c.cpp.o
[ 94%] Building CXX object c_api/CMakeFiles/faiss_c.dir/index_io_c.cpp.o
[100%] Building CXX object c_api/CMakeFiles/faiss_c.dir/impl/AuxIndexStructures_c.cpp.o
[100%] Building CXX object c_api/CMakeFiles/faiss_c.dir/impl/io_c.cpp.o
[100%] Building CXX object c_api/CMakeFiles/faiss_c.dir/utils/distances_c.cpp.o
[100%] Building CXX object c_api/CMakeFiles/faiss_c.dir/utils/utils_c.cpp.o
[100%] Linking CXX shared library libfaiss_c.dylib
[100%] Built target faiss_c
(base) zhangzhao@zhangzhaodeMac-mini faiss % 
```

下面进行安装 ，这个指令是直接安装核心库，跳过测试模块，不然又会报错
`sudo make -C build install/fast 
`
但是果不其然，还是报错了，再安装完faiss等核心库之后，再安装googlebenchmark报错了：
```
CMake Error at _deps/googlebenchmark-build/src/cmake_install.cmake:41 (file):
  file INSTALL cannot find
  "/Users/zhangzhao/Desktop/workspace/faiss/build/_deps/googlebenchmark-build/src/libbenchmark.1.9.4.66.dylib":
  No such file or directory.
Call Stack (most recent call first):
  _deps/googlebenchmark-build/cmake_install.cmake:42 (include)
  perf_tests/cmake_install.cmake:42 (include)
  cmake_install.cmake:72 (include)
```
这个错误可以不用管，因为我需要的faiss已经有了，这就不管它了

总的来说，编译安装成功！！！
```
(base) zhangzhao@zhangzhaodeMac-mini faiss % ls -la /usr/local/lib/libfaiss*
-rwxr-xr-x@ 1 root  wheel   172336 11 10 16:09 /usr/local/lib/libfaiss_c.dylib
-rwxr-xr-x@ 1 root  wheel  5928288 11 10 15:43 /usr/local/lib/libfaiss.dylib
(base) zhangzhao@zhangzhaodeMac-mini faiss % ls -la /usr/local/include/faiss/c_api/Index_c.h
-rw-r--r--@ 1 root  wheel  8611 11 10 14:55 /usr/local/include/faiss/c_api/Index_c.h
(base) zhangzhao@zhangzhaodeMac-mini faiss % 
```
mac的库文件也是在的。没什么问题。