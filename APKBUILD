# Maintainer: Jakub Jirutka <jakub@jirutka.cz>
# Contributor: Jeff Bilyk <jbilyk@gmail.com>
# Contributor: Bartłomiej Piotrowski <nospam@bpiotrowski.pl>
# Contributor: Jakub Jirutka <jakub@jirutka.cz>
#
# secfixes:
#   1.22.1-r0:
#     - CVE-2022-41741
#     - CVE-2022-41742
#   1.20.2-r2:
#     - CVE-2021-46461
#     - CVE-2021-46462
#     - CVE-2021-46463
#     - CVE-2022-25139
#   1.20.1-r1:
#     - CVE-2021-3618
#   1.20.1-r0:
#     - CVE-2021-23017
#   1.16.1-r6:
#     - CVE-2019-20372
#   1.16.1-r0:
#     - CVE-2019-9511
#     - CVE-2019-9513
#     - CVE-2019-9516
#   1.14.1-r0:
#     - CVE-2018-16843
#     - CVE-2018-16844
#     - CVE-2018-16845
#   1.12.1-r0:
#     - CVE-2017-7529
#
pkgname=nginx
# NOTE: Upgrade only to even-numbered versions (e.g. 1.14.z, 1.16.z)!
# Odd-numbered versions are mainline (development) versions.
pkgver=1.23.3
pkgrel=0
# Revision of nginx-tests to use for check().
_nginxver=8771d35d55d0
_librever=3.7.1
_tests_hgrev=1e1d0f3874b0
_njs_ver=0.7.11
pkgdesc="HTTP and reverse proxy server (quic version)"
url="https://www.nginx.org/"
arch="all"
license="BSD-2-Clause"
depends=""
makedepends="
	brotli-dev
	gd-dev
	geoip-dev
	libmaxminddb-dev
	libxml2-dev
	libxslt-dev
	linux-headers
	pcre2-dev
	perl-dev
	pkgconf
	zeromq-dev
	zlib-dev
	autoconf
	automake
	libtool
	"
checkdepends="
	gd
	perl
	perl-fcgi
	perl-io-socket-ssl
	perl-net-ssleay
	perl-protocol-websocket
	tzdata
	uwsgi-python3
	"
pkgusers="nginx"
_grp_ngx="nginx"
_grp_www="www-data"
pkggroups="$_grp_ngx $_grp_www"
install="$pkgname.pre-install $pkgname.post-install $pkgname.pre-upgrade $pkgname.post-upgrade"
# options="!check"
builddir="$srcdir"/$pkgname-quic
subpackages="$pkgname-debug $pkgname-doc $pkgname-openrc $pkgname-vim::noarch"
source="https://nginx.org/download/nginx-$pkgver.tar.gz
	https://hg.nginx.org/nginx-quic/archive/tip.tar.gz
	https://github.com/libressl/portable/archive/refs/tags/v$_librever.tar.gz
	https://hg.nginx.org/nginx-tests-quic/archive/quic.tar.gz
	$pkgname-njs-$_njs_ver.tar.gz::https://hg.nginx.org/njs/archive/$_njs_ver.tar.gz
	nginx-dav-ext-module~pr-56.patch::https://github.com/arut/nginx-dav-ext-module/pull/56.patch
	nginx-dav-ext-module~pr-62.patch::https://github.com/arut/nginx-dav-ext-module/commit/bbf93f75ca58657fb0f8376b0898f854f13cef91.patch
	nginx-upload-progress-module~nginx_1.23.0.patch::https://github.com/masterzen/nginx-upload-progress-module/files/8980323/nginx_1.23.0.patch.txt
	mod_zip~00-660ad95.patch::https://github.com/evanmiller/mod_zip/commit/660ad956a960703558b9368a2474e85ab7a7d1df.patch
	mod_zip~01-bff10ba.patch::https://github.com/evanmiller/mod_zip/commit/bff10ba7314c201dec2ed8cbcf75d585ba4d433e.patch
	mod_zip~02-51cf45d.patch::https://github.com/evanmiller/mod_zip/commit/51cf45d3e9f51e02224af017b235d1d30fbf28fb.patch
	mod_zip~03-555d3b3.patch::https://github.com/evanmiller/mod_zip/commit/555d3b3671dbd896e8e107dea773d452e3334c65.patch
	mod_zip~04-808fb55.patch::https://github.com/evanmiller/mod_zip/commit/808fb55e7235a201ea862e02dab612b87787d5a4.patch
	mod_zip~05-5b2604b.patch::https://github.com/evanmiller/mod_zip/commit/5b2604b3914f87db2077f2239b8a98b66cf622af.patch
	traffic-accounting-nginx-module~disable-stream-module.patch
	nginx_cookie_flag_module~fix-mem-allocations.patch
	njs~mktemp-busybox-compat.patch
	njs~nginx-1.20.x-compat.patch
	nginx.conf
	default.conf
	stream.conf
	$pkgname.logrotate
	$pkgname.initd
	$pkgname.confd
	"

