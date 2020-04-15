[//]: # (title: Security)
[//]: # (auxiliary-id: Security)

We do our best to make sure our software is free of security vulnerabilities. This page describes how to report any security issues you find in Kotlin Multiplatform Mobile and what best practices to follow to reduce the risk of introducing a vulnerability.

## Reporting
We are very eager and grateful to hear about any security issues you find. To report vulnerabilities that you discover in any part of KMM, please post a message directly to our [issue tracker](https://youtrack.jetbrains.com/issues/KT) (setting the issue tag to ['kotlin-security'](https://youtrack.jetbrains.com/issues/KT?q=%23kotlin-security%20)) or send us an [email](mailto:security@jetbrains.org). For further information on how our responsible disclosure process works, please check the JetBrains [Coordinated Disclosure Policy](https://www.jetbrains.com/legal/terms/coordinated-disclosure.html).

## Best practices

* Always use the latest KMM components releases.
* Use the latest versions of your application’s dependencies. If you still need to use a specific dependency version, periodically check if there are any new security vulnerabilities. You can follow [the guidelines from GitHub](https://help.github.com/en/github/managing-security-vulnerabilities/managing-vulnerabilities-in-your-projects-dependencies) or browse known vulnerabilities in the [CVE base](https://cve.mitre.org/cve/search_cve_list.html).
