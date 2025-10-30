# Galaxy Integration

## Develop the Tool Wrapper XML using Planemo

The following commands assume that Planemo is installed.
Either set up the virtual environment using [uv](https://docs.astral.sh/uv/), or install planemo in another way.
uv installation instructions:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync
source .venv/bin/activate
```

Spin up Galaxy, to run the tool [in the browser](http://127.0.0.1:9090/):

```bash
planemo serve --biocontainers
```

Run the tests (will only succeed on Linux):

```bash
planemo test --biocontainers fileidentification-galaxy.xml
```

Keep in mind:

- Before running the tests, make sure that Docker Desktop for Mac allows bind mounting of `/private/var/folders`.
- If there is more than 1 line inside the CDATA of the `<command>` block,
  the lines must end with `&&` (but NOT with `\`),
  and new lines must NOT be indented.
- Inside the Docker container, the app is installed in `/app`, which is read-only in Galaxy.
- The commands inside the `<command>` block of the XML file are executed in a dedicated user directory defined by Galaxy - 
  not in the `WORKDIR` specified in the Dockerfile.
- Only the user directory is writable, all other dirs are read-only: the input dir, `/app`, ...
- The Terminal output of the tool slightly differs across operating systems, e.g. regarding the order of files.
  The e2e test will fail on MacOS, because the expected output doesn't exactly match the actual output.
  You can run the e2e test on MacOS anyways, to get a rough idea about what's going on.


### Rebuilding the Docker Image

Every push to the main branch triggers a rebuild of the Docker image,
and uploads it to [Docker Hub](https://hub.docker.com/r/daschswiss/fileidentification-galaxy).

If you modify the Python code or the Dockerfile, and open a PR, the automated tests will rebuild the Docker image.
But if you want your local planemo to be aware of your changes, make sure to rebuild the image first:

```bash
docker build -t daschswiss/fileidentification-galaxy:latest .
```


## Tool Shed

Publishing to <https://toolshed.g2.bx.psu.edu/> is automatically done by tools-iuc,
and configured by `.shed.yml`.
Steps to test if `.shed.yml` is correct:

- Create an account on <https://testtoolshed.g2.bx.psu.edu/>
- Run `planemo config_init`
- In `~/.planemo.yml` > sheds > testtoolshed, fill in your credentials for <https://testtoolshed.g2.bx.psu.edu/>
- Check the `.shed.yml` with `planemo shed_lint --tools`.
- Create the remote repository with `planemo shed_create --shed_target testtoolshed`.
    - For this to work, you probably need to fill in your test account name in `.shed.yml` > owner.
- DaSCH has an account on the prod toolshed <https://toolshed.g2.bx.psu.edu/>,
  which is probably not used, because of `owner: iuc`.


## Synchronize this fork with the upstream

Make sure the upstream is set correctly:

```bash
git remote -v
origin	  git@github.com:dasch-swiss/fileidentification-galaxy (fetch)
origin	  git@github.com:dasch-swiss/fileidentification-galaxy (push)
upstream	git@github.com:dasch-swiss/fileidentification.git (fetch)
upstream	git@github.com:dasch-swiss/fileidentification.git (push)
```

To synchronize with upstream, create a PR with the changes from upstream:

```bash
git fetch upstream
git checkout -b sync-upstream
git merge upstream/main -m "Pull in changes from upstream"
git push -u origin sync-upstream
gh pr create --title "Sync with upstream" --body "Synchronize fork with upstream changes"
```

Alternatively, use the GitHub UI to create a PR from the upstream branch.


## Resources

- Tutorial: Tool development and integration into Galaxy:
  <https://training.galaxyproject.org/training-material/topics/dev/tutorials/tool-integration/slides.html#1>
- Documentation of the XML Schema: <https://docs.galaxyproject.org/en/latest/dev/schema.html>
- ToolShed Readiness Checklist:
  <https://galaxy-iuc-standards.readthedocs.io/en/latest/best_practices/integration_checklist.html>

Toolshed Yaml File:

- <https://planemo.readthedocs.io/en/latest/publishing.html>
- <https://galaxy-iuc-standards.readthedocs.io/en/latest/best_practices/shed_yml.html>
