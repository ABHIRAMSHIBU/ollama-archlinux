# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgname=ollama
pkgdesc='Create, run and share large language models (LLMs)'
pkgver=0.1.30
pkgrel=1
arch=(x86_64)
url='https://github.com/ollama/ollama'
license=(MIT)
_ollamacommit=756c2575535641f1b96d94b4214941b90f4c30c7 # tag: v0.1.30
# The llama.cpp git submodule commit hash can be found here:
# https://github.com/ollama/ollama/tree/v0.1.30/llm
_llama_cpp_commit=ad3a0505e3b6cd777259ee35e61d428357ffc565
makedepends=(cmake git go)
source=(git+$url#commit=$_ollamacommit
        llama.cpp::git+https://github.com/ggerganov/llama.cpp#commit=$_llama_cpp_commit
        ollama.service
        sysusers.conf
        tmpfiles.d)
b2sums=('b2b108add6c3806bb6084b7f0e78c6a19b9f0b518079ae5f7b416b10fac292d740df5f26f3d1aec7d106900ff07af940ade35891a3c6e9453eb26eb7e9a6e373'
        'eaa62efe3549124d055cad5761343c06f8059db93445e639c4e072d234a998691999145116f33d86213a380a946bdef8a05167f168be3a47427ca1556c8e7634'
        'a773bbf16cf5ccc2ee505ad77c3f9275346ddf412be283cfeaee7c2e4c41b8637a31aaff8766ed769524ebddc0c03cf924724452639b62208e578d98b9176124'
        '3aabf135c4f18e1ad745ae8800db782b25b15305dfeaaa031b4501408ab7e7d01f66e8ebb5be59fc813cfbff6788d08d2e48dcf24ecc480a40ec9db8dbce9fec'
        'e8f2b19e2474f30a4f984b45787950012668bf0acb5ad1ebb25cd9776925ab4a6aa927f8131ed53e35b1c71b32c504c700fe5b5145ecd25c7a8284373bb951ed')

prepare() {
  cd $pkgname

  rm -frv llm/llama.cpp

  # Copy git submodule files instead of symlinking because the build process is sensitive to symlinks
  cp -r "$srcdir/llama.cpp" llm/llama.cpp

  # Turn LTO on and set the build type to Release
  sed -i 's,T_CODE=on,T_CODE=on -D LLAMA_LTO=on -D CMAKE_BUILD_TYPE=Release,g' llm/generate/gen_linux.sh

  # Display a more helpful error message
  sed -i "s|could not connect to ollama server, run 'ollama serve' to start it|ollama is not running, try 'systemctl start ollama'|g" cmd/cmd.go
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
