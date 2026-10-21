# Is IPV6_ENABLED documented for OpenTelemetry Demo?

Research date: 2026-10-21 (Europe/Zurich).
Local checkout: `98648c1de161d6fa13ee623a17f5871e4f1ed495`.

## Conclusion

**Only minimally: the setting is present in configuration, but I did not find
user-facing documentation explaining how to use it in the sources checked.**
The root `.env` declares `IPV6_ENABLED=false` beneath the comment
`# IPv6 Flag control`. This exposes its name and default, but does not explain
its scope, prerequisites, supported networking modes, or limitations.
[Source: .env](https://github.com/open-telemetry/opentelemetry-demo/blob/98648c1de161d6fa13ee623a17f5871e4f1ed495/.env#L26-L27)

## Evidence

| Source checked | Finding |
| --- | --- |
| [Docker deployment documentation](https://opentelemetry.io/docs/demo/docker-deployment/) | No `IPV6_ENABLED` or IPv6 mention. It explains loading `.env` and `.env.override` and documents other configuration, such as `ENVOY_PORT`. |
| [Docker deployment Markdown source](https://github.com/open-telemetry/opentelemetry.io/blob/main/content/en/docs/demo/docker-deployment.md) | Independently retrieved through the GitHub connector; no flag or IPv6 explanation. Retrieved file blob SHA: `5469758e4df09d83c1a929c729c4e7bc6f4812ef`. |
| [Kubernetes deployment documentation](https://opentelemetry.io/docs/demo/kubernetes-deployment/) | No `IPV6_ENABLED` mention. |
| [Currency](https://opentelemetry.io/docs/demo/services/currency/), [Payment](https://opentelemetry.io/docs/demo/services/payment/), [Quote](https://opentelemetry.io/docs/demo/services/quote/), and [Shipping](https://opentelemetry.io/docs/demo/services/shipping/) documentation | No `IPV6_ENABLED` mention on these four service pages. |
| [Local load-generator README](../src/load-generator/README.md) | Documents `LOAD_GENERATOR_WEB_HOST` and `LOCUST_WEB_HOST`, but not `IPV6_ENABLED`. |
| [Changelog](https://github.com/open-telemetry/opentelemetry-demo/blob/98648c1de161d6fa13ee623a17f5871e4f1ed495/CHANGELOG.md#L99) | Mentions a Currency flag-check fix at line 99 and IPv6 support at line 509; these are release-history references, not usage instructions. |

The flag has implementation references. In this checkout, the default Compose
network uses it for `enable_ipv6`, and Compose passes it to Currency, Payment,
Quote, and Shipping. The profiling configuration also sets `enable_ipv6` from
the flag. These references establish that the setting is used, rather than
providing end-user guidance.
[Sources: compose.yaml](https://github.com/open-telemetry/opentelemetry-demo/blob/98648c1de161d6fa13ee623a17f5871e4f1ed495/compose.yaml),
[compose.profiling.yaml](https://github.com/open-telemetry/opentelemetry-demo/blob/98648c1de161d6fa13ee623a17f5871e4f1ed495/compose.profiling.yaml#L4-L8)

The upstream Helm chart also lists `IPV6_ENABLED` with a value of `"false"` in
the environment configuration for those four services. This is another
configuration reference; it is not evidence of a documented end-to-end IPv6
deployment procedure.
[Source: Helm chart values](https://github.com/open-telemetry/opentelemetry-helm-charts/blob/main/charts/opentelemetry-demo/values.yaml)

## Search scope and limits

Before creating this note, searched the local repository with
`rg -n --hidden -g '!.git/**' 'IPV6_ENABLED|IPv6|ipv6' .`.
Matches were configuration, source code, a changelog fragment, and the changelog;
there was no README explanation of this flag. Read the official deployment and
service pages above. GitHub connector searches for
`IPV6_ENABLED repo:open-telemetry/opentelemetry.io` and
`ipv6 repo:open-telemetry/opentelemetry.io path:content/en/docs/demo`
returned no results. Search indexing may be incomplete; these are supporting
checks, not proof that no reference exists anywhere.

This was a documentation investigation, not a runtime test. It does not
establish whether IPv6-only operation is supported. Website and upstream `main`
links can change, and the local checkout is not assumed to equal upstream `main`.

## Suggested documentation follow-up

Recommendation: add a short IPv6 configuration section to the Docker deployment
page, with a more informative comment beside the `.env` default. Explain the
default, affected network and services, how to override the setting, and the
supported networking modes and prerequisites once verified. The deployment page
already documents environment-file loading, and `.env.override` explicitly
identifies itself as the location for local environment-variable overrides.
[Sources: Docker deployment](https://opentelemetry.io/docs/demo/docker-deployment/),
[.env.override](https://github.com/open-telemetry/opentelemetry-demo/blob/98648c1de161d6fa13ee623a17f5871e4f1ed495/.env.override#L1-L2)

This is a proposed location and scope, not an agreed maintainer direction.
