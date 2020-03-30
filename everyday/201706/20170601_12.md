[TOC]

# linux svn install

linux svn install

最近一段时间，需要同步到文件很多东西，那么就学习在linux搭建一套svn服务器

环境介绍

​     操作系统版本

 [root@qbname ~]# cat /etc/issue

 Red Hat Enterprise Linux Server release 6.5 (Santiago)

​         

​         

​         svn版本

 [root@centos65 ~]# svn --version

 svn, version 1.6.11 (r934486)

   compiled Apr 11 2013, 16:13:51

安装步骤

使用yum安装svn

[root@qbname ~]# yum -y install subversion

Loaded plugins: product-id, refresh-packagekit, security, subscription-manager

This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.

Setting up Install Process

Resolving Dependencies

--> Running transaction check

---> Package subversion.x86_64 0:1.6.11-9.el6_4 will be installed

--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================================================

 Package                                           Arch                                          Version                                                Repository                                  Size

=========================================================================================================================================================================================================

Installing:

 subversion                                        x86_64                                        1.6.11-9.el6_4                                         dvd                                        2.3 M

Transaction Summary

=========================================================================================================================================================================================================

Install       1 Package(s)

Total download size: 2.3 M

Installed size: 12 M

Downloading Packages:

subversion-1.6.11-9.el6_4.x86_64.rpm                                                                                                                                              | 2.3 MB     00:00     

Running rpm_check_debug

Running Transaction Test

Transaction Test Succeeded

Running Transaction

  Installing : subversion-1.6.11-9.el6_4.x86_64                                                                                                                                                      1/1 

  Verifying  : subversion-1.6.11-9.el6_4.x86_64                                                                                                                                                      1/1 

Installed:

  subversion.x86_64 0:1.6.11-9.el6_4                                                                                                                                                                     

Complete!

使用rpm查看svn安装信息

[root@qbname ~]# rpm -ql subversion

/etc/bash_completion.d

/etc/bash_completion.d/subversion

/etc/rc.d/init.d/svnserve

/etc/subversion

/usr/bin/svn

/usr/bin/svnadmin

/usr/bin/svndumpfilter

/usr/bin/svnlook

/usr/bin/svnserve

/usr/bin/svnsync

/usr/bin/svnversion

/usr/lib64/libsvn_client-1.so.0

/usr/lib64/libsvn_client-1.so.0.0.0

/usr/lib64/libsvn_delta-1.so.0

/usr/lib64/libsvn_delta-1.so.0.0.0

/usr/lib64/libsvn_diff-1.so.0

/usr/lib64/libsvn_diff-1.so.0.0.0

/usr/lib64/libsvn_fs-1.so.0

/usr/lib64/libsvn_fs-1.so.0.0.0

/usr/lib64/libsvn_fs_base-1.so.0

/usr/lib64/libsvn_fs_base-1.so.0.0.0

/usr/lib64/libsvn_fs_fs-1.so.0

/usr/lib64/libsvn_fs_fs-1.so.0.0.0

/usr/lib64/libsvn_fs_util-1.so.0

/usr/lib64/libsvn_fs_util-1.so.0.0.0

/usr/lib64/libsvn_ra-1.so.0

/usr/lib64/libsvn_ra-1.so.0.0.0

/usr/lib64/libsvn_ra_local-1.so.0

/usr/lib64/libsvn_ra_local-1.so.0.0.0

/usr/lib64/libsvn_ra_neon-1.so.0

/usr/lib64/libsvn_ra_neon-1.so.0.0.0

/usr/lib64/libsvn_ra_svn-1.so.0

/usr/lib64/libsvn_ra_svn-1.so.0.0.0

/usr/lib64/libsvn_repos-1.so.0

/usr/lib64/libsvn_repos-1.so.0.0.0

/usr/lib64/libsvn_subr-1.so.0

/usr/lib64/libsvn_subr-1.so.0.0.0

/usr/lib64/libsvn_swig_py-1.so.0

