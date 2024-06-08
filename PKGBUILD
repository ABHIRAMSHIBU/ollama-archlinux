# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Steven Allen <steven@stebalien.com>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgbase=ollama
pkgname=(ollama ollama-cuda ollama-rocm)
pkgver=0.1.42
_ollamacommit=385a32ecb5b2987f9cd7decaf0052f0a316ac6f6 # tag: v0.1.42
# The llama.cpp git submodule commit hash can be found here:
# https://github.com/ollama/ollama/tree/v0.1.42/llm
_llama_cpp_commit=5921b8f089d3b7bda86aac5a66825df6a6c10603
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
b2sums=('f0bfb4bb1c1133b722bb4b185c46ee1a9b4407e7f61292cdf2f2d69b6a24078c35bd1ad4069d8602943e7e1caa85dfae0233a912408d5b6337afea720cf9f684'
        '21643fc46052e673f747606a774bb7b161e41e3c0166700281d995018003d0af573db6d7c2ddf68765449545b72b41713f9335aa3485df90871431bc66097b27'
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

  local goflags="-buildmode=pie -trimpath -mod=readonly -modcacherw"
  local ldflags="-linkmode=external -buildid= -X github.com/ollama/ollama/version.Version=${pkgver} -X github.com/ollama/ollama/server.mode=release"

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
  export CFLAGS+=' -fcf-protection=none'
  export CXX=/opt/rocm/llvm/bin/clang++
  export CXXFLAGS+=' -fcf-protection=none'
  export LLAMA_CCACHE=OFF
  export OLLAMA_CUSTOM_CPU_DEFS="-DLLAMA_AVX=on -DLLAMA_AVX2=on -DAMDGPU_TARGETS=gfx1030 -DLLAMA_F16C=on -DLLAMA_FMA=on -DLLAMA_LTO=on -DLLAMA_HIPBLAS=1 -DCMAKE_BUILD_TYPE=Release"
  go generate ./...
  go build $goflags -ldflags="$ldflags" -tags rocm
}

check() {
  local ollama_exe="$srcdir/$pkgbase/$pkgbase"
  local ollama_cuda_exe="$srcdir/$pkgbase-cuda/$pkgbase"
  local ollama_rocm_exe="$srcdir/$pkgbase-rocm/$pkgbase"

  "$ollama_exe" --version > /dev/null
  "$ollama_cuda_exe" --version > /dev/null
  "$ollama_rocm_exe" --version > /dev/null

  if [ $(grep -c CUDA "$ollama_exe") -gt $(grep -c CUDA "$ollama_cuda_exe") ]; then
      echo "The number of 'CUDA' occurrences in ollama is greater than in ollama-cuda."
      exit 1
  fi

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

  install -Dm755 $pkgname/$pkgbase "$pkgdir/usr/bin/$pkgbase"
  install -dm755 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
