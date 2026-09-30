# Version de développement : branche development de partiduo-admin, mêmes
# sources que finance/partiduo-admin-devel. Un seul exécutant par serveur :
# elle s'installe aux mêmes emplacements que partiduo-agent et le remplace.
# Voir l'avertissement de finance/partiduo-admin-devel/Makefile (branche
# development pas encore FreeBSD).
PKGNAMESUFFIX=	-devel

COMMENT=	Executor of the Partiduo fleet administration (development version)

CONFLICTS_INSTALL=	partiduo-agent

PARTIDUO_ADMIN_SOURCES=	${.CURDIR}/../../finance/partiduo-admin-devel

MASTERDIR=	${.CURDIR}/../partiduo-agent

.include "${MASTERDIR}/Makefile"
