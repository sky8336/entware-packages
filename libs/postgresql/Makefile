# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=postgresql
PKG_VERSION:=9.6.4
PKG_RELEASE:=1
PKG_MAINTAINER:=Daniel Golle <daniel@makrotopia.org>
PKG_LICENSE:=PostgreSQL

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.bz2
PKG_SOURCE_URL:=\
	https://ftp.postgresql.org/pub/source/v$(PKG_VERSION) \
	http://ftp.postgresql.org/pub/source/v$(PKG_VERSION) \
	ftp://ftp.postgresql.org/pub/source/v$(PKG_VERSION)

PKG_HASH:=2b3ab16d82e21cead54c08b95ce3ac480696944a68603b6c11b3205b7376ce13

PKG_USE_MIPS16:=0
PKG_FIXUP:=autoreconf
PKG_MACRO_PATHS:=config
PKG_BUILD_DEPENDS += readline/host postgresql/host
PKG_INSTALL:=1

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk

define Package/libpq
  SECTION:=libs
  CATEGORY:=Libraries
  DEPENDS:=+zlib +libreadline +libpthread +libncursesw
  TITLE:=PostgreSQL client library
  URL:=http://www.postgresql.org/
  SUBMENU:=database
endef

define Package/libpq/description
PostgreSQL client library.
endef

define Package/pgsql-cli
  SECTION:=utils
  CATEGORY:=Utilities
  DEPENDS:=+libpq +librt
  TITLE:=Command Line Interface (CLI) to PostgreSQL databases
  URL:=http://www.postgresql.org/
  SUBMENU:=database
endef

define Package/pgsql-cli/description
Command Line Interface (CLI) to PostgreSQL databases.
endef

define Package/pgsql-cli-extra
  SECTION:=utils
  CATEGORY:=Utilities
  DEPENDS:=+libpq +librt
  TITLE:=Command Line extras for PostgreSQL databases
  URL:=http://www.postgresql.org/
  SUBMENU:=database
endef

define Package/pgsql-cli-extra/description
Command Line extras for PostgreSQL databases.
endef

define Package/pgsql-server
  SECTION:=utils
  CATEGORY:=Utilities
  DEPENDS:=+libpq +librt +pgsql-cli
  TITLE:=PostgreSQL databases Server
  URL:=http://www.postgresql.org/
  SUBMENU:=database
  USERID:=postgres=5432:postgres=5432
endef

define Package/pgsql-server/description
PostgreSQL databases Server.
endef

PGSQL_SERVER_BIN := \
	pg_archivecleanup \
	pg_basebackup \
	pg_controldata \
	pg_ctl \
	pg_dump \
	pg_dumpall \
	pg_isready \
	pg_receivexlog \
	pg_recvlogical \
	pg_resetxlog \
	pg_restore \
	pg_standby \
	pg_upgrade \
	pg_xlogdump \
	postgres \
	initdb

PGSQL_CLI_EXTRA_BIN := \
	clusterdb	\
	createdb	\
	createlang	\
	createuser	\
	dropdb		\
	droplang	\
	dropuser	\
	pgbench		\
	reindexdb	\
	vacuumdb

PGSQL_CONFIG_VARS:= \
	pgac_cv_snprintf_long_long_int_format="%lld" \
	pgac_cv_snprintf_size_t_support=yes

ifeq ($(CONFIG_USE_UCLIBC),y)
# PostgreSQL does not build against uClibc with locales
# enabled, due to an uClibc bug, see
# http://lists.uclibc.org/pipermail/uclibc/2014-April/048326.html
# so overwrite automatic detection and disable locale support
PGSQL_CONFIG_VARS+= \
		pgac_cv_type_locale_t=no
endif

TARGET_CONFIGURE_OPTS+=$(PGSQL_CONFIG_VARS)

HOST_CONFIGURE_ARGS += \
			--disable-nls \
			--disable-rpath \
			--without-bonjour \
			--without-gssapi \
			--without-ldap \
			--without-openssl \
			--without-pam \
			--without-perl \
			--without-python \
			--without-readline \
			--without-tcl \
			--with-zlib="yes" \
			--enable-depend

CONFIGURE_ARGS += \
			$(DISABLE_NLS) \
			--disable-rpath \
			--without-bonjour \
			--without-gssapi \
			--without-ldap \
			--without-openssl \
			--without-pam \
			--without-perl \
			--without-python \
			--without-tcl \
			--with-zlib="yes" \
			--enable-depend \
			$(if $(CONFIG_TARGET_avr32),--disable-spinlocks)

EXTRA_CFLAGS:= -I$(PKG_BUILD_DIR)/src/include/

