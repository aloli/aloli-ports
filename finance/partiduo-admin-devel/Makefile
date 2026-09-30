# Version de développement : branche development de partiduo-admin (shards
# maison sur leur branche production, voir shard.lock.packaging). Une seule
# administration par serveur : elle s'installe aux mêmes emplacements que
# partiduo-admin et le remplace.
#
# ATTENTION (30 septembre 2026) : la branche development publiée
# (ece5a01) ne contient pas encore le travail FreeBSD de partiduo-admin
# (branche freebsd : exécutant et enveloppes FreeBSD, fichiers rangés sous
# share/partiduo-admin). Tant qu'elle n'y est pas fusionnée, ces sources
# empaquettent l'ancien exécutant Linux et un serveur qui ne trouve pas ses
# gabarits une fois installé : régénérer après la fusion (README.adoc).
PKGNAMESUFFIX=	-devel

COMMENT=	Fleet administration of Partiduo instances (development version)

CONFLICTS_INSTALL=	partiduo-admin

MASTERDIR=	${.CURDIR}/../partiduo-admin

.include "${MASTERDIR}/Makefile"
