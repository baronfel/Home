# ***Vulnerabilities in packages.config restore***

- [Nikolche Kolev](https://github.com/nkolev92)
- GitHub Issue <https://github.com/NuGet/Home/issues/12307>

## Summary

This is a continuation of the [vulnerabilities in PackageReference work](../2022/vulnerabilities-in-restore.md), extending matching functionality for packages.config restore.

## Motivation

See [vulnerabilities in PackageReference restore motivation](../2022/vulnerabilities-in-restore.md#motivation).

## Explanation

### Functional explanation

Packages.config restore combines all packages regardless of project. Instead of raising warnings and errors on a per project level, at restore time, they're raised on a per package level.

The configuration knobs for packages.config restore auditing functionality will be NuGet.config based.
In particular, there will be 2 new configuration keys:

| Key | Acceptable values | Description | Default |
|-----|-------------------|-------------|---------|
| auditForPackagesConfig | enable, disable | Enables or disables NuGet Audit for packages config projects | If not specified, the default will be `enable` |
| auditLevelForPackagesConfig | Critical, high, moderate, low | Configurations the default audit level for NuGet audit for packages config projects |  If not specified, the default will be `disable` |

The audit functionality for packages.config restore will be enabled by default.
To disable it, one can specify a property in the config section of the configuration file.

```xml
<configuration>
    <config>
        <add key="audit" value="enable" />
        <add key="auditLevel" value="low" />
    </config>
</configuration>
```

Whenever a vulnerability for a package is discovered, a warning will be raised indicating the severity and advisory url.
Note that warnings as errors and no warn are not support in packages.config projects, so these warnings will not fail the build.

> Package 'Contoso.Service.APIs' 1.0.3 has a known critical severity vulnerability, https://github.com/advisories/GHSA-1234-5678-9012.

For performance considerations, vulnerability checks are only going to be performed when the packages.config restore downloads a package.

### Technical explanation

An [AuditUtility](https://github.com/NuGet/NuGet.Client/blob/dev/src/NuGet.Core/NuGet.PackageManagement/AuditUtility.cs) already exists for packages.config projects, written along similar lines as the AuditUtility for PackageReference.
For performance considerations, vulnerability checks are only going to be performed when the packages.config restore downloads a package.

When we run restore for packages.config, the following metrics will be considered:

- Was Audit enabled
- Was Audit Run
- Reason why audit was not run. (No package downloads for example)
- Sev0Matches
- Sev1Matches
- Sev2Matches
- Sev3Matches
- InvalidSevMatches
- List of package ids with advisories
- Severity
- DownloadDurationSeconds
- CheckPackagesDurationSeconds
- SourcesWithVulnerabilityData

## Drawbacks

- Vulnerability checking comes with a performance hit.

## Rationale and alternatives

- Enable vulnerability checking on demand.
- Enable vulnerability checking at installation time only.

## Prior Art

- [Vulnerability reporting in PackageReference restore](../2022/vulnerabilities-in-restore.md)

## Unresolved Questions

- Should the NuGet.config configuration *affect* the PackageReference defaults as well?
- Should vulnerability checks be performed at every packages.config restore?
- Should we add the vulnerability metrics in the vs/nuget/restoreinformation event or a dedicated one?

<!-- What parts of the proposal do you expect to resolve before this gets accepted? -->
<!-- What parts of the proposal need to be resolved before the proposal is stabilized? -->
<!-- What related issues would you consider out of scope for this proposal but can be addressed in the future? -->

## Future Possibilities

- NuGet Audit without nuget.org as a package source - [Pull Request #12918](https://github.com/NuGet/Home/pull/12918)