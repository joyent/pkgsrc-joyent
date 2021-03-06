# $NetBSD$

MYSQL_VER=	8.0.21
WSREP_MAJOR=	12

DISTNAME=	Percona-XtraDB-Cluster-${MYSQL_VER}-${WSREP_MAJOR}
PKGNAME=	percona-cluster-${MYSQL_VER}
CATEGORIES=	databases
MASTER_SITES=	https://www.percona.com/downloads/Percona-XtraDB-Cluster-80/${DISTNAME:S/-${WSREP_MAJOR}//}/source/tarball/

COMMENT=	Percona XTraDB Cluster
MAINTAINER=	jperkin@joyent.com
HOMEPAGE=	http://www.percona.com/
LICENSE=	gnu-gpl-v2 AND gnu-gpl-v3

CONFLICTS=	{mysql,percona}*-{client,cluster,server}-[0-9]*

DEPENDS+=	coreutils>=0:../../sysutils/coreutils	# GNU mktemp
DEPENDS+=	findutils>=0:../../sysutils/findutils	# find -regex
DEPENDS+=	grep>=0:../../textproc/grep		# grep -o
DEPENDS+=	socat>=0:../../net/socat
DEPENDS+=	percona-xtrabackup>=0:../../joyent/percona-xtrabackup

# nbpatch fails
BUILD_DEPENDS+=	patch-[0-9]*:../../devel/patch
PATCH=		${LOCALBASE}/bin/gpatch

#
# Has specific boost version requirements so use bundled that we fetch
# ourselves.  Keep the version synced from cmake/boost.cmake.  Note that
# later on we have to pull in boost-libs as a manual DEPENDS as the galera
# plugin requires libboost_program_options.a
#
BOOST=			boost_1_72_0
DISTFILES=		${DISTNAME}${EXTRACT_SUFX} ${BOOST}.tar.bz2
SITES.${BOOST}.tar.bz2=	${MASTER_SITE_SOURCEFORGE:=boost/}

USE_CMAKE=		yes
USE_LANGUAGES=		c99 c++14
USE_TOOLS+=		bash bison gmake perl:run

.include "../../mk/bsd.prefs.mk"

MYSQL_USER?=		mysql
MYSQL_GROUP?=		mysql
MYSQL_DATADIR?=		${VARBASE}/mysql

OWN_DIRS_PERMS+=	${MYSQL_DATADIR} ${MYSQL_USER} ${MYSQL_GROUP} 0700
OWN_DIRS_PERMS+=	${VARBASE}/log/mysql ${MYSQL_USER} ${MYSQL_GROUP} 0700

PKG_USERS_VARS+=	MYSQL_USER
PKG_GROUPS_VARS+=	MYSQL_GROUP

CONFIG_SHELL=		${TOOLS_PATH.bash}

CONFIGURE_DIRS=		${WRKDIR}/build
CMAKE_ARG_PATH=		${WRKSRC}

CMAKE_ARGS+=		-DCMAKE_BUILD_TYPE=Release
CMAKE_ARGS+=		-DINSTALL_DOCDIR="share/doc/mysql"
CMAKE_ARGS+=		-DINSTALL_DOCREADMEDIR="share/doc/mysql"
CMAKE_ARGS+=		-DINSTALL_INCLUDEDIR="include/mysql"
CMAKE_ARGS+=		-DINSTALL_INFODIR="info"
CMAKE_ARGS+=		-DINSTALL_MANDIR="${PKGMANDIR}"
CMAKE_ARGS+=		-DINSTALL_MYSQLSHAREDIR="share/mysql"
CMAKE_ARGS+=		-DINSTALL_MYSQLTESTDIR=""
CMAKE_ARGS+=		-DINSTALL_PLUGINDIR="lib/mysql/plugin"
CMAKE_ARGS+=		-DINSTALL_SBINDIR="sbin"
CMAKE_ARGS+=		-DINSTALL_SUPPORTFILESDIR="share/mysql"
CMAKE_ARGS+=		-DMYSQL_DATADIR=${MYSQL_DATADIR}
CMAKE_ARGS+=		-DWITH_BOOST="${WRKDIR}/${BOOST}"
CMAKE_ARGS+=		-DWITH_LZ4="system"
CMAKE_ARGS+=		-DWITH_PROTOBUF="system"
CMAKE_ARGS+=		-DWITH_SSL="system"
CMAKE_ARGS+=		-DWITH_WSREP=1
CMAKE_ARGS+=		-DWITH_ZLIB="system"

