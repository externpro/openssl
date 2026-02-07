# openssl dependencies

|project|license [^_l]|description [dependencies]|version|source|diff [^_d]|
|-------|-------------|--------------------------|-------|------|----------|
|<a id='openssl' />[openssl](http://www.openssl.org/)|[BSD-style](http://www.openssl.org/source/license.html 'dual OpenSSL and SSLeay License: both are BSD-style licenses')|Cryptography and SSL/TLS Toolkit [pvt deps: _nasm, yasm_]| |[upstream](https://github.com/openssl/openssl 'github.com/openssl/openssl')|  [intro]|
|<a id='nasm' />[nasm](https://www.nasm.us/)|[BSD-2-Clause](https://www.nasm.us/ 'BSD 2-Clause Simplified License')|The Netwide Assembler - an 80x86 and x86-64 assembler (MSW-only)|[xpv2.14.02.4](https://github.com/externpro/nasm/releases/tag/xpv2.14.02.4 'release')|[repo](https://github.com/externpro/nasm 'github.com/externpro/nasm')|[diff](https://github.com/externpro/nasm/compare/v0...xpv2.14.02.4 'github.com/externpro/nasm/compare/v0...xpv2.14.02.4') [bin]|
|<a id='yasm' />[yasm](http://yasm.tortall.net/)|[BSD-2-Clause](https://github.com/yasm/yasm/blob/v1.3.0/COPYING 'BSD 2-Clause Simplified License')|assembler and disassembler for the Intel x86 architecture|[xpv1.3.0.3](https://github.com/externpro/yasm/releases/tag/xpv1.3.0.3 'release')|[repo](https://github.com/externpro/yasm 'github.com/externpro/yasm') [upstream](https://github.com/yasm/yasm 'github.com/yasm/yasm')|[diff](https://github.com/externpro/yasm/compare/v1.3.0...xpv1.3.0.3 'github.com/externpro/yasm/compare/v1.3.0...xpv1.3.0.3') [patch]|

![deps](xprodeps.svg 'dependencies')

Dependency version check: all 2 parent-manifest versions match pinned versions.

|diff  |description|
|------|-----------|
|patch |diff modifies/patches existing cmake|
|intro |diff introduces cmake|
|auto  |diff adds cmake to replace autotools/configure/make|
|native|diff adds cmake but uses existing build system|
|bin   |diff adds cmake to repackage binaries built elsewhere|
|fetch |diff adds cmake and utilizes FetchContent|

[^_l]: see [SPDX License List](https://spdx.org/licenses/ '') for a list of commonly found licenses
[^_d]: see table above with description of diff