/usr/lib64/libsvn_swig_py-1.so.0.0.0

/usr/lib64/libsvn_wc-1.so.0

/usr/lib64/libsvn_wc-1.so.0.0.0

/usr/lib64/python2.6/site-packages/libsvn

/usr/lib64/python2.6/site-packages/libsvn/__init__.py

/usr/lib64/python2.6/site-packages/libsvn/__init__.pyc

/usr/lib64/python2.6/site-packages/libsvn/__init__.pyo

/usr/lib64/python2.6/site-packages/libsvn/_client.so

/usr/lib64/python2.6/site-packages/libsvn/_core.so

/usr/lib64/python2.6/site-packages/libsvn/_delta.so

/usr/lib64/python2.6/site-packages/libsvn/_diff.so

/usr/lib64/python2.6/site-packages/libsvn/_fs.so

/usr/lib64/python2.6/site-packages/libsvn/_ra.so

/usr/lib64/python2.6/site-packages/libsvn/_repos.so

/usr/lib64/python2.6/site-packages/libsvn/_wc.so

/usr/lib64/python2.6/site-packages/libsvn/client.py

/usr/lib64/python2.6/site-packages/libsvn/client.pyc

/usr/lib64/python2.6/site-packages/libsvn/client.pyo

/usr/lib64/python2.6/site-packages/libsvn/core.py

/usr/lib64/python2.6/site-packages/libsvn/core.pyc

/usr/lib64/python2.6/site-packages/libsvn/core.pyo

/usr/lib64/python2.6/site-packages/libsvn/delta.py

/usr/lib64/python2.6/site-packages/libsvn/delta.pyc

/usr/lib64/python2.6/site-packages/libsvn/delta.pyo

/usr/lib64/python2.6/site-packages/libsvn/diff.py

/usr/lib64/python2.6/site-packages/libsvn/diff.pyc

/usr/lib64/python2.6/site-packages/libsvn/diff.pyo

/usr/lib64/python2.6/site-packages/libsvn/fs.py

/usr/lib64/python2.6/site-packages/libsvn/fs.pyc

/usr/lib64/python2.6/site-packages/libsvn/fs.pyo

/usr/lib64/python2.6/site-packages/libsvn/ra.py

/usr/lib64/python2.6/site-packages/libsvn/ra.pyc

/usr/lib64/python2.6/site-packages/libsvn/ra.pyo

/usr/lib64/python2.6/site-packages/libsvn/repos.py

/usr/lib64/python2.6/site-packages/libsvn/repos.pyc

/usr/lib64/python2.6/site-packages/libsvn/repos.pyo

/usr/lib64/python2.6/site-packages/libsvn/wc.py

/usr/lib64/python2.6/site-packages/libsvn/wc.pyc

/usr/lib64/python2.6/site-packages/libsvn/wc.pyo

/usr/lib64/python2.6/site-packages/svn

/usr/lib64/python2.6/site-packages/svn/__init__.py

/usr/lib64/python2.6/site-packages/svn/__init__.pyc

/usr/lib64/python2.6/site-packages/svn/__init__.pyo

/usr/lib64/python2.6/site-packages/svn/client.py

/usr/lib64/python2.6/site-packages/svn/client.pyc

/usr/lib64/python2.6/site-packages/svn/client.pyo

/usr/lib64/python2.6/site-packages/svn/core.py

/usr/lib64/python2.6/site-packages/svn/core.pyc

/usr/lib64/python2.6/site-packages/svn/core.pyo

/usr/lib64/python2.6/site-packages/svn/delta.py

/usr/lib64/python2.6/site-packages/svn/delta.pyc

/usr/lib64/python2.6/site-packages/svn/delta.pyo

/usr/lib64/python2.6/site-packages/svn/diff.py

/usr/lib64/python2.6/site-packages/svn/diff.pyc

/usr/lib64/python2.6/site-packages/svn/diff.pyo

/usr/lib64/python2.6/site-packages/svn/fs.py