CMAKE_ARGS.SunOS+=	-DFORCE_UNSUPPORTED_COMPILER=YES

# Force HAVE_CURSES_H on Solaris since the configure script is broken
# and does not properly detect this, breaking the build later on.  Also
# explicitly disable epoll support as it is Linux-specific.
CMAKE_ARGS.SunOS+=	-DHAVE_SYS_EPOLL_H=NO
CMAKE_ARGS.SunOS+=	-DHAVE_EPOLL=NO
CMAKE_ARGS.SunOS+=	-DEVENT__HAVE_SYS_EPOLL_H=NO
CMAKE_ARGS.SunOS+=	-DEVENT__HAVE_EPOLL=NO

LDFLAGS.SunOS+=		-lsocket

.include "../../mk/readline.buildlink3.mk"

.if ${READLINE_TYPE} == "readline"
CMAKE_ARGS+=		-DWITH_READLINE="system"
.else
CMAKE_ARGS+=		-DWITH_EDITLINE="system"
.endif

BUILD_DEFS+=		VARBASE

CHECK_INTERPRETER_SKIP+=	share/mysql/test/*
CHECK_PORTABILITY_SKIP+=	build-ps/*
CHECK_PORTABILITY_SKIP+=	packaging/deb-in/*
CHECK_PORTABILITY_SKIP+=	percona-xtradb-cluster-galera/scripts/*
CHECK_PORTABILITY_SKIP+=	percona-xtradb-cluster-galera/scripts/mysql/*/mysqld_safe*
CHECK_PORTABILITY_SKIP+=	storage/ndb/mcc/frontend/dojo/dojox/mobile/build/build.sh

# failed to set type size for 601 to 0x200: Invalid argument
CTF_FILES_SKIP+=	${PREFIX}/bin/myisamlog
CTF_FILES_SKIP+=	${PREFIX}/bin/replace

# Required for lib/mysqlrouter/*.so
LDFLAGS+=		${COMPILER_RPATH_FLAG}${PREFIX}/lib/mysql
LDFLAGS+=		${COMPILER_RPATH_FLAG}${PREFIX}/lib/mysqlrouter

PKG_GROUPS=		${MYSQL_GROUP}
PKG_USERS=		${MYSQL_USER}:${MYSQL_GROUP}

PKG_GECOS.${MYSQL_USER}=	MySQL database administrator
PKG_HOME.${MYSQL_USER}=		${MYSQL_DATADIR}
PKG_SHELL.${MYSQL_USER}=	${SH}

RCD_SCRIPTS=		mysqld
SMF_NAME=		percona
SMF_METHODS=		${RCD_SCRIPTS}

FILES_SUBST+=		MYSQL_DATADIR=${MYSQL_DATADIR}
FILES_SUBST+=		MYSQL_USER=${MYSQL_USER} MYSQL_GROUP=${MYSQL_GROUP}
BUILD_DEFS+=		MYSQL_DATADIR

REPLACE_PERL+=		support-files/mysqld_multi.server.sh
REPLACE_PERL+=		scripts/mysqldumpslow.sh

REPLACE_PYTHON+=	scripts/pyclustercheck

SUBST_CLASSES+=		paths
SUBST_STAGE.paths=	pre-configure
SUBST_FILES.paths=	scripts/mysqld_safe.sh
SUBST_FILES.paths+=	support-files/mysql.server.sh
SUBST_SED.paths=	-e "s,chown,${CHOWN},g"
SUBST_SED.paths+=	-e "s,/bin/sh,${RCD_SCRIPTS_SHELL},g"

