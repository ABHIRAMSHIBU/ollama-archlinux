# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Steven Allen <steven@stebalien.com>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgbase=ollama
pkgname=(ollama ollama-docs)
pkgver=0.3.14
pkgrel=1
pkgdesc='Create, run and share large language models (LLMs)'
arch=(x86_64)
url='https://github.com/ollama/ollama'
_llama_cpp_commit=$(curl -sL "https://github.com/ollama/ollama/tree/v$pkgver/llm" | tr ' ' '\n' | tr '"' '\n' | grep ggerganov | cut -d/ -f5 | head -1)
license=(MIT)
makedepends=(cmake git go)
source=(git+$url#tag=v$pkgver
        llama.cpp::git+https://github.com/ggerganov/llama.cpp#commit=$_llama_cpp_commit
        ollama.service
        sysusers.conf
        tmpfiles.d)
b2sums=('de02d51ef1b670526283496f381e4187c99b9dc0eda78093a91cfceb0fcca284f86358f9a4f92a334d0713b1e77be9c0cb894f774574e56fe013a8c14f2490d0'
        '9f0572875d76be6665cbc1b72e385f7549c9919ffd5bda668970d6de3c6ea3099f329cbc166a9cb2731ebc06b71dc58055b40b8e07ff74c5be46c9a90456cdc1'
        '031e0809a7f564de87017401c83956d43ac29bd0e988b250585af728b952a27d139b3cad0ab1e43750e2cd3b617287d3b81efc4a70ddd61709127f68bd15eabd'
        '3aabf135c4f18e1ad745ae8800db782b25b15305dfeaaa031b4501408ab7e7d01f66e8ebb5be59fc813cfbff6788d08d2e48dcf24ecc480a40ec9db8dbce9fec'
        'e8f2b19e2474f30a4f984b45787950012668bf0acb5ad1ebb25cd9776925ab4a6aa927f8131ed53e35b1c71b32c504c700fe5b5145ecd25c7a8284373bb951ed')

prepare() {
  echo "Using llama.cpp git submodule commit $_llama_cpp_commit"

  # Prepare the git submodule by copying in files (the build process is sensitive to symlinks)
  rm -frv $pkgbase/llm/llama.cpp
  cp -r llama.cpp $pkgbase/llm/llama.cpp

  # Set the CMake build type to "Release"
  sed -i 's,T_CODE=on,T_CODE=on -D CMAKE_BUILD_TYPE=Release,g' $pkgbase/llm/generate/gen_linux.sh
}

build() {
  export CFLAGS+=' -w'
  export CXXFLAGS+=' -w'
  export LDFLAGS+=' -L/opt/cuda/targets/x86_64-linux/lib/stubs/'
  export CGO_CFLAGS="$CFLAGS" CGO_CPPFLAGS="$CPPFLAGS" CGO_CXXFLAGS="$CXXFLAGS" CGO_LDFLAGS="$LDFLAGS"

  local goflags='-buildmode=pie -trimpath -mod=readonly -modcacherw'
  local ldflags="-linkmode=external -buildid= -X github.com/ollama/ollama/version.Version=$pkgver -X github.com/ollama/ollama/server.mode=release"

  # Ollama with CPU only support
  export ROCM_PATH=/disabled
  export CUDA_LIB_DIR=/disabled
  cd $pkgbase
  go generate ./...
  go build $goflags -ldflags="$ldflags"
}

check() {
  $pkgbase/$pkgbase --version > /dev/null
  cd $pkgbase
  go test .
}

package_ollama() {
  install -Dm755 $pkgname/$pkgbase "$pkgdir/usr/bin/$pkgbase"
  install -dm755 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
  install -Dm644 $pkgbase/llm/llama.cpp/LICENSE "$pkgdir/usr/share/licenses/$pkgname/llama.cpp-LICENSE"

  ln -s /var/lib/ollama "$pkgdir/usr/share/ollama"
}

package_ollama-docs() {
  pkgdesc='Documentation for Ollama'
  arch=(any)

  install -d "$pkgdir/usr/share/doc"
  cp -r $pkgbase/docs "$pkgdir/usr/share/doc/$pkgbase"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