_modules_dir="usr/lib/$pkgname/modules"
_stream_js_depends="$pkgname-mod-stream"

case "$CARCH" in
	x86) _njs_mods= ;; # has failing tests
	*) _njs_mods="http-js stream-js";;
esac

# Built-in dynamic modules
for _mod in \
	http-geoip \
	http-image-filter \
	http-perl \
	http-xslt-filter \
	mail \
	stream \
	stream-geoip \
	$_njs_mods
do
	subpackages="$subpackages $pkgname-mod-$_mod:_module"
done

# Third-party dynamic modules

# For simplicity we assume that module is hosted on GitHub.
_add_module() {
	local name="$1" ver="$2" url="$3" subdir="$4" enabled="${5:-true}"
	local dirname=${url##*/}-${ver#v}
	local varprefix="_${name//-/_}"

	eval "${varprefix}_ver='$ver'; ${varprefix}_url='$url'"

	# Don't add new flag and source if it's already there, i.e. two or more
	# modules share the same source (e.g. geoip2 that provides http-geoip2
	# and stream-geoip2).
	local present=false
	case "$_extra_flags" in
		*="$srcdir/$dirname"*) present=true;;
	esac

	if ! $present; then
		source="$source $dirname.tar.gz::$url/archive/$ver.tar.gz"
		# $source must be always in-sync with $sha512sums, so we have to
		# add there source of a module that is disabled on the current arch.
		[ "$enabled" = false ] && return

		_extra_flags="$_extra_flags --add-dynamic-module=$srcdir/$dirname/$subdir"
	fi
	subpackages="$subpackages $pkgname-mod-$name:_module"
}

_add_module "devel-kit" "v0.3.1" "https://github.com/vision5/ngx_devel_kit"
_devel_kit_so="ndk_http_module.so"

_add_module "http-accounting" "v2.0" "https://github.com/Lax/traffic-accounting-nginx-module"

_add_module "http-array-var" "v0.05" "https://github.com/openresty/array-var-nginx-module"
_http_array_var_depends="$pkgname-mod-devel-kit"

_add_module "http-brotli" "v1.0.0rc" "https://github.com/google/ngx_brotli"
_http_brotli_so="ngx_http_brotli_filter_module.so ngx_http_brotli_static_module.so"

_add_module "http-cache-purge" "2.5.2" "https://github.com/nginx-modules/ngx_cache_purge"

_add_module "http-cookie-flag" "v1.1.0" "https://github.com/AirisX/nginx_cookie_flag_module"
_http_cookie_flag_so="ngx_http_cookie_flag_filter_module.so"

_add_module "http-dav-ext" "v3.0.0" "https://github.com/arut/nginx-dav-ext-module"

_add_module "http-echo" "v0.63" "https://github.com/openresty/echo-nginx-module"

_add_module "http-encrypted-session" "v0.09" "https://github.com/openresty/encrypted-session-nginx-module"
_http_encrypted_session_depends="$pkgname-mod-devel-kit"

_add_module "http-fancyindex" "v0.5.2" "https://github.com/aperezdc/ngx-fancyindex"

_add_module "http-geoip2" "3.4" "https://github.com/leev/ngx_http_geoip2_module"
_add_module "stream-geoip2" "3.4" "https://github.com/leev/ngx_http_geoip2_module"
_stream_geoip2_depends="$pkgname-mod-stream"

