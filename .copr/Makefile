# Copr "make_srpm" SCM source method entry point. Invoked by Copr's
# builder as `make -f .copr/Makefile srpm outdir=<dir> spec=<path>`,
# already inside a clone of this repo at the configured commit/branch.
# Produces an SRPM for `outdir` from HEAD, using the spec+patches
# committed alongside this Makefile under .copr/. Mirrors what
# make-source-tarball.sh + build-all.sh's SRPM step do in the local/CI
# pipeline (see github.com/ykne/GNOME-X11-specs), scoped to this package.
#
# gvc and jasmine-gjs are real meson wrap subprojects, gitignored on disk
# but declared via subprojects/*.wrap - `git archive` alone drops them,
# so they're fetched and vendored into the source tarball explicitly.
# make_srpm runs as root with network access, so meson is installed here
# rather than relying on any project-level buildroot package list.

PKG := gnome-shell
VERSION := 50.4

.PHONY: srpm
srpm:
	set -e; \
	dnf install -y meson; \
	meson subprojects download gvc jasmine-gjs; \
	hash=$$(git rev-parse --short=8 HEAD); \
	workdir=$$(mktemp -d); \
	mkdir -p "$$workdir/$(PKG)-$(VERSION)/subprojects" "$$workdir/SOURCES"; \
	git archive HEAD | tar -x -C "$$workdir/$(PKG)-$(VERSION)"; \
	cp -r subprojects/gvc subprojects/jasmine-gjs "$$workdir/$(PKG)-$(VERSION)/subprojects/"; \
	tar -cJf "$$workdir/SOURCES/$(PKG)-$(VERSION).tar.xz" -C "$$workdir" "$(PKG)-$(VERSION)"; \
	cp .copr/SOURCES/* "$$workdir/SOURCES/" 2>/dev/null || true; \
	sed "s/^Release:.*\$$/Release:        1.g$${hash}%{?dist}/" "$(spec)" > "$$workdir/$(PKG).spec"; \
	rpmbuild -bs \
		--define "_topdir $$workdir" \
		--define "_sourcedir $$workdir/SOURCES" \
		--define "_specdir $$workdir" \
		--define "_builddir $$workdir/BUILD" \
		--define "_srcrpmdir $(outdir)" \
		"$$workdir/$(PKG).spec"; \
	rm -rf "$$workdir"
