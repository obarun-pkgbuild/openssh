# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/openssh
# 						Maintainer: Gaetan Bisson <bisson@archlinux.org>
# 						Contributor: Aaron Griffin <aaron@archlinux.org>
# 						Contributor: judd <jvinet@zeroflux.org>

pkgname=openssh
pkgver=7.6p1
pkgrel=2
pkgdesc='Free version of the SSH connectivity tools'
url='https://www.openssh.org/portable.html'
license=('custom:BSD')
arch=('x86_64')
makedepends=('linux-headers')
depends=('krb5' 'openssl' 'libedit' 'ldns')
optdepends=('xorg-xauth: X11 forwarding'
            'x11-ssh-askpass: input passphrase in X'
            'sshd-s6serv: sshd s6 service'
            'sshd-runitserv: sshd runit service')
validpgpkeys=('59C2118ED206D927E667EBE3D3E5F56B6D920D30')
source=("https://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/${pkgname}-${pkgver}.tar.gz"
        'openssl-1.1.0.patch'
        'sshd.pam'
        'sshd.conf')
sha1sums=('a6984bc2c72192bed015c8b879b35dd9f5350b3b'
          '6c16b162aa98ff5bdfd6d747e3219919f7b1c729'
          'd93dca5ebda4610ff7647187f8928a3de28703f3'
          'c9b2e4ce259cd62ddb00364d3ee6f00a8bf2d05f')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

backup=('etc/ssh/ssh_config' 'etc/ssh/sshd_config' 'etc/pam.d/sshd')

 prepare() {
 	cd $pkgname-$pkgver

 	# OpenSSL 1.1.0 patch from http://vega.pgw.jp/~kabe/vsd/patch/openssh-7.4p1-openssl-1.1.0c.patch.html
	patch -Np1 -i ../openssl-1.1.0.patch
}

build() {
	cd "${srcdir}/${pkgname}-${pkgver}"

	./configure \
		--prefix=/usr \
		--sbindir=/usr/bin \
		--libexecdir=/usr/lib/ssh \
		--sysconfdir=/etc/ssh \
		--with-ldns \
		--with-libedit \
		--with-ssl-engine \
		--with-pam \
		--with-privsep-user=nobody \
		--with-kerberos5=/usr \
		--with-xauth=/usr/bin/xauth \
		--with-md5-passwords \
		--with-pid-dir=/run \

	make
}

check() {
	cd "${srcdir}/${pkgname}-${pkgver}"

	# Tests require openssh to be already installed system-wide,
	# also connectivity tests will fail under makechrootpkg since
    # it runs as nobody which has /bin/false as login shell.

	if [[ -e /usr/bin/scp && ! -e /.arch-chroot ]]; then
		make tests
	fi
	
}

package() {
	cd "${srcdir}/${pkgname}-${pkgver}"

	make DESTDIR="${pkgdir}" install

	ln -sf ssh.1.gz "${pkgdir}"/usr/share/man/man1/slogin.1.gz
	install -Dm644 LICENCE "${pkgdir}/usr/share/licenses/${pkgname}/LICENCE"
	
	install -Dm644 ../sshd.conf "${pkgdir}"/usr/lib/tmpfiles.d/sshd.conf
	
	install -Dm644 ../sshd.pam "${pkgdir}"/etc/pam.d/sshd
		
	install -Dm755 contrib/findssl.sh "${pkgdir}"/usr/bin/findssl.sh
	install -Dm755 contrib/ssh-copy-id "${pkgdir}"/usr/bin/ssh-copy-id
	install -Dm644 contrib/ssh-copy-id.1 "${pkgdir}"/usr/share/man/man1/ssh-copy-id.1
	
	
	
	sed \
		-e '/^#ChallengeResponseAuthentication yes$/c ChallengeResponseAuthentication no' \
		-e '/^#PrintMotd yes$/c PrintMotd no # pam does that' \
		-e '/^#UsePAM no$/c UsePAM yes' \
		-i "${pkgdir}"/etc/ssh/sshd_config
}