/usr/lib64/python2.6/site-packages/svn/fs.pyc

/usr/lib64/python2.6/site-packages/svn/fs.pyo

/usr/lib64/python2.6/site-packages/svn/ra.py

/usr/lib64/python2.6/site-packages/svn/ra.pyc

/usr/lib64/python2.6/site-packages/svn/ra.pyo

/usr/lib64/python2.6/site-packages/svn/repos.py

/usr/lib64/python2.6/site-packages/svn/repos.pyc

/usr/lib64/python2.6/site-packages/svn/repos.pyo

/usr/lib64/python2.6/site-packages/svn/wc.py

/usr/lib64/python2.6/site-packages/svn/wc.pyc

/usr/lib64/python2.6/site-packages/svn/wc.pyo

/usr/share/doc/subversion-1.6.11

/usr/share/doc/subversion-1.6.11/BUGS

/usr/share/doc/subversion-1.6.11/CHANGES

/usr/share/doc/subversion-1.6.11/COMMITTERS

/usr/share/doc/subversion-1.6.11/COPYING

/usr/share/doc/subversion-1.6.11/HACKING

/usr/share/doc/subversion-1.6.11/INSTALL

/usr/share/doc/subversion-1.6.11/LICENSE

/usr/share/doc/subversion-1.6.11/README

/usr/share/doc/subversion-1.6.11/mod_authz_svn-INSTALL

/usr/share/doc/subversion-1.6.11/svn_load_dirs.README

/usr/share/doc/subversion-1.6.11/svn_load_dirs.pl

/usr/share/doc/subversion-1.6.11/svn_load_dirs_property_table.example

/usr/share/doc/subversion-1.6.11/svnmerge-migrate-history-remotely.py

/usr/share/doc/subversion-1.6.11/svnmerge-migrate-history.py

/usr/share/doc/subversion-1.6.11/svnmerge.README

/usr/share/doc/subversion-1.6.11/svnmerge.py

/usr/share/doc/subversion-1.6.11/svnmerge_test.py

/usr/share/doc/subversion-1.6.11/tools

/usr/share/doc/subversion-1.6.11/tools/README

/usr/share/doc/subversion-1.6.11/tools/backup

/usr/share/doc/subversion-1.6.11/tools/backup/hot-backup.py

/usr/share/doc/subversion-1.6.11/tools/bdb

/usr/share/doc/subversion-1.6.11/tools/bdb/erase-all-text-data.py

/usr/share/doc/subversion-1.6.11/tools/bdb/skel.py

/usr/share/doc/subversion-1.6.11/tools/bdb/svn-bdb-view.py

/usr/share/doc/subversion-1.6.11/tools/bdb/svnfs.py

/usr/share/doc/subversion-1.6.11/tools/bdb/whatis-rep.py

/usr/share/doc/subversion-1.6.11/tools/buildbot

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/README

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/i686-debian-sarge1

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/i686-debian-sarge1/mount-ramdrive.c

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/i686-debian-sarge1/svnbuild.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/i686-debian-sarge1/svncheck.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/i686-debian-sarge1/svnclean.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/i686-debian-sarge1/svnlog.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/i686-debian-sarge1/unmount-ramdrive.c

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/osx10.4-gcc4.0.1-ia32

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/osx10.4-gcc4.0.1-ia32/svnbuild.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/osx10.4-gcc4.0.1-ia32/svncheck.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/osx10.4-gcc4.0.1-ia32/svnclean.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/osx10.4-gcc4.0.1-ia32/svnlog.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/ubuntu-x64

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/ubuntu-x64/mount-ramdrive.c

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/ubuntu-x64/svnbuild.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/ubuntu-x64/svncheck-bindings.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/ubuntu-x64/svncheck.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/ubuntu-x64/svnclean.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/ubuntu-x64/svnlog.sh

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/ubuntu-x64/unmount-ramdrive.c

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/win32-xp-VS2005

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/win32-xp-VS2005/config.bat

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/win32-xp-VS2005/do_all.bat

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/win32-xp-VS2005/svnbuild.bat

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/win32-xp-VS2005/svncheck.bat

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/win32-xp-VS2005/svnclean.bat

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/win32-xp-VS2005/svnlog.bat

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/xp-vc60-ia32

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/xp-vc60-ia32/config.bat.tmpl

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/xp-vc60-ia32/svnbuild.bat

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/xp-vc60-ia32/svncheck.bat

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/xp-vc60-ia32/svnclean.bat

