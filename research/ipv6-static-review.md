# IPv6 static review and proposed issue structure

Date: 2026-10-21 (Europe/Zurich).

## Agreed scope

Review every service declared in the repository's Docker Compose layers before
running resource-intensive tests. Propose working dual-stack networking when
`IPV6_ENABLED=true`, retaining the IPv4 baseline when false. IPv6-only and
Kubernetes support remain separate requirements for maintainer discussion.
Static evidence does not certify runtime connectivity. Separate confirmed
configuration facts, suspected connection failures, and unresolved image or
library defaults.

## Baseline and branch changes

Reviewed HEAD: `98648c1de161d6fa13ee623a17f5871e4f1ed495`, branch
`fix/4053-adhere-ipv6-flag`.
Pre-fix comparison: `85c2cb253ae3519ef2b3a02984c32614106de5ea`.

- `4c02ddf9` adds `enable_ipv6: ${IPV6_ENABLED}` to the base and profiling networks.
- `538c94c1` removes the forced IPv4 Locust web listener.
- `98648c1d` separates Envoy's destination variable (`LOAD_GENERATOR_WEB_HOST`)
  from Locust's listener option (`LOCUST_WEB_HOST`).

These are local branch changes, not an established upstream support contract.
The upstream [compose.yaml](https://github.com/open-telemetry/opentelemetry-demo/blob/main/compose.yaml)
retrieved during this review has blob SHA
`c82d94641b4bbcb47b582b49d59e90c3330f0f21`, matching the pre-fix file, and
does not declare `enable_ipv6`. This comparison verifies that file, not the
entire upstream branch.

Docker documents IPv6 address assignment separately from disabling IPv4.
Consequently, adding `enable_ipv6: true` without disabling IPv4 proposes a
dual-stack network; it does not automatically reconfigure service listeners
or clients. [Docker Compose network reference](https://docs.docker.com/reference/compose-file/networks/)

## Evidence inventory

- [Application services](ipv6-application-review.md).
- [Infrastructure and supporting services](ipv6-infrastructure-review.md).
- [Existing documentation investigation](ipv6-enabled-documentation.md).

The two inventory tables cover all 34 unique services in the combined Compose
configuration. Per-service conclusions are limited to the inspected source and
configuration; image defaults and actual address selection remain explicit
uncertainties where relevant.

### Priority findings

1. Chatbot's Gradio listener defaults to `0.0.0.0`, and its Compose layer does
   not forward `CHATBOT_ENDPOINT`. Envoy selects backend addresses independently.
   This is the strongest additional Locust-like investigation candidate.
2. Collector and Jaeger OTLP receivers use service hostnames as bind addresses.
   A hostname is not a wildcard listener; investigate whether the selected socket
   accepts the family selected by callers. Kafka has a similar hostname-binding
   question with different runtime semantics.
3. Payment and Shipping probe `127.0.0.1` while selecting `[::]` when enabled.
   Confirm IPv4 acceptance by their actual server implementations before calling
   this a health-check defect.
4. Envoy ingress, Agent, and MCP explicitly configure IPv4 listeners. Record the
   limitation separately from whether ordinary dual-stack flows still work.
5. Image-provider and telemetry-docs already configure both listener families.
   Commented Grafana examples and Docker's IPv4 DNS resolver address are not
   evidence of broken application listeners.

Evidence and primary-source references are in the two inventory tables. These
are static findings and investigation priorities, not newly reproduced failures.

## Lightweight validation performed

Docker Compose v5.3.1 successfully rendered all seven layers with the flag set
to each of `false` and `true`. Both renderings contained 34 services and the
expected boolean at `networks.default.enable_ipv6`. No containers were started,
images built or pulled, or networks created.

Layers: `compose.yaml`, `compose.full.yaml`, `compose.observability.yaml`,
`compose.extras.yaml`, `compose.agent.yaml`, `compose.profiling.yaml`, and
`compose.tests.yaml`. Configuration used `.env` and `.env.override` with a
temporary process environment override for the flag, restored afterwards.
Only network-flag and service-name results were printed. Docker warned that
its user configuration file was inaccessible; configuration rendering still
succeeded for both values.

## Proposed follow-up structure

Prepare a parent issue named **Clarify and validate IPv6 support in Docker
Compose**, linking:

- [Document the purpose and supported usage of IPV6_ENABLED](https://github.com/open-telemetry/opentelemetry-demo/issues/4150).
- [IPv6 brakes frontend-proxy (Envoy) to load-generator (LOCUST) communication](https://github.com/open-telemetry/opentelemetry-demo/issues/4053).
- Concrete service investigations or fixes supported by the inventory.

Keep the shared network flag wiring with the existing Locust issue initially:
it already discusses the network configuration. Create another network issue
only for an independently actionable defect or a maintainer-requested split.
The parent tracks the support expectation and progress; child issues contain
detailed evidence. Potential runtime failures must be described as investigation
candidates until demonstrated, not as reproduced outages.

## Proposed documentation-issue comment

The following is an unposted draft requiring verbatim human approval:

> The initial static review supports treating dual-stack Docker Compose operation as the proposed scope for this documentation. Docker's `enable_ipv6` setting enables IPv6 address assignment; disabling IPv4 is a separate setting. This does not automatically make every service listener dual-stack.
>
> The current fix branch adds `enable_ipv6: ${IPV6_ENABLED}` to the default network in the base and profiling Compose files. The upstream base Compose file checked during the review does not yet contain that setting, so the documentation should distinguish existing behavior from the proposed change.
>
> The flag is also passed to Currency, Payment, Quote, and Shipping, which switch their listening address when its value is the literal string `true`. Documentation should explain both effects: network address assignment and service listener configuration.
>
> I suggest covering the default, the `.env.override` workflow, affected services, and the agreed networking mode in the Docker deployment guide. IPv6-only and Kubernetes support would remain separate questions.
>
> To coordinate this with the existing Locust fix and any service-specific follow-ups, would a parent issue titled "Clarify and validate IPv6 support in Docker Compose" be useful? The shared network flag change could remain in the existing Locust issue unless maintainers prefer to split it out.
>
> Reference: https://docs.docker.com/reference/compose-file/networks/

## Proposed parent issue

Title: **Clarify and validate IPv6 support in Docker Compose**

The following body is an unposted draft requiring verbatim human approval:

> IPv6 support in the Demo spans Docker network configuration, service listeners, and client address selection. The existing documentation and Locust issues cover parts of this work; this issue would coordinate the support expectation and follow-ups.
>
> Proposed scope: Docker Compose with IPv4 as the default and working dual-stack operation when `IPV6_ENABLED=true`, subject to maintainer agreement. IPv6-only operation and Kubernetes support are separate requirements.
>
> An initial static review of all 34 services declared across the Compose layers has been completed. Configuration rendering succeeded with the flag set to `false` and `true`; this does not establish runtime connectivity. Further validation should use small, targeted reproductions before considering a full-stack run.
>
> Tracking checklist:
>
> - [ ] Agree on the supported Docker Compose networking modes.
> - [ ] Resolve the network configuration and Locust communication issue: https://github.com/open-telemetry/opentelemetry-demo/issues/4053.
> - [ ] Document the flag's default, scope, overrides, and limitations: https://github.com/open-telemetry/opentelemetry-demo/issues/4150.
> - [ ] Investigate the IPv4-only Chatbot listener behind Envoy and determine whether it reproduces the same address-family mismatch.
> - [ ] Resolve remaining listener and client address-family questions with focused evidence, creating separate service issues where warranted.
> - [ ] Record validated behavior and remaining limitations without treating static review as runtime certification.
>
> Keep the shared Compose network flag change with the existing Locust issue initially. A separate network issue is needed only if an independent problem is identified or maintainers prefer a split.

No GitHub comments or issues were posted as part of this review.
