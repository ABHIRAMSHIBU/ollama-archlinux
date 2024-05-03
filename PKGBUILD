# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgbase=ollama
pkgname=(ollama ollama-cuda ollama-rocm)
pkgver=0.1.33
pkgrel=1
pkgdesc='Create, run and share large language models (LLMs)'
arch=(x86_64)
url='https://github.com/ollama/ollama'
license=(MIT)
_ollamacommit=9164b0161bcb24e543cba835a8863b80af2c0c21 # tag: v0.1.33
# The llama.cpp git submodule commit hash can be found here:
# https://github.com/ollama/ollama/tree/v0.1.33/llm
_llama_cpp_commit=952d03dbead16e4dbdd1d3458486340673cc2465
makedepends=(clblast cmake cuda git go rocm-hip-sdk rocm-opencl-sdk)
source=(git+$url#commit=$_ollamacommit
        llama.cpp::git+https://github.com/ggerganov/llama.cpp#commit=$_llama_cpp_commit
        ollama.service
        ollama-rocm.service
        sysusers.conf
        tmpfiles.d)
b2sums=('7308a93697cb66b88869ced655c178c2598315252c0daa50b6a2f2dfe20a10b311ceb1a30b1310a61c6386b2a99e91b741c31b154967e626c4225670e2919b70'
        'f2e7fdf5eaf21f725fa868b4dd18ab602a4ec3f6bdd703bc674bf6537e9b57c7c87be945a2b6b774b6a7abfb6a36b2d15cf40ea67ba0c7958ced7eea46880702'
        '2bf4c2076b7841de266ec40da2e2cbb675dcbfebfa8aed8d4ede65435854cb43d39ea32bc9210cfc28a042382dd0094a153e351edfa5586eb7c6a0783f3bc517'
        '1397e36c703799e60bc53ec365f1f9f8152091d48d704adea2fda3e7599d938274bec825b09b0a7eb56b7fce2bed1b6d1a034687138d68c51a0133d75a1eb12f'
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
  export ROCM_PATH=/disabled
  export CUDA_LIB_DIR=/disabled
  export CGO_CFLAGS="$CFLAGS" CGO_CPPFLAGS="$CPPFLAGS" CGO_CXXFLAGS="$CXXFLAGS" CGO_LDFLAGS="$LDFLAGS"
  export CFLAGS+=' -w'
  export CXXFLAGS+=' -w'

  local goflags="-buildmode=pie -trimpath -mod=readonly -modcacherw"
  local ldflags="-linkmode=external -buildid= -X github.com/ollama/ollama/version.Version=${pkgver}"

  # Ollama with CPU only support
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
  export ROCM_PATH=/opt/rocm
  export CUDA_LIB_DIR=/disabled
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
  elif [ $(grep -c CUDA "$ollama_rocm_exe") -gt $(grep -c CUDA "$ollama_cuda_exe") ]; then
      echo "The number of 'CUDA' occurrences in ollama-rocm is greater than in ollama-cuda."
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

  install -Dm755 $pkgname/$pkgbase "$pkgdir/usr/bin/$pkgbase"
  install -dm755 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama-rocm.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
