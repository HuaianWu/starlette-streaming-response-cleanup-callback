FROM public.ecr.aws/x8v8d7g8/mars-base:latest
WORKDIR /app

# Git time-travel: clone, then make the repo's default branch point AT the base
# commit with no future history — a real branch checkout (not a detached HEAD),
# future commits/tags gc'd away so the reference solution can't leak from history.
ARG BASE_SHA=39fd0ffac25593fce39466320c9a666957ce8b8c
RUN git clone https://github.com/Kludex/starlette . \
 && DEFAULT="$(git remote show origin | sed -n 's/.*HEAD branch: //p')" \
 && git checkout -B "$DEFAULT" "$BASE_SHA" \
 && git remote remove origin \
 && for b in $(git for-each-ref --format='%(refname:short)' refs/heads | grep -vx "$DEFAULT"); do git branch -D "$b" || true; done \
 && for t in $(git tag); do git merge-base --is-ancestor "$t" HEAD 2>/dev/null || git tag -d "$t"; done \
 && git reflog expire --expire=now --all \
 && git gc --prune=now \
 && { test ! -s .gitmodules || git submodule update --init --recursive; }
# Pinned from the repo's uv.lock (`uv export --frozen --no-default-groups --group <tests>`) at
# BASE_SHA, resolved for Linux / CPython 3.12 (the mars-base default interpreter), so the
# image matches upstream CI for this commit and does not drift when PyPI moves.
RUN pip install --no-cache-dir \
        "anyio==4.14.2" \
        "ast-serialize==0.6.0" \
        "attrs==25.3.0" \
        "certifi==2025.8.3" \
        "cffi==2.0.0" \
        "charset-normalizer==3.4.3" \
        "coverage==7.15.2" \
        "cryptography==48.0.1" \
        "docutils==0.22" \
        "h11==0.16.0" \
        "httpcore2==2.9.1" \
        "httpcore==1.0.9" \
        "httpx2==2.9.1" \
        "httpx==0.28.1" \
        "id==1.5.0" \
        "idna==3.18" \
        "importlib-metadata==9.0.0" \
        "iniconfig==2.1.0" \
        "itsdangerous==2.2.0" \
        "jaraco-classes==3.4.0" \
        "jaraco-context==6.0.1" \
        "jaraco-functools==4.3.0" \
        "jeepney==0.9.0" \
        "jinja2==3.1.6" \
        "keyring==25.6.0" \
        "librt==0.13.0" \
        "markdown-it-py==4.0.0" \
        "markupsafe==3.0.2" \
        "mdurl==0.1.2" \
        "more-itertools==10.8.0" \
        "mypy-extensions==1.1.0" \
        "mypy==2.1.0" \
        "nh3==0.3.0" \
        "opentelemetry-api==1.44.0" \
        "opentelemetry-sdk==1.44.0" \
        "opentelemetry-semantic-conventions==0.65b0" \
        "outcome==1.3.0.post0" \
        "packaging==26.2" \
        "pathspec==1.0.4" \
        "pluggy==1.6.0" \
        "pycparser==2.22" \
        "pygments==2.20.0" \
        "pytest-codspeed==5.0.3" \
        "pytest==9.1.1" \
        "python-multipart==0.0.32" \
        "pyyaml==6.0.3" \
        "readme-renderer==44.0" \
        "requests-toolbelt==1.0.0" \
        "requests==2.33.0" \
        "rfc3986==2.0.0" \
        "rich==15.0.0" \
        "ruff==0.16.0" \
        "secretstorage==3.3.3" \
        "sniffio==1.3.1" \
        "sortedcontainers==2.4.0" \
        "trio==0.33.0" \
        "truststore==0.10.4" \
        "twine==7.0.0" \
        "types-pyyaml==6.0.12.20250516" \
        "typing-extensions==4.15.0" \
        "urllib3==2.7.0" \
        "zipp==3.23.0" \
        "zstandard==0.25.0" \
    && pip install --no-cache-dir --no-deps -e . \
    && test -z "$(git status --porcelain)"

# v1.1 node-id scoring: pytest ships a native JUnit XML reporter (--junitxml),
# so no extra reporter dependency is required.

# Disable git commit hooks (pre-commit, husky etc.): dev-workflow tooling, not task content.
# Broken hook environments otherwise block the agent's (and oracle's) commits.
RUN cd /app && git config core.hooksPath /dev/null

CMD ["/bin/bash"]
