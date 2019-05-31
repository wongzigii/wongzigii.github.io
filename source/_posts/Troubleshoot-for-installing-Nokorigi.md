title: Troubleshoot Installing Nokorigi
date: 2016-01-05 18:37:14
tags: ruby
---

Troubleshoot installing Nokorigi for the dependency of rails.

````bash
   ERROR: Error installing rails:
   ERROR: Failed to build gem native extension.
   current directory: /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/ext/nokogiri
   /Users/Wongzigii/.rvm/rubies/ruby-2.3.0/bin/ruby -r ./siteconf20160105-5281-1xpczbo.rb extconf.rb
   checking if the C compiler accepts ... yes
   checking if the C compiler accepts -Wno-error=unused-command-line-argument-hard-error-in-future... no
   Building nokogiri using packaged libraries.
   Using mini_portile version 2.0.0
   checking for iconv.h... yes
   checking for gzdopen() in -lz... yes
   checking for iconv using --with-opt-* flags... yes
   ************************************************************************
   IMPORTANT NOTICE:
	Building Nokogiri with a packaged version of libxml2-2.9.2
	with the following patches applied:
	- 0001-Revert-Missing-initialization-for-the-catalog-module.patch
	- 0002-Fix-missing-entities-after-CVE-2014-3660-fix.patch
	- 0003-Stop-parsing-on-entities-boundaries-errors.patch
	- 0004-Cleanup-conditional-section-error-handling.patch
	- 0005-CVE-2015-1819-Enforce-the-reader-to-run-in-constant-.patch
	- 0006-Another-variation-of-overflow-in-Conditional-section.patch
	- 0007-Fix-an-error-in-previous-Conditional-section-patch.patch
	- 0008-CVE-2015-8035-Fix-XZ-compression-support-loop.patch
	- 0009-Updated-config.guess.patch
	- 0010-Fix-parsering-short-unclosed-comment-uninitialized-access.patch
	- 0011-Avoid-extra-processing-of-MarkupDecl-when-EOF.patch
	- 0012-Avoid-processing-entities-after-encoding-conversion-.patch
	- 0013-CVE-2015-7497-Avoid-an-heap-buffer-overflow-in-xmlDi.patch
	- 0014-CVE-2015-5312-Another-entity-expansion-issue.patch
	- 0015-Add-xmlHaltParser-to-stop-the-parser.patch
	- 0016-Detect-incoherency-on-GROW.patch
	- 0017-CVE-2015-7500-Fix-memory-access-error-due-to-incorre.patch
	- 0018-CVE-2015-8242-Buffer-overead-with-HTML-parser-in-pus.patch

	Team Nokogiri will keep on doing their best to provide security
	updates in a timely manner, but if this is a concern for you and want
	to use the system library instead; abort this installation process and
	reinstall nokogiri as follows:

    gem install nokogiri -- --use-system-libraries
        [--with-xml2-config=/path/to/xml2-config]
        [--with-xslt-config=/path/to/xslt-config]

	If you are using Bundler, tell it to use the option:

    bundle config build.nokogiri --use-system-libraries
    bundle install

	Note, however, that nokogiri is not fully compatible with arbitrary
	versions of libxml2 provided by OS/package vendors.
	************************************************************************
	Extracting libxml2-2.9.2.tar.gz into tmp/x86_64-apple-darwin15.2.0/ports/libxml2/2.9.2... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0001-Revert-Missing-initialization-for-the-catalog-module.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0002-Fix-missing-entities-after-CVE-2014-3660-fix.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0003-Stop-parsing-on-entities-boundaries-errors.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0004-Cleanup-conditional-section-error-handling.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0005-CVE-2015-1819-Enforce-the-reader-to-run-in-constant-.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0006-Another-variation-of-overflow-in-Conditional-section.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0007-Fix-an-error-in-previous-Conditional-section-patch.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0008-CVE-2015-8035-Fix-XZ-compression-support-loop.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0009-Updated-config.guess.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0010-Fix-parsering-short-unclosed-comment-uninitialized-access.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0011-Avoid-extra-processing-of-MarkupDecl-when-EOF.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0012-Avoid-processing-entities-after-encoding-conversion-.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0013-CVE-2015-7497-Avoid-an-heap-buffer-overflow-in-xmlDi.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0014-CVE-2015-5312-Another-entity-expansion-issue.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0015-Add-xmlHaltParser-to-stop-the-parser.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0016-Detect-incoherency-on-GROW.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0017-CVE-2015-7500-Fix-memory-access-error-due-to-incorre.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxml2/0018-CVE-2015-8242-Buffer-overead-with-HTML-parser-in-pus.patch... OK
	Running 'configure' for libxml2 2.9.2... OK
	Running 'compile' for libxml2 2.9.2... OK
	Running 'install' for libxml2 2.9.2... OK
	Activating libxml2 2.9.2 (from /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/ports/x86_64-apple-darwin15.2.0/libxml2/2.9.2)...
	************************************************************************
	IMPORTANT NOTICE:

	Building Nokogiri with a packaged version of libxslt-1.1.28
	with the following patches applied:
	- 0001-Adding-doc-update-related-to-1.1.28.patch
	- 0002-Fix-a-couple-of-places-where-f-printf-parameters-wer.patch
	- 0003-Initialize-pseudo-random-number-generator-with-curre.patch
	- 0004-EXSLT-function-str-replace-is-broken-as-is.patch
	- 0006-Fix-str-padding-to-work-with-UTF-8-strings.patch
	- 0007-Separate-function-for-predicate-matching-in-patterns.patch
	- 0008-Fix-direct-pattern-matching.patch
	- 0009-Fix-certain-patterns-with-predicates.patch
	- 0010-Fix-handling-of-UTF-8-strings-in-EXSLT-crypto-module.patch
	- 0013-Memory-leak-in-xsltCompileIdKeyPattern-error-path.patch
	- 0014-Fix-for-bug-436589.patch
	- 0015-Fix-mkdir-for-mingw.patch
	- 0016-Fix-for-type-confusion-in-preprocessing-attributes.patch
	- 0017-Updated-config.guess.patch

	Team Nokogiri will keep on doing their best to provide security
	updates in a timely manner, but if this is a concern for you and want
	to use the system library instead; abort this installation process and
	reinstall nokogiri as follows:

    gem install nokogiri -- --use-system-libraries
        [--with-xml2-config=/path/to/xml2-config]
        [--with-xslt-config=/path/to/xslt-config]

	If you are using Bundler, tell it to use the option:

    bundle config build.nokogiri --use-system-libraries
    bundle install
	************************************************************************
	Extracting libxslt-1.1.28.tar.gz into tmp/x86_64-apple-darwin15.2.0/ports/libxslt/1.1.28... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0001-Adding-doc-update-related-to-1.1.28.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0002-Fix-a-couple-of-places-where-f-printf-parameters-wer.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0003-Initialize-pseudo-random-number-generator-with-curre.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0004-EXSLT-function-str-replace-is-broken-as-is.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0006-Fix-str-padding-to-work-with-UTF-8-strings.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0007-Separate-function-for-predicate-matching-in-patterns.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0008-Fix-direct-pattern-matching.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0009-Fix-certain-patterns-with-predicates.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0010-Fix-handling-of-UTF-8-strings-in-EXSLT-crypto-module.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0013-Memory-leak-in-xsltCompileIdKeyPattern-error-path.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0014-Fix-for-bug-436589.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0015-Fix-mkdir-for-mingw.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0016-Fix-for-type-confusion-in-preprocessing-attributes.patch... OK
	Running git apply with /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/patches/libxslt/0017-Updated-config.guess.patch... OK
	Running 'configure' for libxslt 1.1.28... ERROR, review '/Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1/ext/nokogiri/tmp/x86_64-apple-darwin15.2.0/ports/libxslt/1.1.28/configure.log' to see what happened. Last lines are:
	========================================================================
	 Darwin Kernel Version 15.2.0: Fri Nov 13 19:56:56 PST 2015; root:xnu-3248.20.55~2/RELEASE_X86_64
	Kernel configured for up to 8 processors.
	4 processors are physically available.
	8 processors are logically available.
	Processor type: x86_64h (Intel x86-64h Haswell)
	Processors active: 0 1 2 3 4 5 6 7
	Primary memory available: 16.00 gigabytes
	Default processor set: 376 tasks, 1947 threads, 8 processors
	Load average: 1.84, Mach factor: 6.14
	/bin/universe          =
	/usr/bin/arch -k       =
	/bin/arch              =
	/usr/bin/oslevel       =
	/usr/convex/getsysinfo =

	UNAME_MACHINE = x86_64
	UNAME_RELEASE = 15.2.0
	UNAME_SYSTEM  = Darwin
	UNAME_VERSION = Darwin Kernel Version 15.2.0: Fri Nov 13 19:56:56 PST 2015; root:xnu-3248.20.55~2/RELEASE_X86_64
	configure: error: cannot guess build type; you must specify one
	========================================================================
	*** extconf.rb failed ***
	Could not create Makefile due to some reason, probably lack of necessary
	libraries and/or headers.  Check the mkmf.log file for more details.  You may
	need configuration options.

	Provided configuration options:
	--with-opt-dir
	--with-opt-include
	--without-opt-include=${opt-dir}/include
	--with-opt-lib
	--without-opt-lib=${opt-dir}/lib
	--with-make-prog
	--without-make-prog
	--srcdir=.
	--curdir
	--ruby=/Users/Wongzigii/.rvm/rubies/ruby-2.3.0/bin/$(RUBY_BASE_NAME)
	--help
	--clean
	--use-system-libraries
	--enable-static
	--disable-static
	--with-zlib-dir
	--without-zlib-dir
	--with-zlib-include
	--without-zlib-include=${zlib-dir}/include
	--with-zlib-lib
	--without-zlib-lib=${zlib-dir}/lib
	--enable-cross-build
	--disable-cross-build
	/Users/Wongzigii/.rvm/rubies/ruby-2.3.0/lib/ruby/gems/2.3.0/gems/mini_portile2-2.0.0/lib/mini_portile2/mini_portile.rb:366:in `block in execute': Failed to complete configure task (RuntimeError)
	from /Users/Wongzigii/.rvm/rubies/ruby-2.3.0/lib/ruby/gems/2.3.0/gems/mini_portile2-2.0.0/lib/mini_portile2/mini_portile.rb:337:in `chdir'
	from /Users/Wongzigii/.rvm/rubies/ruby-2.3.0/lib/ruby/gems/2.3.0/gems/mini_portile2-2.0.0/lib/mini_portile2/mini_portile.rb:337:in `execute'
	from /Users/Wongzigii/.rvm/rubies/ruby-2.3.0/lib/ruby/gems/2.3.0/gems/mini_portile2-2.0.0/lib/mini_portile2/mini_portile.rb:106:in `configure'
	from /Users/Wongzigii/.rvm/rubies/ruby-2.3.0/lib/ruby/gems/2.3.0/gems/mini_portile2-2.0.0/lib/mini_portile2/mini_portile.rb:149:in `cook'
	from extconf.rb:289:in `block (2 levels) in process_recipe'
	from extconf.rb:182:in `block in chdir_for_build'
	from extconf.rb:181:in `chdir'
	from extconf.rb:181:in `chdir_for_build'
	from extconf.rb:288:in `block in process_recipe'
	from extconf.rb:187:in `tap'
	from extconf.rb:187:in `process_recipe'
	from extconf.rb:490:in `<main>'

	To see why this extension failed to compile, please check the mkmf.log which can be found here:

  	/Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/extensions/x86_64-darwin-15/2.3.0/nokogiri-1.6.7.1/mkmf.log

	extconf failed, exit code 1

	Gem files will remain installed in /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/gems/nokogiri-1.6.7.1 for inspection.
	Results logged to /Users/Wongzigii/.rvm/gems/ruby-2.3.0@rails4.2/extensions/x86_64-darwin-15/2.3.0/nokogiri-1.6.7.1/gem_make.out
````

**The Problem:**

> For some reason Apple’s Yosemite version of OSX does not have a system accessible installation of libxml2.  Nokogiri requires this in order to compile and luckily Xcode has a version of libxml2 bundled with it — we just need to specify it when installing the gem.  It’s important to get Nokogiri installed correctly because as of right now Rails 4.2.1.rc4 automatically attempts to install it and you will feel pain.

**Solved:**

````bash
gem install nokogiri -- --use-system-libraries=true --with-xml2-include=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk/usr/include/libxml2/
````



