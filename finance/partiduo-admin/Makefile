PORTNAME=	partiduo-admin
PORTREVISION=	1
CATEGORIES=	finance
DIST_SUBDIR=	partiduo

MAINTAINER=	philippe@aloli.fr
COMMENT?=	Fleet administration of Partiduo instances (web application)
WWW=		https://github.com/aloli-crystal/partiduo-admin

LICENSE=	AGPLv3+
LICENSE_FILE=	${WRKSRC}/LICENSE

BUILD_DEPENDS=	crystal:lang/crystal
# Bibliothèques liées (ligne d'édition de liens de crystal build pour
# FreeBSD) ; OpenSSL et zlib : système de base.
LIB_DEPENDS=	libgc-threaded.so:devel/boehm-gc-threaded \
		libgmp.so:math/gmp \
		libpcre2-8.so:devel/pcre2 \
		libyaml.so:textproc/libyaml

USES=		pkgconfig ssl
USE_GITHUB=	yes
GH_ACCOUNT=	aloli-crystal
GH_PROJECT=	partiduo-admin

# Une seule administration par serveur : partiduo-admin (branche production
# de partiduo-admin) et partiduo-admin-devel (branche development, port
# esclave) s'installent aux mêmes emplacements.
CONFLICTS_INSTALL?=	partiduo-admin-devel

USE_RC_SUBR=	partiduo_admin
SUB_FILES=	partiduo-admin.cron pkg-message
SUB_LIST=	PARTIDUO_ADMIN_HOME=${PARTIDUO_ADMIN_HOME} \
		PKGNAME=${PKGBASE}

# Tout ce que les programmes lisent à l'exécution est rangé à côté d'eux
# (bin/, share/partiduo-admin/{assets,artifacts}, voir
# config/settings/production.cr de partiduo-admin).
PARTIDUO_ADMIN_HOME=	${PREFIX}/lib/partiduo-admin

# Sources : le projet et chaque shard de son lock d'empaquetage, produits
# par Tools/packaging-lock et Tools/shards-tuples (poudriere construit sans
# réseau). Elles servent aussi aux ports sysutils/partiduo-agent[-devel],
# qui les lisent ici : même projet, même branche, mêmes archives.
PARTIDUO_ADMIN_SOURCES?=	${.CURDIR}
DISTINFO_FILE=	${PARTIDUO_ADMIN_SOURCES}/distinfo
.include "${PARTIDUO_ADMIN_SOURCES}/Makefile.version"
.include "${PARTIDUO_ADMIN_SOURCES}/Makefile.shards"

# Marten : pas de précompilation de sa ligne de commande (postinstall de
# shards, non lancé ici) ; ses programmes sont construits directement.
_PARTIDUO_ENV=	HOME=${WRKDIR} \
		CRYSTAL_CACHE_DIR=${WRKDIR}/crystal-cache \
		SKIP_MARTEN_CLI_PRECOMPILATION=1
# Réglages exigés par la configuration de production pour lancer
# partiduo-admin-manage à la construction (aucune base n'est ouverte).
_PARTIDUO_MANAGE_ENV=	${_PARTIDUO_ENV} \
		MARTEN_ENV=production \
		MARTEN_SECRET_KEY=construction \
		DATABASE_URL="postgres:///construction?host=/nonexistent"

_PARTIDUO_PROGRAMS=	partiduo-admin:src/server.cr \
			partiduo-admin-manage:manage.cr

do-build:
	${MKDIR} ${WRKSRC}/bin
.for _p in ${_PARTIDUO_PROGRAMS}
	cd ${WRKSRC} && ${SETENV} ${_PARTIDUO_ENV} crystal build --release --no-debug \
		-o bin/${_p:C/:.*//} ${_p:C/^[^:]*://}
.endfor

do-install:
	${MKDIR} ${STAGEDIR}${PARTIDUO_ADMIN_HOME}/bin \
		${STAGEDIR}${PARTIDUO_ADMIN_HOME}/share/partiduo-admin \
		${STAGEDIR}${PREFIX}/etc/partiduo-admin \
		${STAGEDIR}${PREFIX}/etc/cron.d \
		${STAGEDIR}/var/db/partiduo-admin \
		${STAGEDIR}/var/log/partiduo-admin
.for _p in ${_PARTIDUO_PROGRAMS}
	${INSTALL_PROGRAM} ${WRKSRC}/bin/${_p:C/:.*//} ${STAGEDIR}${PARTIDUO_ADMIN_HOME}/bin
.endfor
# Fichiers statiques d'abord : une fois artifacts/ présent, il devient la
# racine des applications (config/settings/production.cr).
	cd ${WRKSRC} && ${SETENV} ${_PARTIDUO_MANAGE_ENV} \
		PARTIDUO_ADMIN_ASSETS_ROOT=${STAGEDIR}${PARTIDUO_ADMIN_HOME}/share/partiduo-admin/assets \
		bin/partiduo-admin-manage collectassets --no-input
	cd ${WRKSRC} && ${SETENV} ${_PARTIDUO_MANAGE_ENV} \
		bin/partiduo-admin-manage collectartifacts --no-input \
		--dest-path ${STAGEDIR}${PARTIDUO_ADMIN_HOME}/share/partiduo-admin/artifacts
	${LN} -s ${PARTIDUO_ADMIN_HOME}/bin/partiduo-admin-manage \
		${STAGEDIR}${PREFIX}/bin/partiduo-admin-manage
	${INSTALL_DATA} ${FILESDIR}/admin.env.sample \
		${STAGEDIR}${PREFIX}/etc/partiduo-admin/admin.env.sample
	${INSTALL_DATA} ${WRKDIR}/partiduo-admin.cron \
		${STAGEDIR}${PREFIX}/etc/cron.d/partiduo-admin

post-install:
	cd ${STAGEDIR}${PREFIX} && ${FIND} lib/partiduo-admin \( -type f -o -type l \) | \
		${SORT} >> ${TMPPLIST}
	${ECHO_CMD} bin/partiduo-admin-manage >> ${TMPPLIST}
	${ECHO_CMD} "@sample(root,partiduo-admin,0640) etc/partiduo-admin/admin.env.sample" >> ${TMPPLIST}
	${ECHO_CMD} etc/cron.d/partiduo-admin >> ${TMPPLIST}
# Répertoire de travail (verrou de la planification) et journal de la
# planification, écrits par le compte partiduo-admin ; conservés s'ils ne
# sont pas vides.
	${ECHO_CMD} "@dir(partiduo-admin,partiduo-admin,0750) /var/db/partiduo-admin" >> ${TMPPLIST}
	${ECHO_CMD} "@dir(partiduo-admin,partiduo-admin,0750) /var/log/partiduo-admin" >> ${TMPPLIST}

.include <bsd.port.mk>
