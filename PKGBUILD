# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgname=ollama
pkgdesc='Create, run and share large language models (LLMs)'
pkgver=0.1.32
pkgrel=1
arch=(x86_64)
url='https://github.com/ollama/ollama'
license=(MIT)
_ollamacommit=fb9580df85c562295d919b6c2632117d3d8cea89 # tag: v0.1.32
# The llama.cpp git submodule commit hash can be found here:
# https://github.com/ollama/ollama/tree/v0.1.32/llm
_llama_cpp_commit=7593639ce335e8d7f89aa9a54d616951f273af60
makedepends=(cmake git go)
source=(git+$url#commit=$_ollamacommit
        llama.cpp::git+https://github.com/ggerganov/llama.cpp#commit=$_llama_cpp_commit
        ollama.service
        sysusers.conf
        tmpfiles.d)
b2sums=('12b952df20a194e7b41040574b449aa9781dc40e9316d6d459caf0228c51c588d93a79223fe4009ca6532dffb9846853a6e74769db15c25135032bf5ccfde65e'
        'de8f50fb5a99a0251f6a0f3d596975cf43ad76770ac2edeaac6497b81214a7f31574b499f10b77f0542f9d46c5d33e0f5ac1e49fa72343d860c70fd3ca0ca113'
        '2bf4c2076b7841de266ec40da2e2cbb675dcbfebfa8aed8d4ede65435854cb43d39ea32bc9210cfc28a042382dd0094a153e351edfa5586eb7c6a0783f3bc517'
        '3aabf135c4f18e1ad745ae8800db782b25b15305dfeaaa031b4501408ab7e7d01f66e8ebb5be59fc813cfbff6788d08d2e48dcf24ecc480a40ec9db8dbce9fec'
        'e8f2b19e2474f30a4f984b45787950012668bf0acb5ad1ebb25cd9776925ab4a6aa927f8131ed53e35b1c71b32c504c700fe5b5145ecd25c7a8284373bb951ed')

prepare() {
  cd $pkgname

  rm -frv llm/llama.cpp

  # Copy git submodule files instead of symlinking because the build process is sensitive to symlinks
  cp -r "$srcdir/llama.cpp" llm/llama.cpp

  # Turn LTO on and set the build type to Release
  sed -i 's,T_CODE=on,T_CODE=on -D LLAMA_LTO=on -D CMAKE_BUILD_TYPE=Release,g' llm/generate/gen_linux.sh
}

build() {
  cd $pkgname
  export CGO_CFLAGS="$CFLAGS" CGO_CPPFLAGS="$CPPFLAGS" CGO_CXXFLAGS="$CXXFLAGS" CGO_LDFLAGS="$LDFLAGS"
  go generate ./...
  go build -buildmode=pie -trimpath -mod=readonly -modcacherw -ldflags=-linkmode=external \
    -ldflags=-buildid='' -ldflags="-X=github.com/ollama/ollama/version.Version=$pkgver"
}


check() {
  cd $pkgname
  ./ollama --version >/dev/null
  go test .
}

package() {
  install -Dm755 $pkgname/$pkgname "$pkgdir/usr/bin/$pkgname"
  install -dm755 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 $pkgname/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
