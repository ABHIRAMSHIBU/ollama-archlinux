# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Maintainer: Sven-Hendrik Haase <svenstaro@archlinux.org>
# Contributor: Steven Allen <steven@stebalien.com>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgbase=ollama
pkgname=(ollama ollama-rocm ollama-cuda ollama-docs)
pkgver=0.4.7
pkgrel=1
pkgdesc='Create, run and share large language models (LLMs)'
arch=(x86_64)
url='https://github.com/ollama/ollama'
license=(MIT)
makedepends=(cmake git go hipblas cuda clblast)
source=(git+https://github.com/ollama/ollama#tag=v$pkgver
        ollama-7499.patch::https://github.com/ollama/ollama/pull/7499.patch
        ollama-ld.conf
        ollama.service
        sysusers.conf
        tmpfiles.d)
b2sums=('66267f66e3ab1c2e6263e5608d6bb170be73a0930001ce6d6969c45ec3f3d23b3ccca41afb5b2d80ea3997301cb5ad8cbfe2beba65874dd47a537c333daacd84'
        '974b803ceaf31faca29da05d146f1bd3a7fabaf43fb62dd4e981a93da9e425af46d54e6b3e12f75c66715c7d6b53aadc2ffe728eda162d9c32a132c74ebc0527'
        '121a7854b5a7ffb60226aaf22eed1f56311ab7d0a5630579525211d5c096040edbcfd2608169a4b6d83e8b4e4855dbb22f8ebf3d52de78a34ea3d4631b7eff36'
        '031e0809a7f564de87017401c83956d43ac29bd0e988b250585af728b952a27d139b3cad0ab1e43750e2cd3b617287d3b81efc4a70ddd61709127f68bd15eabd'
        '3aabf135c4f18e1ad745ae8800db782b25b15305dfeaaa031b4501408ab7e7d01f66e8ebb5be59fc813cfbff6788d08d2e48dcf24ecc480a40ec9db8dbce9fec'
        'e8f2b19e2474f30a4f984b45787950012668bf0acb5ad1ebb25cd9776925ab4a6aa927f8131ed53e35b1c71b32c504c700fe5b5145ecd25c7a8284373bb951ed')

prepare() {
  cd ollama
  patch -Np1 -i "$srcdir"/ollama-7499.patch
}

build() {
  export CGO_CPPFLAGS="${CPPFLAGS}"
  export CGO_CFLAGS="${CFLAGS}"
  export CGO_CXXFLAGS="${CXXFLAGS}"
  export CGO_LDFLAGS="${LDFLAGS}"
  export GOPATH="${srcdir}"
  export GOFLAGS="-buildmode=pie -mod=readonly -modcacherw '-ldflags=-linkmode=external -compressdwarf=false -X=github.com/ollama/ollama/version.Version=$pkgver -X=github.com/ollama/ollama/server.mode=release'"

  cd ollama

  # Unset these otherwise somehow nvcc will try to use them.
  unset CFLAGS CXXFLAGS
  make dist CUDA_12_PATH=/opt/cuda
  go build .
}

check() {
  $pkgbase/$pkgbase --version > /dev/null
  cd $pkgbase
  go test .
}

package_ollama() {
  install -Dm755 ollama/ollama "$pkgdir/usr/bin/ollama"
  mkdir -p "$pkgdir"/usr/lib/ollama/runners
  cp -r ollama/dist/linux-amd64/lib/ollama/runners/cpu* "$pkgdir"/usr/lib/ollama/runners/

  install -Dm755 $pkgname/$pkgbase "$pkgdir/usr/bin/$pkgbase"
  install -dm755 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

  ln -s /var/lib/ollama "$pkgdir/usr/share/ollama"
}

package_ollama-rocm() {
  pkgdesc='Create, run and share large language models (LLMs) with ROCm'
  depends+=(ollama hipblas)

  mkdir -p "$pkgdir"/usr/lib/ollama/runners
  cp -r ollama/dist/linux-amd64/lib/ollama/runners/rocm* "$pkgdir"/usr/lib/ollama/runners
}

package_ollama-cuda() {
  pkgdesc='Create, run and share large language models (LLMs) with CUDA'
  depends+=(ollama cuda)

  mkdir -p "$pkgdir"/usr/lib/ollama/runners
  cp -r ollama/dist/linux-amd64/lib/ollama/runners/cuda* "$pkgdir"/usr/lib/ollama/runners
}

package_ollama-docs() {
  pkgdesc='Documentation for Ollama'

  install -d "$pkgdir/usr/share/doc"
  cp -r $pkgbase/docs "$pkgdir/usr/share/doc/$pkgbase"
  install -Dm644 $pkgbase/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
