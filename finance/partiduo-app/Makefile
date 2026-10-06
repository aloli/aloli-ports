PORTNAME=	partiduo-app
DISTVERSION?=	0.1.0
PORTREVISION=	1
CATEGORIES=	finance
DIST_SUBDIR=	partiduo

MAINTAINER=	philippe@aloli.fr
COMMENT?=	Invoicing and accounting for small businesses (instance server)
WWW=		https://github.com/aloli-crystal/partiduo-dist

LICENSE=	AGPLv3
LICENSE_FILE=	${WRKSRC}/LICENSE

BUILD_DEPENDS=	crystal:lang/crystal
LIB_DEPENDS=	libevent.so:devel/libevent \
		libgc-threaded.so:devel/boehm-gc-threaded \
		libgmp.so:math/gmp \
		libpcre2-8.so:devel/pcre2 \
		libxml2.so:textproc/libxml2 \
		libyaml.so:textproc/libyaml
# Polices du convertisseur PDF de MODELES (share/partiduo-modeles-pdf/fonts).
RUN_DEPENDS=	${LOCALBASE}/share/fonts/dejavu/DejaVuSans.ttf:x11-fonts/dejavu

USES=		pkgconfig ssl
USE_GITHUB=	yes
GH_ACCOUNT=	aloli-crystal
GH_PROJECT=	partiduo-dist

# Une seule version par serveur : partiduo-app (branche production de
# partiduo-dist) et partiduo-app-devel (branche development, port esclave,
# pour le serveur de test) s'installent aux mêmes emplacements.
CONFLICTS_INSTALL?=	partiduo-app-devel

# Tout ce qu'un programme lit à l'exécution est rangé à côté de lui (bin/,
# share/).
PARTIDUO_NAME=	partiduo
PARTIDUO_RC=	partiduo
PARTIDUO_HOME=	${PREFIX}/lib/${PARTIDUO_NAME}

SUB_FILES=	partiduo-rc pkg-message
SUB_LIST=	PARTIDUO_NAME=${PARTIDUO_NAME} \
		PARTIDUO_RC=${PARTIDUO_RC} \
		PARTIDUO_HOME=${PARTIDUO_HOME} \
		PKGNAME=${PKGBASE}

# Sources : le projet et chaque shard de son shard.lock, produits par
# Tools/shards-tuples (poudriere construit sans réseau).
DISTINFO_FILE=	${.CURDIR}/distinfo
.include "${.CURDIR}/Makefile.shards"

# Marten : pas de précompilation de sa ligne de commande (postinstall de
# shards, non lancé ici) ; ses programmes sont construits directement.
_PARTIDUO_ENV=	HOME=${WRKDIR} \
		CRYSTAL_CACHE_DIR=${WRKDIR}/crystal-cache \
		SKIP_MARTEN_CLI_PRECOMPILATION=1
# Réglages exigés par la configuration de production pour lancer
# partiduo-manage à la construction (aucune base n'est ouverte).
_PARTIDUO_MANAGE_ENV=	${_PARTIDUO_ENV} \
		MARTEN_ENV=production \
		MARTEN_SECRET_KEY=construction \
		MARTEN_ALLOWED_HOSTS=localhost \
		DATABASE_URL="postgres:///construction?host=/nonexistent"

_PARTIDUO_PROGRAMS=	partiduo-server:src/server.cr \
			partiduo-manage:manage.cr \
			partiduo-modeles-pdf:lib/partiduo-modeles/src/partiduo-modeles/pdf.cr

do-build:
	${MKDIR} ${WRKSRC}/bin
.for _p in ${_PARTIDUO_PROGRAMS}
	cd ${WRKSRC} && ${SETENV} ${_PARTIDUO_ENV} crystal build --release --no-debug \
		-o bin/${_p:C/:.*//} ${_p:C/^[^:]*://}
.endfor

do-install:
	${MKDIR} ${STAGEDIR}${PARTIDUO_HOME}/bin \
		${STAGEDIR}${PARTIDUO_HOME}/share/partiduo \
		${STAGEDIR}${PARTIDUO_HOME}/share/partiduo-modeles-pdf
.for _p in ${_PARTIDUO_PROGRAMS}
	${INSTALL_PROGRAM} ${WRKSRC}/bin/${_p:C/:.*//} ${STAGEDIR}${PARTIDUO_HOME}/bin
.endfor
# Fichiers statiques d'abord : une fois artifacts/ présent, il devient la
# racine des applications (partiduo-dist, config/settings/production.cr).
	cd ${WRKSRC} && ${SETENV} ${_PARTIDUO_MANAGE_ENV} \
		PARTIDUO_ASSETS_ROOT=${STAGEDIR}${PARTIDUO_HOME}/share/partiduo/assets \
		bin/partiduo-manage collectassets --no-input
	cd ${WRKSRC} && ${SETENV} ${_PARTIDUO_MANAGE_ENV} \
		bin/partiduo-manage collectartifacts --no-input \
		--dest-path ${STAGEDIR}${PARTIDUO_HOME}/share/partiduo/artifacts
	${LN} -s ${LOCALBASE}/share/fonts/dejavu \
		${STAGEDIR}${PARTIDUO_HOME}/share/partiduo-modeles-pdf/fonts
	${LN} -s ${PARTIDUO_HOME}/bin/partiduo-manage \
		${STAGEDIR}${PREFIX}/bin/${PARTIDUO_NAME}-manage
	${INSTALL_SCRIPT} ${WRKDIR}/partiduo-rc ${STAGEDIR}${PREFIX}/etc/rc.d/${PARTIDUO_RC}
# Création d'une instance (partiduo-app, bin/partiduo-provision) : le script
# trouve ses gabarits dans ../deploy/templates.
	${INSTALL_SCRIPT} ${WRKSRC}/lib/partiduo/bin/partiduo-provision ${STAGEDIR}${PARTIDUO_HOME}/bin
	${MKDIR} ${STAGEDIR}${PARTIDUO_HOME}/deploy
	cd ${WRKSRC}/lib/partiduo/deploy && ${COPYTREE_SHARE} templates ${STAGEDIR}${PARTIDUO_HOME}/deploy
	${LN} -s ${PARTIDUO_HOME}/bin/partiduo-provision \
		${STAGEDIR}${PREFIX}/bin/${PARTIDUO_NAME}-provision

post-install:
	cd ${STAGEDIR}${PREFIX} && ${FIND} lib/${PARTIDUO_NAME} \( -type f -o -type l \) | \
		${SORT} >> ${TMPPLIST}
	${ECHO_CMD} bin/${PARTIDUO_NAME}-manage >> ${TMPPLIST}
	${ECHO_CMD} bin/${PARTIDUO_NAME}-provision >> ${TMPPLIST}
	${ECHO_CMD} etc/rc.d/${PARTIDUO_RC} >> ${TMPPLIST}

.include <bsd.port.mk>
