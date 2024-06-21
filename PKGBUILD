# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Steven Allen <steven@stebalien.com>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgbase=ollama
pkgname=(ollama ollama-cuda ollama-rocm)
pkgver=0.1.45
_ollamacommit=e01e535cbbb92e0d9645bd726e259e7d8a6c7598 # tag: v0.1.45
# The llama.cpp git submodule commit hash can be found here:
# https://github.com/ollama/ollama/tree/v0.1.45/llm
_llama_cpp_commit=7c26775adb579e92b59c82e8084c07a1d0f75e9c
pkgrel=1
pkgdesc='Create, run and share large language models (LLMs)'
arch=(x86_64)
url='https://github.com/ollama/ollama'
license=(MIT)
makedepends=(clblast cmake cuda git go rocm-hip-sdk rocm-opencl-sdk)
source=(git+$url#commit=$_ollamacommit
        llama.cpp::git+https://github.com/ggerganov/llama.cpp#commit=$_llama_cpp_commit
        ollama.service
        sysusers.conf
        tmpfiles.d)
b2sums=('109eee6c42771c3fb3170fbf1602c624c051b56e1f74464a6f1f3ad22ba7b78130b1e295ca912b3c94a0531b12a6111abe856a2f2e8ba1a6ad570a9f4a78bfcc'
        'df130b63f823035cda495e596959ec85af6c49be03cff56623096b354704e5014190c51efd581e473a0c4c924945a5e422ab29bb6366906652345eedae2b79a6'
        '18a1468f5614f9737f6ff2e6c7dfb3dfc0ba82836a98e3f14f8e544e3aba8f74ef0a03c5376a0d0aa2e59e948701d7c639dda69477b051b732896021e753e32e'
        '3aabf135c4f18e1ad745ae8800db782b25b15305dfeaaa031b4501408ab7e7d01f66e8ebb5be59fc813cfbff6788d08d2e48dcf24ecc480a40ec9db8dbce9fec'
        'e8f2b19e2474f30a4f984b45787950012668bf0acb5ad1ebb25cd9776925ab4a6aa927f8131ed53e35b1c71b32c504c700fe5b5145ecd25c7a8284373bb951ed')

prepare() {
  # Prepare the git submodule by copying in files (the build process is sensitive to symlinks)
  rm -frv $pkgbase/llm/llama.cpp
  cp -r llama.cpp $pkgbase/llm/llama.cpp

  # Enable LTO and set the CMake build type to "Release"
  sed -i 's,T_CODE=on,T_CODE=on -D LLAMA_LTO=on -D CMAKE_BUILD_TYPE=Release,g' $pkgbase/llm/generate/gen_linux.sh

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