_add_module "http-headers-more" "v0.34" "https://github.com/openresty/headers-more-nginx-module"
_http_headers_more_so="ngx_http_headers_more_filter_module.so"

_add_module "http-log-zmq" "v1.0.0" "https://github.com/danifbento/nginx-log-zmq"

# luajit is required for lua-nginx-module since v0.10.14
# _add_module "http-lua" "v0.10.22" "https://github.com/openresty/lua-nginx-module" "" "$_has_luajit"
# _http_lua_depends="$pkgname-mod-devel-kit lua-resty-core"
# _http_lua_provides="$pkgname-lua"  # for backward compatibility

# _add_module "http-lua-upstream" "v0.07" "https://github.com/openresty/lua-upstream-nginx-module" "" "$_has_luajit"
# _http_lua_upstream_depends="$pkgname-mod-http-lua"

# _add_module "http-naxsi" "1.3" "https://github.com/nbs-system/naxsi" "naxsi_src"
# _naxsi_provides="$pkgname-naxsi"  # for backward compatibility

_add_module "http-nchan" "v1.3.4" "https://github.com/slact/nchan"
_http_nchan_so="ngx_nchan_module.so"

_add_module "http-redis2" "v0.15" "https://github.com/openresty/redis2-nginx-module"

_add_module "http-set-misc" "v0.33" "https://github.com/openresty/set-misc-nginx-module"
_http_set_misc_depends="$pkgname-mod-devel-kit"

_add_module "http-shibboleth" "v2.0.1" "https://github.com/nginx-shib/nginx-http-shibboleth"

_add_module "http-untar" "v1.1" "https://github.com/ajax16384/ngx_http_untar_module"

_add_module "http-upload" "2.3.0" "https://github.com/fdintino/nginx-upload-module"

_add_module "http-upload-progress" "v0.9.2" "https://github.com/masterzen/nginx-upload-progress-module"
_http_upload_progress_so="ngx_http_uploadprogress_module.so"

_add_module "http-upstream-fair" "0.1.3" "https://github.com/itoffshore/nginx-upstream-fair"

_add_module "http-upstream-jdomain" "1.4.0" "https://github.com/nicholaschiasson/ngx_upstream_jdomain"

_add_module "http-vod" "1.30" "https://github.com/kaltura/nginx-vod-module"

_add_module "http-vts" "v0.2.1" "https://github.com/vozlt/nginx-module-vts"
_http_vts_so="ngx_http_vhost_traffic_status_module.so"

_add_module "http-zip" "1.2.0" "https://github.com/evanmiller/mod_zip"

_add_module "rtmp" "v1.2.2" "https://github.com/arut/nginx-rtmp-module"
_rtmp_provides="$pkgname-rtmp"  # for backward compatibility


prepare() {
	#git clone -b v3.6.1 https://github.com/libressl-portable/portable.git "$srcdir"/libressl
	mv -f "$srcdir"/portable-$_librever "$srcdir"/libressl
	cd "$srcdir"/libressl
	./autogen.sh
	#hg clone --cwd "$srcdir" -b quic https://hg.nginx.org/nginx-quic
	mv -f "$srcdir"/$pkgname-quic-quic "$srcdir"/$pkgname-quic
	cp "$srcdir"/$pkgname-$pkgver/LICENSE "$builddir"/
	
	local file; for file in $source; do
		file=${file%%::*}

		case $file in
		*~*.patch)
			msg $file
			cd "$srcdir"/${file%%~*}-*
			patch -p 1 -i "$srcdir/$file"
			;;
		*.patch)
			msg $file
			cd "$builddir"
			patch -p 1 -i "$srcdir/$file"
			;;
		esac
	done

	# This test requires superuser privileges and CAP_NET_ADMIN.
	#rm "$srcdir"/nginx-tests-*/proxy_bind_transparent.t
	#rm "$srcdir"/nginx-tests-*/proxy_bind_transparent_capability.t
	# Travis and Drone.io does not support IPv6...
	rm -f "$srcdir"/nginx-tests-*/h2_headers.t
}

