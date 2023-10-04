# License Finder

We use [License Finder](https://github.com/pivotal/LicenseFinder) to check our dependencies. This allows us to bring in any dependencies that we want, ensuring that we don't put ourselves into any legal troubles by using licenses that are not allowable for what we do.

## Which configuration to use?

As licenses have different requirements, we have split the license approval files into approved licenses and internal only licenses. This is so that we can have externally facing projects (such as websites, mobile applications, etc.) to have a simple config to import and the internal only focused tools can still use dependencies that would otherwise require us to provide our source code (e.g. relying on a GPL licensed dependency would require us to provide our website source code).

- If the project is going to be distributed, such as part of a website or mobile application, or it's a library that will be part of a distributed project, then bring in the `distributed.yml`.
- If the proejct is not going be distributed and will never be distributed, then bring in the `internal_or_oss.yml`.
- If the project was distributed and no longer is, then the `internal_or_oss.yml` configuration can be brought in _after_ it has been removed from all distributed channels.
- If the project was not distrubted and now is, _before_ the project is distributed, make sure the `distributed.yml` configuration is being used, to make sure that we are not exposing us to any license issues.

There maybe some use cases where a dependency is approved under the `internal_or_oss.yml` configuration, but the project itself has a distributed aspect _not_ including this dependency. In which case, it's safe to add an approval at the _project_ level and _not_ on a common level. This allows dependencies to still be used, but requires the project to have had some forethought and control over whether the dependency will actually violate the license or not. For example, if a GPL v3 licensed dependency is _only_ used as part of the unit tests for a website, then the project is distributed and therefore GPL v3 licenses aren't permitted, but as the dependency in question is part of the unit tests and therefore _not_ distributed, it is safe to approve that dependency.

## Install License Finder locally

1. Install [Ruby](https://www.ruby-lang.org/en/) if it's not already installed
2. Install the License Finder gem
   ```bash
   gem install license_finder
   ```

## Run License Finder check

To run the report, simply run `license_finder action_items`. Unfortunately, this is output straight to the console and isn't always the most readable. Instead, use the `--format html` option to produce something that can be more readable.

```bash
license_finder action_items --format html > output.html
```

## Allow dependency/license

License Finder uses the package manager present in the repo for finding out what license is applicable to the dependency (e.g. using the license property in a `package.json` file). Unfortunately, some libraries aren't very clear in their usage (e.g. have a combination of licenses and use a custom format to denote that), don't bundle the license in with the dependency or simply don't use common conventions to signify what the license is.

Sometimes, we have dependencies that are necessary for what we do and simply aren't licensed (e.g. the Netlify CLI library brings in other Netlify libraries which are unlicensed, but needed to use the CLI).

_If the change is for a specific workspace/repo, then use `--decisions-file doc/workspace_dependencies.yml`. This should only be required in a monorepo, referencing the different projects within it, or for very specific licensing we have for that project. Otherwise, the approval/permission should be done in this repo as much as possible, so all projects don't need to have the same rules specified separately._

### Add license

Assuming that the license that is missing is one that should be compatible for all dependencies that are attributed that license, we can add said license.

_N.B. If the license has a `*` after it, then it means `yarn` has tried to figure it out through other files, so best to explicitely add the license to the dependency instead of adding the license, in case it got the license wrong_

```bash
license_finder permitted_licenses add "MIT" --decisions-file doc/approved_licenses.yml --who "Benjamin Sproule" --why "Compatible license"
```

### Add license to a dependency with a license that can't be found

Assuming the license that is attributed to the dependency is compatible for us to use, the dependency can be assigned a license for license_finder to use.

_N.B. Make sure to specify who added it, the reason for adding it and the version, in case it changes in the future_

```bash
license_finder licenses add fsevents "MIT" --version 1.0.0  --decisions-file doc/fixed_dependency_licenses.yml --who "Benjamin Sproule" --why "Later versions added license"
```

### Add dependency without a license

This should only be used for extremley specific use cases, for example a deployment tool like Netlify CLI. For anything else, it should be specified in the project explicitely.

Assuming that the dependency is required for all projects to use, the dependency can be explicitly approved.

_N.B. Make sure to specify who added it, the reason for adding it and the version, in case it changes in the future_

```bash
license_finder approvals add traffic-mesh-agent-linux-x64 0.1.2 --decisions-file doc/approved_dependencies.yml --who "Benjamin Sproule" --why "license_finder can't access the repo (required by Netlify CLI)"
```

### Unapprove dependency with incompatible license

Assuming that a dependency was once required and didn't have a license and now has an incompatible license, it is important that it is removed from the approved depdencies list. The cleanest way to deal with this is to simply delete the relevant entry from the `approved_dependencies.yml`. Using the `license_finder` CLI just adds another entry into the file.