/usr/share/doc/subversion-1.6.11/tools/buildbot/slaves/xp-vc60-ia32/svnlog.bat

/usr/share/doc/subversion-1.6.11/tools/client-side

/usr/share/doc/subversion-1.6.11/tools/client-side/bash_completion_test

/usr/share/doc/subversion-1.6.11/tools/client-side/change-svn-wc-format.py

/usr/share/doc/subversion-1.6.11/tools/client-side/server-version.py

/usr/share/doc/subversion-1.6.11/tools/client-side/showchange.pl

/usr/share/doc/subversion-1.6.11/tools/client-side/svn-graph.pl

/usr/share/doc/subversion-1.6.11/tools/client-side/svnmucc

/usr/share/doc/subversion-1.6.11/tools/client-side/svnmucc/svnmucc-test.py

/usr/share/doc/subversion-1.6.11/tools/client-side/svnmucc/svnmucc.c

/usr/share/doc/subversion-1.6.11/tools/client-side/wcfind

/usr/share/doc/subversion-1.6.11/tools/dev

/usr/share/doc/subversion-1.6.11/tools/dev/check-license.py

/usr/share/doc/subversion-1.6.11/tools/dev/contribulyze.py

/usr/share/doc/subversion-1.6.11/tools/dev/datecheck.py

/usr/share/doc/subversion-1.6.11/tools/dev/find-bad-style.py

/usr/share/doc/subversion-1.6.11/tools/dev/gcov.patch

/usr/share/doc/subversion-1.6.11/tools/dev/gen-javahl-errors.py

/usr/share/doc/subversion-1.6.11/tools/dev/gnuify-changelog.pl

/usr/share/doc/subversion-1.6.11/tools/dev/graph-dav-servers.py

/usr/share/doc/subversion-1.6.11/tools/dev/iz

/usr/share/doc/subversion-1.6.11/tools/dev/iz/defect.dem

/usr/share/doc/subversion-1.6.11/tools/dev/iz/ff2csv.command

/usr/share/doc/subversion-1.6.11/tools/dev/iz/ff2csv.py

/usr/share/doc/subversion-1.6.11/tools/dev/iz/find-fix.py

/usr/share/doc/subversion-1.6.11/tools/dev/iz/run-queries.sh

/usr/share/doc/subversion-1.6.11/tools/dev/lock-check.py

/usr/share/doc/subversion-1.6.11/tools/dev/min-includes.sh

/usr/share/doc/subversion-1.6.11/tools/dev/mlpatch.py

/usr/share/doc/subversion-1.6.11/tools/dev/normalize-dump.py

/usr/share/doc/subversion-1.6.11/tools/dev/po-merge.py

/usr/share/doc/subversion-1.6.11/tools/dev/prebuild-cleanup.sh

/usr/share/doc/subversion-1.6.11/tools/dev/random-commits.py

/usr/share/doc/subversion-1.6.11/tools/dev/scramble-tree.py

/usr/share/doc/subversion-1.6.11/tools/dev/stress.pl

/usr/share/doc/subversion-1.6.11/tools/dev/svn-dev.el

/usr/share/doc/subversion-1.6.11/tools/dev/svn-dev.vim

/usr/share/doc/subversion-1.6.11/tools/dev/svn-entries.el

/usr/share/doc/subversion-1.6.11/tools/dev/svn-merge-revs.py

/usr/share/doc/subversion-1.6.11/tools/dev/trails.py