_build() {
	# --without-pcre2 - Lua module is not compatible with PCRE2 yet
	#   https://github.com/openresty/lua-nginx-module/issues/1984
	auto/configure \
		--prefix=/var/lib/$pkgname \
		--sbin-path=/usr/sbin/$pkgname \
		--modules-path=/$_modules_dir \
		--conf-path=/etc/$pkgname/$pkgname.conf \
		--pid-path=/run/$pkgname/$pkgname.pid \
		--lock-path=/run/$pkgname/$pkgname.lock \
		--http-client-body-temp-path=/var/lib/$pkgname/tmp/client_body \
		--http-proxy-temp-path=/var/lib/$pkgname/tmp/proxy \
		--http-fastcgi-temp-path=/var/lib/$pkgname/tmp/fastcgi \
		--http-uwsgi-temp-path=/var/lib/$pkgname/tmp/uwsgi \
		--http-scgi-temp-path=/var/lib/$pkgname/tmp/scgi \
		--with-perl_modules_path=/usr/lib/perl5/vendor_perl \
		\
		--user=$pkgusers \
		--group=$_grp_ngx \
		--with-threads \
		--with-file-aio \
		\
		--with-http_ssl_module \
		--with-http_v2_module \
		--with-http_v3_module \
		--with-stream_quic_module \
		--build=nginx-quic \
		--with-openssl="$srcdir"/libressl \
		\
		--with-http_realip_module \
		--with-http_addition_module \
		--with-http_xslt_module=dynamic \
		--with-http_image_filter_module=dynamic \
		--with-http_geoip_module=dynamic \
		--with-http_sub_module \
		--with-http_dav_module \
		--with-http_flv_module \
		--with-http_mp4_module \
		--with-http_gunzip_module \
		--with-http_gzip_static_module \
		--with-http_auth_request_module \
		--with-http_random_index_module \
		--with-http_secure_link_module \
		--with-http_degradation_module \
		--with-http_slice_module \
		--with-http_stub_status_module \
		--with-http_perl_module=dynamic \
		--with-mail=dynamic \
		--with-mail_ssl_module \
		--with-stream=dynamic \
		--with-stream_ssl_module \
		--with-stream_realip_module \
		--with-stream_geoip_module=dynamic \
		--with-stream_ssl_preread_module \
		\
		${_njs_mods:+"--add-dynamic-module=$srcdir/njs-$_njs_ver/nginx"} \
		$_extra_flags \
		"$@"

	make
}

build() {
	cd "$builddir"

	_build --with-debug
	mv objs objs-debug

	make clean
	_build
}

check() {
	msg "Running nginx tests..."
	cd "$srcdir"/nginx-tests-*

	case "$CARCH" in
	mips*)
		# Sporadic failure on builder.
		rm ssl.t
		;;
	esac

	TEST_NGINX_BINARY="$builddir/objs/nginx" prove .

	if [ -n "$_njs_mods" ]; then
		msg "Running njs tests..."
		cd "$srcdir"/njs-*
		make test
	fi
}

