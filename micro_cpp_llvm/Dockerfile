FROM foonathan/micro_cpp_gcc

ARG LLVM_FULL_VERSION=7.0.0
ARG CLANG_BINARY=clang-7

# install build dependencies
RUN apk add --no-cache python3 ninja \
# download source code
    && wget -q http://releases.llvm.org/$LLVM_FULL_VERSION/llvm-$LLVM_FULL_VERSION.src.tar.xz -O llvm.tar.xz \
    && wget -q http://releases.llvm.org/$LLVM_FULL_VERSION/cfe-$LLVM_FULL_VERSION.src.tar.xz  -O clang.tar.xz \
# extract
    && tar -xf llvm.tar.xz && tar -xf clang.tar.xz \
    && mv llvm-$LLVM_FULL_VERSION.src llvm-src && mv cfe-$LLVM_FULL_VERSION.src llvm-src/tools/clang \
# build
    && mkdir llvm-build && mkdir /opt/llvm \
    && cd llvm-build \
    && cmake -G"Ninja" -DCMAKE_INSTALL_PREFIX="/opt/llvm" -DCMAKE_BUILD_TYPE=Release \
                       -DGCC_INSTALL_PREFIX="/usr" \
                       -DLIBCLANG_BUILD_STATIC=ON -DLLVM_BUILD_EXAMPLES=OFF -DLLVM_BUILD_TESTS=OFF \
                        ../llvm-src \
    && cmake --build . \
    && cmake --build . --target install \
    # static libclang library is not installed, so have to do manually
    && find . -name 'libclang.a' -exec cp {} /opt/llvm/lib \; \
    && cd ../ \
# remove build files
    && rm -r llvm-src/ llvm-build/ \
    && rm llvm.tar.xz clang.tar.xz \
# remove contents of share and libexec directory (it is not needed for our purposes)
    && rm -r /opt/llvm/share/* \
    && rm -r /opt/llvm/libexec/* \
# remove unnecessary tools (i.e. not clang and not llvm-config)
    && find /opt/llvm/bin -type f -not -name $CLANG_BINARY -not -name 'llvm-config' -delete \
# remove symlinks broken by the command above
    && find -L /opt/llvm/bin -maxdepth 1 -type l -delete \
# remove shared libraries that were built anyway
    && find /opt/llvm/lib \( -type f -or -type l \) \( -name '*.so' -or -name '*.so.*' \) -delete \
# remove build dependencies
    && apk del -r python3 ninja

# update PATH
ENV PATH="/opt/llvm/bin:${PATH}"