/usr/share/doc/subversion-1.6.11/tools/dev/verify-history.py

/usr/share/doc/subversion-1.6.11/tools/dev/warn-ignored-err.sh

/usr/share/doc/subversion-1.6.11/tools/dev/which-error.py

/usr/share/doc/subversion-1.6.11/tools/diff

/usr/share/doc/subversion-1.6.11/tools/diff/diff.c

/usr/share/doc/subversion-1.6.11/tools/diff/diff3.c

/usr/share/doc/subversion-1.6.11/tools/diff/diff4.c

/usr/share/doc/subversion-1.6.11/tools/dist

/usr/share/doc/subversion-1.6.11/tools/dist/construct-rolling-environment.sh

/usr/share/doc/subversion-1.6.11/tools/dist/dist.sh

/usr/share/doc/subversion-1.6.11/tools/dist/download-release.sh

/usr/share/doc/subversion-1.6.11/tools/dist/extract-for-examination.sh

/usr/share/doc/subversion-1.6.11/tools/dist/gen_nightly_ann.py

/usr/share/doc/subversion-1.6.11/tools/dist/getsigs.py

/usr/share/doc/subversion-1.6.11/tools/dist/nightly.sh

/usr/share/doc/subversion-1.6.11/tools/dist/post-to-tigris.py

/usr/share/doc/subversion-1.6.11/tools/dist/roll.sh

/usr/share/doc/subversion-1.6.11/tools/dist/sample-rc-warning-index.html

/usr/share/doc/subversion-1.6.11/tools/dist/test.sh

/usr/share/doc/subversion-1.6.11/tools/dist/write-announcement.py

/usr/share/doc/subversion-1.6.11/tools/examples

/usr/share/doc/subversion-1.6.11/tools/examples/SvnCLBrowse

/usr/share/doc/subversion-1.6.11/tools/examples/blame.py

/usr/share/doc/subversion-1.6.11/tools/examples/check-modified.py

/usr/share/doc/subversion-1.6.11/tools/examples/dumpprops.py

/usr/share/doc/subversion-1.6.11/tools/examples/get-location-segments.py

/usr/share/doc/subversion-1.6.11/tools/examples/getfile.py

/usr/share/doc/subversion-1.6.11/tools/examples/getlocks_test.c

/usr/share/doc/subversion-1.6.11/tools/examples/geturl.py

/usr/share/doc/subversion-1.6.11/tools/examples/headrev.c

/usr/share/doc/subversion-1.6.11/tools/examples/minimal_client.c

/usr/share/doc/subversion-1.6.11/tools/examples/putfile.py

/usr/share/doc/subversion-1.6.11/tools/examples/revplist.py

/usr/share/doc/subversion-1.6.11/tools/examples/svnlog2html.rb

/usr/share/doc/subversion-1.6.11/tools/examples/svnlook.py

/usr/share/doc/subversion-1.6.11/tools/examples/svnlook.rb

/usr/share/doc/subversion-1.6.11/tools/examples/svnput.c

/usr/share/doc/subversion-1.6.11/tools/examples/svnserve-sgid.c

/usr/share/doc/subversion-1.6.11/tools/examples/svnshell.py

/usr/share/doc/subversion-1.6.11/tools/examples/svnshell.rb

/usr/share/doc/subversion-1.6.11/tools/examples/testwrite.c

/usr/share/doc/subversion-1.6.11/tools/hook-scripts

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/README

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/commit-access-control.cfg.example

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/commit-access-control.pl

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/commit-email.rb

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/log-police.py

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/mailer

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/mailer/mailer.conf.example

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/mailer/mailer.py

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/mailer/tests

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/mailer/tests/mailer-init.sh

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/mailer/tests/mailer-t1.output

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/mailer/tests/mailer-t1.sh

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/mailer/tests/mailer-tweak.py

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/mailer/tests/mailer.conf

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/svn2feed.py

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/svnperms.conf.example

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/svnperms.py

