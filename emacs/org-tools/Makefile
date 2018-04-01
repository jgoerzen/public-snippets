.DEFAULT: sync
.PHONY: all
all: sync

.PHONY: clean
clean:
	git clean -f -X

GITOUTPUT := 2>&1 | egrep -v '^(gpg: Signature made|gpg:.*using RSA key|gpg: Good signature from|gcrypt: Decrypting manifest|gcrypt: Encrypting to|gcrypt: Requesting manifest signature)'

.PHONY: branchcheck

branchcheck: 
	@if git branch | grep '\* master'; then \
		echo "On master branch; aborting." ; \
		exit 2 ; fi
	@if [ -e .git/MERGE_HEAD ]; then \
		echo "Merge in progress; aborting." ; \
		exit 3; fi

push: .tag-push
.tag-push: *.org */*.org *.org_archive Makefile .git/index
	$(MAKE) branchcheck
	-git add data *.org */*.org *.org_archive Makefile .gitignore
# Do this test with wc -c to avoid calling git commit with no changes.
# This lets us honor the git commit exit code.
	[ "`git diff --cached | wc -c`" -lt 2 ] || git commit -m "Automated commit by make push on `hostname`"
	@echo "+++ Pushing..."
	@git push $(GITOUTPUT)
	touch .tag-push

fetch:
	git fetch $(GITOUTPUT) || true

# Due to set -e, this will abort the script on conflicts.  This is as
# desired.  Fix the conflict and rerun script.

sync:
	$(MAKE) branchcheck
	-git add data *.org */*.org *.org_archive
# Do this test with wc -c to avoid calling git commit with no changes.
# This lets us honor the git commit exit code.
	[ "`git diff --cached | wc -c`" -lt 2 ] || git commit -am "Commit from make sync on `hostname`"
	@echo "+++ Pulling"
	@git pull $(GITOUTPUT)
	@set -e; for BRANCH in `git branch -r`; do \
		echo "+++ Merging $$BRANCH" && \
		git merge "$$BRANCH" $(GITOUTPUT) && \
	        [ ! -f .git/MERGE_HEAD ];  done
	@echo "+++ Running git push"
	@git push $(GITOUTPUT)
	@echo "+++ Pushing to master"
	@git push origin HEAD:master $(GITOUTPUT)
	@echo "+++ Pushing tags"
	@git push --tags $(GITOUTPUT)
	touch .tag-push