package() {
	cd "$builddir"

	make DESTDIR="$pkgdir" install
	chown root:root "$pkgdir"/usr/sbin/nginx

	install -Dm644 LICENSE "$pkgdir"/usr/share/licenses/$pkgname/LICENSE
	install -Dm644 README "$pkgdir"/usr/share/doc/$pkgname/README

	install -Dm644 objs/$pkgname.8 "$pkgdir"/usr/share/man/man8/$pkgname.8

	local name; for name in ngx_devel_kit nginx-rtmp-module; do
		cp -r "$srcdir"/$name-*/doc* "$pkgdir"/usr/share/doc/$pkgname/$name
	done

	cd "$pkgdir"

	install -Dm644 "$srcdir"/nginx.conf ./etc/$pkgname/nginx.conf
	install -dm755 ./etc/$pkgname/http.d ./etc/$pkgname/modules

	install -Dm755 "$srcdir"/$pkgname.initd ./etc/init.d/$pkgname
	install -Dm644 "$srcdir"/$pkgname.confd ./etc/conf.d/$pkgname
	install -Dm644 "$srcdir"/$pkgname.logrotate ./etc/logrotate.d/$pkgname

	install -dm750 -o $pkgusers -g $_grp_ngx ./var/lib/$pkgname
	install -dm700 -o $pkgusers -g $_grp_ngx ./var/lib/$pkgname/tmp
	install -dm755 -g $_grp_www ./var/www/localhost/htdocs

	install -Dm644 "$srcdir"/default.conf ./usr/share/$pkgname/http-default_server.conf

	install -dm755 ./var/log
	mv ./var/lib/$pkgname/logs ./var/log/$pkgname
	chown $pkgusers:$_grp_ngx var/log/$pkgname

	ln -sf /$_modules_dir ./var/lib/$pkgname/modules
	ln -sf /var/log/$pkgname ./var/lib/$pkgname/logs
	ln -sf /run/$pkgname ./var/lib/$pkgname/run

	# Remove archaic charset maps.
	rm ./etc/$pkgname/koi-* ./etc/$pkgname/win-utf

	rm -rf ./run ./etc/$pkgname/*.default
}

# http://nginx.org/en/docs/debugging_log.html
debug() {
	pkgdesc="nginx built with support for debugging log"
	depends="$pkgname=$pkgver-r$pkgrel"
	options="!strip"

	install -Dm755 "$builddir"/objs-debug/nginx "$subpkgdir"/usr/sbin/nginx-debug
}

vim() {
	pkgdesc="$pkgdesc (vim syntax)"
	install_if="vim $pkgname=$pkgver-r$pkgrel"
	depends=

	mkdir -p "$subpkgdir"/usr/share/vim
	cp -r "$builddir"/contrib/vim "$subpkgdir"/usr/share/vim/vimfiles
}

_module() {
	local name="${subpkgname#$pkgname-mod-}"; name="${name//-/_}"
	local ver=$(getvar _${name}_ver)

	pkgdesc="Nginx module ${name//_/-}"
	[ "$ver" ] && pkgdesc="Nginx third-party module ${name//_/-} (version $ver)"

	url=$(getvar "_${name}_url" "$url")
	sonames=$(getvar "_${name}_so" "ngx_${name}_module.so")
	depends="$pkgname $(getvar "_${name}_depends")"
	provides=$(getvar "_${name}_provides")

	# Numeric prefix for the module config to ensure that modules with
	# dependencies on other modules will be loaded after their dependencies.
	# For simplicity, we don't actually resolve dependency tree. Instead,
	# we just prefix the module name with a number that reflects number of
	# the module's dependencies times ten (e.g. 10, 20, 30, ...).
	local conf_prefix="$(echo "$depends" | wc -w)0_"

	mkdir -p "$subpkgdir"/$_modules_dir
	mkdir -p "$subpkgdir"/etc/nginx/modules

	cd "$subpkgdir"

	local soname; for soname in $sonames; do
		mv "$pkgdir"/$_modules_dir/$soname ./$_modules_dir/$soname
		echo "load_module \"modules/$soname\";" >> ./etc/nginx/modules/${conf_prefix}$name.conf
	done

	case "$name" in
		http_perl)
			mv "$pkgdir"/usr/lib/perl5 "$subpkgdir"/usr/lib/
		;;
		http_naxsi)
			install -m644 -D "$srcdir"/naxsi-*/naxsi_config/naxsi_core.rules \
				./etc/nginx/naxsi_core.rules
		;;
		stream)
			mkdir -p ./etc/nginx/stream.d
			install -m644 -D "$srcdir"/stream.conf ./etc/nginx/conf.d/stream.conf
		;;
	esac
}

# Print value of the specified variable, or the default if empty or not defined.
getvar() {
	eval "printf '%s\n' \"\${$1:-$2}\""
}

