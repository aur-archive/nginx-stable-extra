# Maintainer: Augusto Elesbão <aelesbao@gmail.com>

pkgname=nginx-stable-extra
_pkgname=nginx
pkgver=1.4.1
pkgrel=1
pkgdesc="lightweight HTTP server - stable release with debug enabled and easy external modules administration"
url="http://nginx.org"
license=('custom')
arch=('i686' 'x86_64')
provides=('nginx')
conflicts=('nginx' 'nginx-custom')
depends=('pcre' 'geoip' 'gd' 'libxslt' 'openssl' 'zlib')
makedepends=('geoip' 'passenger' 'gperftools')
optdepends=('passenger')
options=('!emptydirs')
backup=('etc/nginx/fastcgi.conf'
        'etc/nginx/fastcgi_params'
        'etc/nginx/koi-win'
        'etc/nginx/koi-utf'
        'etc/nginx/mime.types'
        'etc/nginx/nginx.conf'
        'etc/nginx/scgi_params'
        'etc/nginx/uwsgi_params'
        'etc/nginx/win-utf'
        'etc/logrotate.d/nginx')
source=(http://nginx.org/download/$_pkgname-$pkgver.tar.gz
        nginx
        nginx.service
        nginx.logrotate)
md5sums=('fea7dfab995545ce27fe4c49dc21a972'
         'f62c7c9b5a53471d4666a4c49ad363fb'
         '62d494d23aef31d0b867161f9fffa6eb'
         'b38744739022876554a0444d92e6603b')

_cfgdir=/etc/nginx
_logdir=/var/log/nginx
_tmpdir=/var/tmp/nginx

_configure_params=

add_module() {
  local module=$1 && shift
  local src=$1 && shift

  echo "* $module"

  if [[ "$src" == "github" ]]; then
    src=$(add_github_module $module $@)
  elif [[ "$src" =~ ^(https?|ftp):// ]]; then
    src=$(add_external_module $module $src)
  fi

  [ -d $src ] && _configure_params+=" --add-module=$src"
}

add_external_module() {
  local module=$1
  local src=$2

  [ -d $module ] || curl --silent $src | tar -xz

  echo "$(cd "$module" && pwd)"
}

add_github_module() {
  local module=$1
  local github_user=$2
  local branch=$3

  [ -d ./$module/.git ] || git clone -q git://github.com/$github_user/$module.git
  [ -n "$branch" ] && (cd $module && git checkout -q $branch)

  echo "$(cd "$module" && pwd)"
}

build() {
  _configure_params+=" --prefix=$_cfgdir"
  _configure_params+=" --conf-path=$_cfgdir/nginx.conf"
  _configure_params+=" --sbin-path=/usr/sbin/nginx"
  _configure_params+=" --pid-path=/var/run/nginx.pid"
  _configure_params+=" --lock-path=/var/lock/nginx.lock"
  _configure_params+=" --user=http --group=http"
  _configure_params+=" --http-log-path=$_logdir/access.log"
  _configure_params+=" --error-log-path=$_logdir/error.log"
  _configure_params+=" --http-client-body-temp-path=$_tmpdir/client-body"
  _configure_params+=" --http-proxy-temp-path=$_tmpdir/proxy"
  _configure_params+=" --http-fastcgi-temp-path=$_tmpdir/fastcgi"
  _configure_params+=" --http-scgi-temp-path=$_tmpdir/scgi"
  _configure_params+=" --http-uwsgi-temp-path=$_tmpdir/uwsgi"
  _configure_params+=" --with-ipv6"
  _configure_params+=" --with-pcre-jit"
  _configure_params+=" --with-file-aio"
  _configure_params+=" --with-http_dav_module"
  _configure_params+=" --with-http_image_filter_module"
  _configure_params+=" --with-http_gzip_static_module"
  _configure_params+=" --with-http_stub_status_module"
  _configure_params+=" --with-http_realip_module"
  _configure_params+=" --with-http_ssl_module"
  _configure_params+=" --with-http_sub_module"
  _configure_params+=" --with-http_xslt_module"
  _configure_params+=" --with-http_geoip_module"
  _configure_params+=" --with-google_perftools_module"
  _configure_params+=" --with-debug"
  _configure_params+=" --with-cc-opt='-Wno-error'"
  #_configure_params+=" --with-imap --with-imap_ssl_module"
  #_configure_params+=" --with-http_mp4_module"
  #_configure_params+=" --with-http_realip_module"
  #_configure_params+=" --with-http_addition_module"
  #_configure_params+=" --with-http_flv_module"
  #_configure_params+=" --with-http_random_index_module"
  #_configure_params+=" --with-http_secure_link_module"
  #_configure_params+=" --with-http_degradation_module"
  #_configure_params+=" --with-http_perl_module"

  _modulesdir="$srcdir/modules"

  echo "Configuring modules"

  mkdir -p $_modulesdir
  cd $_modulesdir

  add_module "passenger" "/usr/lib/passenger/ext/nginx"
  add_module "nginx_http_push_module-0.692" "http://pushmodule.slact.net/downloads/nginx_http_push_module-0.692.tar.gz"

  cd "$srcdir/$_pkgname-$pkgver"

  ./configure $_configure_params
  make
}

package() {
  cd "$srcdir/nginx-${pkgver}"
  make DESTDIR="$pkgdir" install

  install -d "$pkgdir"/etc/logrotate.d
  install -m644 $srcdir/nginx.logrotate $pkgdir/etc/logrotate.d/nginx

  sed -e 's|\<user\s\+\w\+;|user html;|g' \
      -e '44s|html|/usr/share/nginx/html|' \
      -e '54s|html|/usr/share/nginx/html|' \
      -i $pkgdir/etc/nginx/nginx.conf

  install -d $pkgdir/$_logdir
  install -d $pkgdir/$_tmpdir

  install -d $pkgdir/usr/share/nginx
  mv $pkgdir/etc/nginx/html/ $pkgdir/usr/share/nginx

  install -D -m755 $srcdir/nginx $pkgdir/etc/rc.d/nginx
  install -D -m644 $srcdir/nginx.service $pkgdir/usr/lib/systemd/system/nginx.service
  install -D -m644 LICENSE $pkgdir/usr/share/licenses/nginx/LICENSE

  rm -rf $pkgdir/var/run
}