# Need a native ecpg, pg_config and zic for build
define Host/Compile
	$(MAKE) -C $(HOST_BUILD_DIR)/src/bin/pg_config CC="$(HOSTCC)"
	$(MAKE) -C $(HOST_BUILD_DIR)/src/interfaces/ecpg/preproc CC="$(HOSTCC)"
	$(MAKE) -C $(HOST_BUILD_DIR)/src/timezone CC="$(HOSTCC)"
endef

define Host/Install
	$(INSTALL_DIR) $(STAGING_DIR_HOSTPKG)/lib/
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/src/common/libpgcommon.a $(STAGING_DIR_HOSTPKG)/lib/
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/src/port/libpgport.a $(STAGING_DIR_HOSTPKG)/lib/
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/src/bin/pg_config/pg_config $(STAGING_DIR_HOSTPKG)/lib/
	$(INSTALL_DIR) $(STAGING_DIR_HOSTPKG)/bin/
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/src/interfaces/ecpg/preproc/ecpg $(STAGING_DIR_HOSTPKG)/bin/
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/src/timezone/zic $(STAGING_DIR_HOSTPKG)/bin/
endef

define Build/Configure
	$(Build/Configure/Default)
	$(SED) 's@ECPG = ../../preproc/ecpg@ECPG = $(STAGING_DIR_HOSTPKG)/bin/ecpg@' $(PKG_BUILD_DIR)/src/interfaces/ecpg/test/Makefile.regress
endef

TARGET_CFLAGS += $(FPIC) -lpthread

# because PROFILE means something else in the project Makefile
unexport PROFILE

define Package/libpq/install
	$(INSTALL_DIR) $(1)/opt/lib
	$(CP) $(PKG_INSTALL_DIR)/opt/lib/libpq.so.* $(1)/opt/lib/
endef

define Package/pgsql-cli/install
	$(INSTALL_DIR) $(1)/opt/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/opt/bin/psql $(1)/opt/bin/
endef

define Package/pgsql-cli-extra/install
	$(INSTALL_DIR) $(1)/opt/bin
	$(INSTALL_BIN) $(foreach bin,$(PGSQL_CLI_EXTRA_BIN),$(PKG_INSTALL_DIR)/opt/bin/$(bin)) $(1)/opt/bin/
endef

define Package/pgsql-server/install
	$(INSTALL_DIR) $(1)/opt/bin
	$(INSTALL_BIN) $(foreach bin,$(PGSQL_SERVER_BIN),$(PKG_INSTALL_DIR)/opt/bin/$(bin)) $(1)/opt/bin/

	ln -sf postgres $(1)/opt/bin/postmaster

	$(INSTALL_DIR) $(1)/opt/share/postgresql
	$(CP) $(PKG_INSTALL_DIR)/opt/share/postgresql/* \
		$(1)/opt/share/postgresql

	$(INSTALL_DIR) $(1)/opt/lib
	$(CP) $(PKG_INSTALL_DIR)/opt/lib/postgresql \
		$(1)/opt/lib/

	$(INSTALL_DIR) $(1)/opt/etc/init.d
	$(INSTALL_BIN) ./files/S98postgresql $(1)/opt/etc/init.d/
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/opt/bin
	$(CP) $(STAGING_DIR_HOSTPKG)/lib/pg_config $(1)/opt/bin
	$(INSTALL_DIR) $(1)/host/bin/
	$(LN) $(STAGING_DIR)/opt/bin/pg_config $(1)/host/bin
	$(INSTALL_DIR) $(1)/opt/include
	$(CP) $(PKG_INSTALL_DIR)/opt/include/libpq $(1)/opt/include/
	$(CP) $(PKG_INSTALL_DIR)/opt/include/libpq-fe.h $(1)/opt/include/
	$(CP) $(PKG_INSTALL_DIR)/opt/include/pg_config.h $(1)/opt/include/
	$(CP) $(PKG_INSTALL_DIR)/opt/include/pg_config_manual.h $(1)/opt/include/
	$(CP) $(PKG_INSTALL_DIR)/opt/include/postgres_ext.h $(1)/opt/include/
	$(CP) $(PKG_INSTALL_DIR)/opt/include/pg_config_ext.h $(1)/opt/include/
	$(CP) $(PKG_INSTALL_DIR)/opt/include/postgresql $(1)/opt/include/
	$(INSTALL_DIR) $(1)/opt/lib
	$(CP) $(PKG_INSTALL_DIR)/opt/lib/libpq.{a,so*} $(1)/opt/lib/
	$(INSTALL_DIR) $(1)/opt/lib/pkgconfig
	$(CP) $(PKG_INSTALL_DIR)/opt/lib/pkgconfig/libpq.pc $(1)/opt/lib/pkgconfig/
endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,libpq))
$(eval $(call BuildPackage,pgsql-cli))
$(eval $(call BuildPackage,pgsql-cli-extra))
$(eval $(call BuildPackage,pgsql-server))
