# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Steven Allen <steven@stebalien.com>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgbase=ollama
pkgname=(ollama ollama-cuda ollama-rocm)
pkgver=0.2.8
_ollama_commit=c0648233f2236f82f6830d2aaed552ae0f72379b # tag: v0.2.8
_llama_cpp_commit=$(curl -sL "https://github.com/ollama/ollama/tree/$_ollama_commit/llm" | tr ' ' '\n' | tr '"' '\n' | grep ggerganov | cut -d/ -f5 | head -1)
pkgrel=1
pkgdesc='Create, run and share large language models (LLMs)'
arch=(x86_64)
url='https://github.com/ollama/ollama'
license=(MIT)
makedepends=(clblast cmake cuda git go rocm-hip-sdk rocm-opencl-sdk)
source=(git+$url#commit=$_ollama_commit
        llama.cpp::git+https://github.com/ggerganov/llama.cpp#commit=$_llama_cpp_commit
        ollama.service
        sysusers.conf
        tmpfiles.d)
b2sums=('7e67a4d25d0a21bc3ce6588e63d48c5c7ec7a8830709902fa61239449abb82b506f197b570d01f5f8ce2b0b87796c5d8a8d648e471e3ec09dc31cc989ffc7477'
        'aca0f7a361293545b322b500f243208354df2b6064da3ce4ba23eb119475113a635b1dce67e3641856ec523672ffea22775611511599bb6dd90c85c17b07c1b6'
        '18a1468f5614f9737f6ff2e6c7dfb3dfc0ba82836a98e3f14f8e544e3aba8f74ef0a03c5376a0d0aa2e59e948701d7c639dda69477b051b732896021e753e32e'
        '3aabf135c4f18e1ad745ae8800db782b25b15305dfeaaa031b4501408ab7e7d01f66e8ebb5be59fc813cfbff6788d08d2e48dcf24ecc480a40ec9db8dbce9fec'
        'e8f2b19e2474f30a4f984b45787950012668bf0acb5ad1ebb25cd9776925ab4a6aa927f8131ed53e35b1c71b32c504c700fe5b5145ecd25c7a8284373bb951ed')

prepare() {
  echo "Using ollama git submodule llama.cpp commit $_llama_cpp_commit"

  # Prepare the git submodule by copying in files (the build process is sensitive to symlinks)
  rm -frv $pkgbase/llm/llama.cpp
  cp -r llama.cpp $pkgbase/llm/llama.cpp

  # Set the CMake build type to "Release"
  sed -i 's,T_CODE=on,T_CODE=on -D CMAKE_BUILD_TYPE=Release,g' $pkgbase/llm/generate/gen_linux.sh

  # Copy the ollama directory to ollama-cuda and ollama-rocm
  cp -r $pkgbase $pkgbase-cuda
  cp -r $pkgbase $pkgbase-rocm

  # Prepare the ollama-rocm directory for building for ROCm
  cd $pkgbase-rocm/llm/generate
  sed -i 's,g++,/opt/rocm/llvm/bin/clang++,g' gen_common.sh
  sed -i 's,T_CODE=on,T_CODE=on -D LLAMA_HIPBLAS=1 -D AMDGPU_TARGETS=gfx1030,g' gen_linux.sh
}

build() {
  export CFLAGS+=' -w'
  export CXXFLAGS+=' -w'
  export CGO_CFLAGS="$CFLAGS" CGO_CPPFLAGS="$CPPFLAGS" CGO_CXXFLAGS="$CXXFLAGS" CGO_LDFLAGS="$LDFLAGS"

  local goflags='-buildmode=pie -trimpath -mod=readonly -modcacherw'
  local ldflags="-linkmode=external -buildid= -X github.com/ollama/ollama/version.Version=$pkgver -X github.com/ollama/ollama/server.mode=release"

  # Ollama with CPU only support
  export ROCM_PATH=/disabled
  export CUDA_LIB_DIR=/disabled
  cd $pkgbase
  go generate ./...
  go build $goflags -ldflags="$ldflags"

  # Ollama with CUDA support
  cd "$srcdir/$pkgbase-cuda"
  export CUDA_LIB_DIR=/opt/cuda/targets/x86_64-linux/lib
  go generate ./...
  go build $goflags -ldflags="$ldflags" -tags cuda

  # Ollama with ROCm support
  cd "$srcdir/$pkgbase-rocm"
  export CUDA_LIB_DIR=/disabled
  export ROCM_PATH=/opt/rocm
  export CC=/opt/rocm/llvm/bin/clang
  export CXX=/opt/rocm/llvm/bin/clang++
  export CFLAGS+=' -fcf-protection=none'
  export CXXFLAGS+=' -fcf-protection=none'
  export OLLAMA_CUSTOM_CPU_DEFS='-DLLAMA_AVX=on -DLLAMA_AVX2=on -DAMDGPU_TARGETS=gfx1030 -DLLAMA_F16C=on -DLLAMA_FMA=on -DLLAMA_LTO=on -DLLAMA_HIPBLAS=1 -DCMAKE_BUILD_TYPE=Release'
  go generate ./...
  go build $goflags -ldflags="$ldflags" -tags rocm
}

check() {
  $pkgbase/$pkgbase --version > /dev/null
  $pkgbase-cuda/$pkgbase --version > /dev/null
  $pkgbase-rocm/$pkgbase --version > /dev/null

  cd $pkgbase
  go test .
  cd ../$pkgbase-cuda
  go test .
  cd ../$pkgbase-rocm
  go test .
}

package_ollama() {
  pkgdesc='Create, run and share large language models (LLMs)'

  install -Dm755 $pkgname/$pkgbase "$pkgdir/usr/bin/$pkgbase"
  install -dm755 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_ollama-cuda() {
  pkgdesc='Create, run and share large language models (LLMs) with CUDA'
  provides=(ollama)
  conflicts=(ollama)
  optdepends=('nvidia-utils: monitor GPU usage with nvidia-smi')

  install -Dm755 $pkgname/$pkgbase "$pkgdir/usr/bin/$pkgbase"
  install -dm755 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_ollama-rocm() {
  pkgdesc='Create, run and share large language models (LLMs) with ROCm'
  provides=(ollama)
  conflicts=(ollama)
  depends=(hipblas)
  optdepends=('rocm-smi-lib: monitor GPU usage with rocm-smi')

  install -Dm755 $pkgname/$pkgbase "$pkgdir/usr/bin/$pkgbase"
  install -dm755 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
