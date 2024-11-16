# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Steven Allen <steven@stebalien.com>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgbase=ollama
pkgname=(ollama ollama-docs)
pkgver=0.4.2
pkgrel=1
pkgdesc='Create, run and share large language models (LLMs)'
arch=(x86_64)
url='https://github.com/ollama/ollama'
license=(MIT)
makedepends=(cmake git go)
source=(git+$url#tag=v$pkgver
        ollama.service
        sysusers.conf
        tmpfiles.d)
b2sums=('02154afa3129c64d2dc3365a49860748271003c25441fd663a1abfc6e7ef3b324a0057625ceb47812466dc8616896d670bbc8462937a5fe7a6b5845e03a9e2f6'
        '031e0809a7f564de87017401c83956d43ac29bd0e988b250585af728b952a27d139b3cad0ab1e43750e2cd3b617287d3b81efc4a70ddd61709127f68bd15eabd'
        '3aabf135c4f18e1ad745ae8800db782b25b15305dfeaaa031b4501408ab7e7d01f66e8ebb5be59fc813cfbff6788d08d2e48dcf24ecc480a40ec9db8dbce9fec'
        'e8f2b19e2474f30a4f984b45787950012668bf0acb5ad1ebb25cd9776925ab4a6aa927f8131ed53e35b1c71b32c504c700fe5b5145ecd25c7a8284373bb951ed')

build() {
  export CFLAGS+=' -w' CXXFLAGS+=' -w'
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

  ln -s /var/lib/ollama "$pkgdir/usr/share/ollama"
}

package_ollama-docs() {
  pkgdesc='Documentation for Ollama'

  install -d "$pkgdir/usr/share/doc"
  cp -r $pkgbase/docs "$pkgdir/usr/share/doc/$pkgbase"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