SUBST_CLASSES+=		prefix
SUBST_STAGE.prefix=	pre-configure
SUBST_FILES.prefix=	scripts/wsrep_sst_xtrabackup-v2.sh
SUBST_VARS.prefix=	LOCALBASE

CONF_FILES=		${PREFIX}/share/examples/mysql/my-cluster.cnf \
			${PKG_SYSCONFDIR}/my.cnf

INSTALLATION_DIRS+=	share/examples/mysql

post-extract:
	${MKDIR} ${CONFIGURE_DIRS}

#
# Sigh.  The galera plugin needs libboost_program_options.a which isn't
# built as part of the bundled boost.  We can't just buildlink boost as
# normal though because it's then picked up before the bundled boost.
#
# So we're left to handle it with DEPENDS and then munge the files we need
# manually during the build to work around buildlink.
#
BUILD_DEPENDS+=		boost-libs>0:../../devel/boost-libs
#
# The file needs to be copied into the work directory to avoid buildlink.
#
GALERA_SCONS_ARGS+=	bpostatic=${WRKDIR}/libboost_program_options.a

#
# Scons deliberately strips environment settings and causes havoc with
# buildlink, so we need to put some things back manually..
#
GALERA_SCONS_ENV=	${MAKE_ENV}
GALERA_SCONS_ENV+=	CPPFLAGS=-I${WRKDIR}/${BOOST}
GALERA_SCONS_ENV+=	LIBPATH=${BUILDLINK_PREFIX.openssl}/lib
GALERA_SCONS_ENV+=	RPATH=${BUILDLINK_PREFIX.openssl}/lib
GALERA_SCONS_ARGS+=	${_MAKE_JOBS}

post-build:
	${CP} ${LOCALBASE}/lib/libboost_program_options.a ${WRKDIR}
	${RUN} cd ${WRKSRC}/percona-xtradb-cluster-galera && \
		revno=`cat GALERA-REVISION` && \
		${SETENV} ${GALERA_SCONS_ENV} revno=$$revno ${SCONSBIN} \
		    ${GALERA_SCONS_ARGS} psi=1 libgalera_smm.so && \
		${SETENV} ${GALERA_SCONS_ENV} revno=$$revno ${SCONSBIN} \
		    ${GALERA_SCONS_ARGS} garb/garbd
	${SED} ${FILES_SUBST_SED} ${FILESDIR}/my-cluster.cnf.sh \
	    >${WRKSRC}/my-cluster.cnf

post-install:
	${INSTALL_LIB} ${WRKSRC}/percona-xtradb-cluster-galera/libgalera_smm.so ${DESTDIR}${PREFIX}/lib/mysql/plugin/
	${INSTALL_PROGRAM} ${WRKSRC}/percona-xtradb-cluster-galera/garb/garbd ${DESTDIR}${PREFIX}/bin/
	${INSTALL_MAN} ${WRKSRC}/percona-xtradb-cluster-galera/man/garbd.8 ${DESTDIR}${PREFIX}/${PKGMANDIR}/man8/
	${INSTALL_DATA} ${WRKSRC}/my-cluster.cnf ${DESTDIR}${PREFIX}/share/examples/mysql/my-cluster.cnf
	${CHMOD} g-w ${DESTDIR}${PREFIX}
	${CHMOD} -x ${DESTDIR}${PREFIX}/bin/wsrep_sst_common
	${CHMOD} -x ${DESTDIR}${PREFIX}/share/mysql/mysql-log-rotate

.include "../../archivers/lz4/buildlink3.mk"
.include "../../devel/check/buildlink3.mk"
.include "../../devel/libevent/buildlink3.mk"
.include "../../devel/protobuf/buildlink3.mk"
.include "../../devel/scons/buildlink3.mk"
.include "../../devel/zlib/buildlink3.mk"
.include "../../lang/python/application.mk"
.include "../../security/openssl/buildlink3.mk"
.include "../../security/tcp_wrappers/buildlink3.mk"
.include "../../www/curl/buildlink3.mk"
.include "../../mk/bsd.pkg.mk"
