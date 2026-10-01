# Version de développement : branche development de partiduo-admin (shards
# maison sur leur branche production, voir shard.lock.packaging). Une seule
# administration par serveur : elle s'installe aux mêmes emplacements que
# partiduo-admin et le remplace.
PKGNAMESUFFIX=	-devel

COMMENT=	Fleet administration of Partiduo instances (development version)

CONFLICTS_INSTALL=	partiduo-admin

MASTERDIR=	${.CURDIR}/../partiduo-admin

.include "${MASTERDIR}/Makefile"