/usr/share/doc/subversion-1.6.11/tools/hook-scripts/verify-po.py

/usr/share/doc/subversion-1.6.11/tools/po

/usr/share/doc/subversion-1.6.11/tools/po/l10n-report.py

/usr/share/doc/subversion-1.6.11/tools/po/po-update.sh

/usr/share/doc/subversion-1.6.11/tools/server-side

/usr/share/doc/subversion-1.6.11/tools/server-side/fsfs-reshard.py

/usr/share/doc/subversion-1.6.11/tools/server-side/svn-backup-dumps.py

/usr/share/doc/subversion-1.6.11/tools/server-side/svn-populate-node-origins-index.c

/usr/share/doc/subversion-1.6.11/tools/server-side/svn_server_log_parse.py

/usr/share/doc/subversion-1.6.11/tools/server-side/svnauthz-validate.c

/usr/share/doc/subversion-1.6.11/tools/server-side/test_svn_server_log_parse.py

/usr/share/doc/subversion-1.6.11/tools/xslt

/usr/share/doc/subversion-1.6.11/tools/xslt/svnindex.css

/usr/share/doc/subversion-1.6.11/tools/xslt/svnindex.xsl

/usr/share/doc/subversion-1.6.11/wcgrep

/usr/share/emacs/site-lisp/psvn-init.el

/usr/share/emacs/site-lisp/psvn.el

/usr/share/locale/de/LC_MESSAGES/subversion.mo

/usr/share/locale/es/LC_MESSAGES/subversion.mo

/usr/share/locale/fr/LC_MESSAGES/subversion.mo

/usr/share/locale/it/LC_MESSAGES/subversion.mo

/usr/share/locale/ja/LC_MESSAGES/subversion.mo

/usr/share/locale/ko/LC_MESSAGES/subversion.mo

/usr/share/locale/nb/LC_MESSAGES/subversion.mo

/usr/share/locale/pl/LC_MESSAGES/subversion.mo

/usr/share/locale/pt_BR/LC_MESSAGES/subversion.mo

/usr/share/locale/sv/LC_MESSAGES/subversion.mo

/usr/share/locale/zh_CN/LC_MESSAGES/subversion.mo

/usr/share/locale/zh_TW/LC_MESSAGES/subversion.mo

/usr/share/man/man1/svn.1.gz

/usr/share/man/man1/svnadmin.1.gz

/usr/share/man/man1/svndumpfilter.1.gz

/usr/share/man/man1/svnlook.1.gz

/usr/share/man/man1/svnsync.1.gz

/usr/share/man/man1/svnversion.1.gz

/usr/share/man/man5/svnserve.conf.5.gz

/usr/share/man/man8/svnserve.8.gz

/usr/share/xemacs/site-packages/lisp/psvn.el

在/var/目录下创建文件夹svn，在文件夹svn下创建svnrepos

[root@qbname ~]# mkdir /var/svn/svnrepos

可能会报错

​              创建访问路径

[root@qbname ~]# svnadmin create /var/svn/svnrepos/dragon_1111

到达 dragon_jfyq路径下的conf文件夹下

[root@qbname ~]# cd /var/svn/svnrepos/dragon_1111/

[root@qbname dragon_1111]# ls -ls

??? 24

4 drwxr-xr-x 2 root root 4096 6?  15 10:51 conf

4 drwxr-sr-x 6 root root 4096 6?  15 10:51 db

4 -r--r--r-- 1 root root    2 6?  15 10:51 format

4 drwxr-xr-x 2 root root 4096 6?  15 10:51 hooks

4 drwxr-xr-x 2 root root 4096 6?  15 10:51 locks

4 -rw-r--r-- 1 root root  229 6?  15 10:51 README.txt