sha512sums="
4a5413c0ec251c02fb73dfb4d351045f857a36d45ebb7ae2c29f4a4f320a6543d0a049b147b08318de0b7b0406773c329dbf43bf98bb088f76e506ea532cd8ef  nginx-1.23.2.tar.gz
18127a70ccd2e5c0d89bcfa8647db1a787a32d1e21cdfae9673cc455f94c329d22c23fb3b5f0557a8ce0472ace0c0d903970b4c01b06f316a4169c4d3a20b032  nginx-tests-1d88487eafbf.tar.gz
d76d654e1a73dbbbf5ad4ab6ae32bb43f5ea502e2468f9f0fe9485eb78c0ee6d550c4032e3a771ac5888d2ec7c387fd4a1fefb7eb47ba2e21afd4a294780df78  nginx-njs-0.7.7.tar.gz
4c7a94aaebbb69599b0067e74f9f3db54ec383ca9499292fec5b875bb0b5859aa11dc14cef5664c94dd54aba231f31e85feacddc49f7622aa4d0fdb38709b6e1  nginx-dav-ext-module~pr-56.patch
fdd66e433126e194a3ef22737993191a04fcc4c8caa044b27cb22bea0e7f16c8fdbc900553507d2bb541cdb82b542845a297db2a48c2460a38dd772d0ebfca9d  nginx-dav-ext-module~pr-62.patch
2899636d730583c0eaa21e89d50ccb7a888e7f27fa194102909e42fb28cb8e239416978f55bed0a9115b65d0ac718cb7da8c1fa589eb79e9f66eea41dfc3458b  nginx-upload-progress-module~nginx_1.23.0.patch
169373a1457519d8a844c2bca4fec0bb4e0387999a205b7cdb1f1ffd4ca5548c3ddc7de41eaa5842336a7c2a1876554e057f58ccfaf01978edcc9cd060d6bfdb  mod_zip~00-660ad95.patch
2a361f825bf3ffde2769bacbb272ae121d506afa0bc069153546c0d061879a157baedf454b6cb752bfce654b8239a8fba17b817a15382c8a53a2156697effedc  mod_zip~01-bff10ba.patch
27b075d08a644577f7efa7ceb97dc2db8f964c0e11c1193e5552fffc102c9c7810a2131a70ddb2cd3590be7a60035c76fd12f036046ce8d649569ab9ece8f2d1  mod_zip~02-51cf45d.patch
4b38807502dcf9a5f2ed748cda3688c04c0abf0beecfad326fe346836f62ee9010bc74c4ad7a136d86b26eed073eb2a5393e7bf3bbcaefd7ad14f2c461577ead  mod_zip~03-555d3b3.patch
35d98b6df1b16dc2a815025826e3adb39448b6d2821fb0d779c2af6ffa04b7badc5d8f90f7f8b24c92350e644b85a9fa930420db6267786bc7fd251f7d7df5f7  mod_zip~04-808fb55.patch
316d09265f61564b3158d713e0e0bcd0254341288ed6527cbee33b211ec18aab6fd5254a1b31f98d288db703d8a8f272b4a610ac338df7f0688aae4a250718e0  mod_zip~05-5b2604b.patch
09ec9f18323197eafa55ff68e8c836ad3dd830e6cd3bd4aeaf34e179ef3f72f734a0117288c1c58813aff59f3f1f0f29ccd772a672e17551e7a4fd0693a89c92  traffic-accounting-nginx-module~disable-stream-module.patch
ac0f912ae90e0083cc761a622290223edeed0bd32213bbe766d637ac2dfd9835d163e5d16ef28740cbad05d6d92cc418d62df3413c70b4f2c63db02f8ca1c7cc  nginx_cookie_flag_module~fix-mem-allocations.patch
4db527d663dbe9e8b503c3cbaa4eae34b45990a5359b3bb98ce970c705faefcac98de49439f2557756a2be8e2e06acc67f98942de01674c498832d80c3cb90c3  njs~mktemp-busybox-compat.patch
df1d910d5a433ef8aa6620e46bd46cb82c45c840e35420bf81a76e5a868ac73ad88aa3934d1c11bcf004a88a9cd13bf69a96ee1b08540251b09903a30430b199  njs~nginx-1.20.x-compat.patch
d4e16ef5dbd4f88b6dacdcff7cf2313a1d02a196000bc5c7bbc5b88ddfee686c0e24f926f779308589a4f3a7ad38d444c3435291296db889e6ea2dcae7d80e46  njs~fix-dangling-pointer.patch
ea6ad54b9eb84fd557a2922560ae8f0dca458b09679e5aab2676e495fbf7580b42841460c094fd28ee2e98b667cee59ab3b04c7be0b0f330fb99b492c52147a7  nginx.conf
0907f69dc2d3dc1bad3a04fb6673f741f1a8be964e22b306ef9ae2f8e736e1f5733a8884bfe54f3553fff5132a0e5336716250f54272c3fec2177d6ba16986f3  default.conf
426f0c317322af7cab152f2070398c7aa5c059276ba504617a212f1e060bbb1dd9edc54e62d4cf5f14e3678235351c808ebeabe8b122757c74b3f505e8427106  stream.conf
09b110693e3f4377349ccea3c43cb8199c8579ee351eae34283299be99fdf764b0c1bddd552e13e4d671b194501618b29c822e1ad53b34101a73a63954363dbb  nginx.logrotate
ee10a5687740dde0c3d18d8b3555f49fcdc6abfc0a3bc2de1de3be0e99951a346fe8027d916aab73071ecd4e2c50871e7c867aca3a7a0fd16e3374c5caed1c57  nginx.initd
0b9b9ed01ac077e334c034faa2679f6e26740fb3362eebf8cef82d22b2af2a3faaa53bae3c9e14af51cbf64720a7c66431905ca5cc43f978366456cc4e3b7f4a  nginx.confd
de1e3349d8dd08e5982279b2219dc8a8006739f0409b8e0f5c50d93434beff1fbafba43e9c5ac85a5fab90afc5c0a7244a340610339c36f82f2cba7233e72de9  ngx_devel_kit-0.3.1.tar.gz
0df34c3765e18dc5cc5a053d3a17dbee17a686a1f6e76ad057c262741c4e4465c66bcef86e627a19258f836cad5f14745bf046fd396b00960ad79ed20c2a07bb  traffic-accounting-nginx-module-2.0.tar.gz
7c9fa9b76bc7cd2473ceae6d5ffb8de26993be9293ea967908d6c4550e086affa7016df4c936fb0b79f1142dc0aa1a5f2058d417e6433b5a3497a45d7e866e84  array-var-nginx-module-0.05.tar.gz
05a880d5e48ac83be84498ed41fb4750211b827a9d7541acfd6ef494e5205a6e853d5594bfec3ab4ae668ea3f803e4f4b0ba550c76811971c8e266e42663c56d  ngx_brotli-1.0.0rc.tar.gz
a5e5f901823559d42421f9bc8f2aaba6cc3ad9ddfba2bbad154bb6b70e2001ed1bf25781c2117faeea3c20d824c74827cdf6b97f45eda343b60f50b964e69c40  ngx_cache_purge-2.5.2.tar.gz
352cc3d033cc67ee34209f958dac13ada2147de429f4dd3da301c865d52970d80c8aa3c193f7fb28cf4854b88baff07b6efc3bae1fb813fe53d5956a87dfc81a  nginx_cookie_flag_module-1.1.0.tar.gz
d0193ba90f1ef46c4e470630c4394bdf99d94fd2e3bd8be6cb2ba1655ec59944b1269025f032b79dc2c6dad366e54389ef6a6da2ddeb91d535a4027f2162fbde  nginx-dav-ext-module-3.0.0.tar.gz
c325ac4e3f3f735739e156d8c7ada503b34475c62533b4830231ff1b42c25cb0c841aae06b3448b589c2ab35da8d211436ed194d6fd062cad925af8152c5e789  echo-nginx-module-0.63.tar.gz
399ce2690e85ee27802e8031954a1a3aa3fdc9246e17323a72a298d235931a8dbebdcb121ac8788e074872df0ef5b5a8a3d512b17fbe860b38f696ce42de3655  encrypted-session-nginx-module-0.09.tar.gz
c208cdf3e245527d7b313f9ef1f5d36ca26e3bdafe67df56492a13b7726587538665e5d9fd50f295fc933f218dc33394f2fe442713d15631701dbfc4a156799b  ngx-fancyindex-0.5.2.tar.gz
18dea21e5ae2647bea1fc448058a1b773c936917245edef8d861d5e23ed92e9a3b1ec4ef43ffb2ece7b5899d787910adcf4fbd39f84d7e8d7c54759e2fee5b72  ngx_http_geoip2_module-3.4.tar.gz
2c0c140feeb29f0154a223dc3020ff956f894d63e0232a7bc0ca33fcb26f8b807bda868159ae30b6cac7456ec25b831c3d299ea18e234202ae5d14c1ff471a4b  headers-more-nginx-module-0.34.tar.gz
015a358d987476bb61302fbbe1cb105f5314edc1a8b7ee6310aae697f755c79fcb1834ff561fced054c8cd5624f5387fcc1de729731ccd70662f2eb72bcdc174  nginx-log-zmq-1.0.0.tar.gz
f57d66d4f3025b3056d9558b00727a592b4d7b63b74b35925b871a2be7ac5fe97ee5a99eb34e13f490a21d77ec27c1cd74321359201f151f008ccf3519ebd9c1  nchan-1.3.4.tar.gz
d6ca250db8de93edbd7875afca35e73cecdaf82132d1a7ee933cf94c6b8afa8e629e9e647a9321f2bc1fbb92137ec0d32dcd89b82ac5fae31e342537fb7e0431  redis2-nginx-module-0.15.tar.gz
1ff4c947538a5bd5f9d6adcd87b37f2702f5cc90e3342bc08359cbe8f290b705a3a2daa3dedfb1df3ce4bc19478c8fcac07081c4a53a804fc2862d50078278dc  set-misc-nginx-module-0.33.tar.gz
1730845ea2e52be8c2f6cfceb2894304c5a07959a96940bb1617ee0e7cf81d22283304f411d9a219ddb71e4d9a66012bba0f6f5574d101aeb3c406f26c5d6a4e  nginx-http-shibboleth-2.0.1.tar.gz
c3a7dd29c4a4e47d396b19622a290a04d4cceb97c1b8a508bc984eb8c81d17da4bf5789514bb996987f8343bc801ee4457a86a274bde98b49a809afdfc04cbde  ngx_http_untar_module-1.1.tar.gz
a0da355888398f86a6b5d065e58c05f9e057589ee785be9f515d77d7a020ae5a7b0656f5af30fb3b026f457326db2e26f4fed29026221ec5fc8156ef8586da25  nginx-upload-module-2.3.0.tar.gz
c31c46344d49704389722325a041b9cd170fa290acefe92cfc572c07f711cd3039de78f28df48ca7dcb79b2e4bbe442580aaaf4d92883fd3a14bf41d66dd9d8c  nginx-upload-progress-module-0.9.2.tar.gz
8adb7453c27748f4e685e3352e9b318b408da818754dc5b6244e908423941a8ba337561104f6e481f2553cbc0e334dcea73b57f8e810a9d6e974bb69ff8859e5  nginx-upstream-fair-0.1.3.tar.gz
bbf3f4808f17fa797fa0c27bc1351098aa5d6b5d227376a0ab01b4f424942ec5ad9f5c09c7af284345988b119268bf579b24065fb6bc6bc2f2b2392e918f09bc  ngx_upstream_jdomain-1.4.0.tar.gz
23bc21d1f841b1ebddec53836d9df64ed1bd3ae5d16f966f940ba617072a28f6a85cc0d32ad6ad06ff60185d2d2b4509fffd8470045b5e211f57aed0d2e6505e  nginx-vod-module-1.30.tar.gz
fadd4727ffc56111b443364d90e5b0597f09b25006404b11377586f0ed754f5a85e0b84796360be927bd455f43eb28e18004991f086b611146cd340937a6e5e9  nginx-module-vts-0.2.1.tar.gz
92e1e5aa570b68a19bb718817f864c4347f6dc89f90d828071ab3d06c784cc2786674d9d64fffef8c23749d0a653f2eb996b412ab10015eea1ed895d81268ce1  mod_zip-1.2.0.tar.gz
3f8c803221854c4b1a06aadc6313fbfec74bd7179c0ee51d4365b26ffa8875881a6e1e48f777a9c9efbb9170ab7478a82920d5448a2c2df485503d37bb03ab81  nginx-rtmp-module-1.2.2.tar.gz
"
