PORTNAME=	partiduo-agent
CATEGORIES=	sysutils
DIST_SUBDIR=	partiduo

MAINTAINER=	philippe@aloli.fr
COMMENT?=	Executor of the Partiduo fleet administration on a hosting server
WWW=		https://github.com/aloli-crystal/partiduo-admin

LICENSE=	AGPLv3+
LICENSE_FILE=	${WRKSRC}/LICENSE

BUILD_DEPENDS=	crystal:lang/crystal
# Bibliothèques liées (ligne d'édition de liens de crystal build pour
# FreeBSD) ; OpenSSL et zlib : système de base.
LIB_DEPENDS=	libgc-threaded.so:devel/boehm-gc-threaded \
		libpcre2-8.so:devel/pcre2
# sudo : les deux enveloppes (sudoers.d/partiduo-agent) ; client PostgreSQL
# (psql, createdb, dropdb, pg_dump, pg_restore) : bases des instances.
RUN_DEPENDS=	sudo:security/sudo

USES=		pgsql pkgconfig ssl
WANT_PGSQL=	client
USE_GITHUB=	yes
GH_ACCOUNT=	aloli-crystal
GH_PROJECT=	partiduo-admin

# Un seul exécutant par serveur : partiduo-agent (branche production de
# partiduo-admin) et partiduo-agent-devel (branche development, port
# esclave) s'installent aux mêmes emplacements.
CONFLICTS_INSTALL?=	partiduo-agent-devel

USE_RC_SUBR=	partiduo_agent
SUB_FILES=	pkg-message
SUB_LIST=	PKGNAME=${PKGBASE}

# Sources : celles du port finance/partiduo-admin de la même branche (même
# projet, mêmes shards : Makefile.version, Makefile.shards, distinfo), pour
# qu'une seule génération (Tools/packaging-lock, Tools/shards-tuples) serve
# l'administration et l'exécutant.
PARTIDUO_ADMIN_SOURCES?=	${.CURDIR}/../../finance/partiduo-admin
DISTINFO_FILE=	${PARTIDUO_ADMIN_SOURCES}/distinfo
.include "${PARTIDUO_ADMIN_SOURCES}/Makefile.version"
.include "${PARTIDUO_ADMIN_SOURCES}/Makefile.shards"

# Marten : pas de précompilation de sa ligne de commande (postinstall de
# shards, non lancé ici).
_PARTIDUO_ENV=	HOME=${WRKDIR} \
		CRYSTAL_CACHE_DIR=${WRKDIR}/crystal-cache \
		SKIP_MARTEN_CLI_PRECOMPILATION=1

do-build:
	${MKDIR} ${WRKSRC}/bin
	cd ${WRKSRC} && ${SETENV} ${_PARTIDUO_ENV} crystal build --release --no-debug \
		-o bin/partiduo-agent src/agent.cr

# Enveloppes de sudo : root, 0755, dans un répertoire root 0755 ; règles
# sudoers root 0440 ; configuration des enveloppes root 0644 (@sample).
do-install:
	${INSTALL_PROGRAM} ${WRKSRC}/bin/partiduo-agent ${STAGEDIR}${PREFIX}/bin
	${MKDIR} ${STAGEDIR}${PREFIX}/libexec/partiduo-agent \
		${STAGEDIR}${PREFIX}/etc/sudoers.d \
		${STAGEDIR}${PREFIX}/etc/partiduo-agent \
		${STAGEDIR}/var/backups/partiduo \
		${STAGEDIR}/var/db/partiduo-stage \
		${STAGEDIR}/var/db/partiduo-agent
.for _h in partiduo-agent-root partiduo-agent-instance
	${INSTALL} -m 755 ${WRKSRC}/deploy/libexec/${_h} ${STAGEDIR}${PREFIX}/libexec/partiduo-agent
.endfor
	${INSTALL} -m 440 ${WRKSRC}/deploy/partiduo-agent.sudoers \
		${STAGEDIR}${PREFIX}/etc/sudoers.d/partiduo-agent
	${INSTALL_DATA} ${WRKSRC}/deploy/helpers.conf.example \
		${STAGEDIR}${PREFIX}/etc/partiduo-agent/helpers.conf.sample

post-install:
	${ECHO_CMD} bin/partiduo-agent >> ${TMPPLIST}
	${ECHO_CMD} "@(root,wheel,0755) libexec/partiduo-agent/partiduo-agent-root" >> ${TMPPLIST}
	${ECHO_CMD} "@(root,wheel,0755) libexec/partiduo-agent/partiduo-agent-instance" >> ${TMPPLIST}
	${ECHO_CMD} "@(root,wheel,0440) etc/sudoers.d/partiduo-agent" >> ${TMPPLIST}
	${ECHO_CMD} "@sample(root,wheel,0644) etc/partiduo-agent/helpers.conf.sample" >> ${TMPPLIST}
# Sauvegardes : les deux comptes y écrivent (partiduo par pg_dump et tar,
# partiduo-agent par le groupe ; les répertoires des dossiers héritent du
# groupe, règle BSD). Fichiers de partiduo-provision (compte des instances),
# relus par root. État de l'exécutant (tâches en cours, clé du serveur des
# sauvegardes). Conservés s'ils ne sont pas vides.
	${ECHO_CMD} "@dir(partiduo,partiduo-agent,0770) /var/backups/partiduo" >> ${TMPPLIST}
	${ECHO_CMD} "@dir(partiduo,partiduo,0750) /var/db/partiduo-stage" >> ${TMPPLIST}
	${ECHO_CMD} "@dir(partiduo-agent,partiduo-agent,0700) /var/db/partiduo-agent" >> ${TMPPLIST}

.include <bsd.port.mk>