[root@qbname dragon_1111# cd conf

[root@qbname conf]# ls -ls

??? 12

4 -rw-r--r-- 1 root root 1080 6?  15 10:51 authz

4 -rw-r--r-- 1 root root  309 6?  15 10:51 passwd

4 -rw-r--r-- 1 root root 2279 6?  15 10:51 svnserve.conf

分别修改authz,passwd,svnserve.conf文件

authz:登录用户名及权限

passwd：登录用户密码

svnserve.conf:服务控制

[root@qbname conf]# cat authz

\### This file is an example authorization file for svnserve.

\### Its format is identical to that of mod_authz_svn authorization

\### files.

\### As shown below each section defines authorizations for the path and

\### (optional) repository specified by the section name.

\### The authorizations follow. An authorization line can refer to:

\###  - a single user,

\###  - a group of users defined in a special [groups] section,

\###  - an alias defined in a special [aliases] section,

\###  - all authenticated users, using the '$authenticated' token,

\###  - only anonymous users, using the '$anonymous' token,

\###  - anyone, using the '*' wildcard.

\###

\### A match can be inverted by prefixing the rule with '~'. Rules can

\### grant read ('r') access, read-write ('rw') access, or no access

\### ('').

[aliases]

\# joe = /C=XZ/ST=Dessert/L=Snake City/O=Snake Oil, Ltd./OU=Research Institute/CN=Joe Average

[groups]

\# harry_and_sally = harry,sally

\# harry_sally_and_joe = harry,sally,&joe

\# [/foo/bar]

\# harry = rw

\# &joe = r

\# * =

\# [repository:/baz/fuz]

\# @harry_and_sally = rw

\# * = r

[\]

dragon_11 = rw

dragon_111 = rw

​             

​            [ root@qbname conf]# cat passwd 

\### This file is an example password file for svnserve.

\### Its format is similar to that of svnserve.conf. As shown in the

\### example below it contains one section labelled [users].

\### The name and password for each user follow, one account per line.

[users]

\# harry = harryssecret

\# sally = sallyssecret

dragon_11 =111111

dragon_111 = 111111

​             [root@qbname conf]# cat svnserve.conf 

\### This file controls the configuration of the svnserve daemon, if you

\### use it to allow access to this repository.  (If you only allow

\### access through http: and/or file: URLs, then this file is

\### irrelevant.)

\### Visit http://subversion.tigris.org/ for more information.

[general]

\### These options control access to the repository for unauthenticated

\### and authenticated users.  Valid values are "write", "read",

\### and "none".  The sample settings below are the defaults.

anon-access = read

auth-access = write

\### The password-db option controls the location of the password

\### database file.  Unless you specify a path starting with a /,

\### the file's location is relative to the directory containing

\### this configuration file.

\### If SASL is enabled (see below), this file will NOT be used.

\### Uncomment the line below to use the default password file.

password-db = passwd

\### The authz-db option controls the location of the authorization

\### rules for path-based access control.  Unless you specify a path

\### starting with a /, the file's location is relative to the the

\### directory containing this file.  If you don't specify an

\### authz-db, no path-based access control is done.

\### Uncomment the line below to use the default authorization file.

\# authz-db = authz

\### This option specifies the authentication realm of the repository.

\### If two repositories have the same authentication realm, they should

\### have the same password database, and vice versa.  The default realm

\### is repository's uuid.

realm = My First Repository

[sasl]

\### This option specifies whether you want to use the Cyrus SASL

\### library for authentication. Default is false.

\### This section will be ignored if svnserve is not built with Cyrus

\### SASL support; to check, run 'svnserve --version' and look for a line

\### reading 'Cyrus SASL authentication is available.'

\# use-sasl = true

\### These options specify the desired strength of the security layer

\### that you want SASL to provide. 0 means no encryption, 1 means

\### integrity-checking only, values larger than 1 are correlated

\### to the effective key length for encryption (e.g. 128 means 128-bit

\### encryption). The values below are the defaults.

\# min-encryption = 0

\# max-encryption = 256

标注颜色的为自己修改

启动

[root@qbname conf]# svnserve -d -r /var/svn/svnrepos/

​    

<http://www.cnblogs.com/mymelon/p/5483215.html>