---
reviewers:
- smarterclayton
- lavalamp
- caesarxuchao
- deads2k
- liggitt
- jpbetz
title: Dynamic Admission Control
content_type: concept
weight: 45
---

<!-- overview -->
In addition to [compiled-in admission plugins](/docs/reference/access-authn-authz/admission-controllers/),
admission plugins can be developed as extensions and run as webhooks configured at runtime.
This page describes how to build, configure, use, and monitor admission webhooks.

<!-- body -->

## What are admission webhooks?

Admission webhooks are HTTP callbacks that receive admission requests and do
something with them. You can define two types of admission webhooks,
[validating admission webhook](/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook)
and
[mutating admission webhook](/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook).
Mutating admission webhooks are invoked first, and can modify objects sent to the API server to enforce custom defaults.
After all object modifications are complete, and after the incoming object is validated by the API server,
validating admission webhooks are invoked and can reject requests to enforce custom policies.

{{< note >}}
Admission webhooks that need to guarantee they see the final state of the object in order to enforce policy
should use a validating admission webhook, since objects can be modified after being seen by mutating webhooks.
{{< /note >}}

## Experimenting with admission webhooks

Admission webhooks are essentially part of the cluster control-plane. You should
write and deploy them with great caution. Please read the
[user guides](/docs/reference/access-authn-authz/extensible-admission-controllers/#write-an-admission-webhook-server)
for instructions if you intend to write/deploy production-grade admission webhooks.
In the following, we describe how to quickly experiment with admission webhooks.

### Prerequisites

* Ensure that MutatingAdmissionWebhook and ValidatingAdmissionWebhook
  admission controllers are enabled.
  [Here](/docs/reference/access-authn-authz/admission-controllers/#is-there-a-recommended-set-of-admission-controllers-to-use)
  is a recommended set of admission controllers to enable in general.

* Ensure that the `admissionregistration.k8s.io/v1` API is enabled.

### Write an admission webhook server

Please refer to the implementation of the [admission webhook server](https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go)
that is validated in a Kubernetes e2e test. The webhook handles the
`AdmissionReview` request sent by the API servers, and sends back its decision
as an `AdmissionReview` object in the same version it received.

See the [webhook request](#request) section for details on the data sent to webhooks.

See the [webhook response](#response) section for the data expected from webhooks.

The example admission webhook server leaves the `ClientAuth` field
[empty](https://github.com/kubernetes/kubernetes/blob/v1.22.0/test/images/agnhost/webhook/config.go#L38-L39),
which defaults to `NoClientCert`. This means that the webhook server does not
authenticate the identity of the clients, supposedly API servers. If you need
mutual TLS or other ways to authenticate the clients, see
how to [authenticate API servers](#authenticate-apiservers).

### Deploy the admission webhook service

The webhook server in the e2e test is deployed in the Kubernetes cluster, via
the [deployment API](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#deployment-v1-apps).
The test also creates a [service](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core)
as the front-end of the webhook server. See
[code](https://github.com/kubernetes/kubernetes/blob/v1.22.0/test/e2e/apimachinery/webhook.go#L748).

You may also deploy your webhooks outside of the cluster. You will need to update
your webhook configurations accordingly.

### Configure admission webhooks on the fly

You can dynamically configure what resources are subject to what admission
webhooks via
[ValidatingWebhookConfiguration](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#validatingwebhookconfiguration-v1-admissionregistration-k8s-io)
or
[MutatingWebhookConfiguration](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#mutatingwebhookconfiguration-v1-admissionregistration-k8s-io).

The following is an example `ValidatingWebhookConfiguration`, a mutating webhook configuration is similar.
See the [webhook configuration](#webhook-configuration) section for details about each config field.

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  rules:
  - apiGroups:   [""]
    apiVersions: ["v1"]
    operations:  ["CREATE"]
    resources:   ["pods"]
    scope:       "Namespaced"
  clientConfig:
    service:
      namespace: "example-namespace"
      name: "example-service"
    caBundle: <CA_BUNDLE>
  admissionReviewVersions: ["v1"]
  sideEffects: None
  timeoutSeconds: 5
```

{{< note >}}
You must replace the `<CA_BUNDLE>` in the above example by a valid CA bundle
which is a PEM-encoded (field value is Base64 encoded) CA bundle for validating the webhook's server certificate.
{{< /note >}}

The `scope` field specifies if only cluster-scoped resources ("Cluster") or namespace-scoped
resources ("Namespaced") will match this rule. "&lowast;" means that there are no scope restrictions.

{{< note >}}
When using `clientConfig.service`, the server cert must be valid for
`<svc_name>.<svc_namespace>.svc`.
{{< /note >}}

{{< note >}}
Default timeout for a webhook call is 10 seconds,
You can set the `timeout` and it is encouraged to use a short timeout for webhooks.
If the webhook call times out, the request is handled according to the webhook's
failure policy.
{{< /note >}}

When an API server receives a request that matches one of the `rules`, the
API server sends an `admissionReview` request to webhook as specified in the
`clientConfig`.

After you create the webhook configuration, the system will take a few seconds
to honor the new configuration.

### Authenticate API servers   {#authenticate-apiservers}

If your admission webhooks require authentication, you can configure the
API servers to use basic auth, bearer token, or a cert to authenticate itself to
the webhooks. There are three steps to complete the configuration.

* When starting the API server, specify the location of the admission control
  configuration file via the `--admission-control-config-file` flag.

* In the admission control configuration file, specify where the
  MutatingAdmissionWebhook controller and ValidatingAdmissionWebhook controller
  should read the credentials. The credentials are stored in kubeConfig files
  (yes, the same schema that's used by kubectl), so the field name is
  `kubeConfigFile`. Here is an example admission control configuration file:

{{< tabs name="admissionconfiguration_example1" >}}
{{% tab name="apiserver.config.k8s.io/v1" %}}
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ValidatingAdmissionWebhook
  configuration:
    apiVersion: apiserver.config.k8s.io/v1
    kind: WebhookAdmissionConfiguration
    kubeConfigFile: "<path-to-kubeconfig-file>"
- name: MutatingAdmissionWebhook
  configuration:
    apiVersion: apiserver.config.k8s.io/v1
    kind: WebhookAdmissionConfiguration
    kubeConfigFile: "<path-to-kubeconfig-file>"
```
{{% /tab %}}
{{% tab name="apiserver.k8s.io/v1alpha1" %}}
```yaml
# Deprecated in v1.17 in favor of apiserver.config.k8s.io/v1
apiVersion: apiserver.k8s.io/v1alpha1
kind: AdmissionConfiguration
plugins:
- name: ValidatingAdmissionWebhook
  configuration:
    # Deprecated in v1.17 in favor of apiserver.config.k8s.io/v1, kind=WebhookAdmissionConfiguration
    apiVersion: apiserver.config.k8s.io/v1alpha1
    kind: WebhookAdmission
    kubeConfigFile: "<path-to-kubeconfig-file>"
- name: MutatingAdmissionWebhook
  configuration:
    # Deprecated in v1.17 in favor of apiserver.config.k8s.io/v1, kind=WebhookAdmissionConfiguration
    apiVersion: apiserver.config.k8s.io/v1alpha1
    kind: WebhookAdmission
    kubeConfigFile: "<path-to-kubeconfig-file>"
```
{{% /tab %}}
{{< /tabs >}}

For more information about `AdmissionConfiguration`, see the
[AdmissionConfiguration (v1) reference](/docs/reference/config-api/apiserver-webhookadmission.v1/).
See the [webhook configuration](#webhook-configuration) section for details about each config field.

In the kubeConfig file, provide the credentials:

```yaml
apiVersion: v1
kind: Config
users:
# name should be set to the DNS name of the service or the host (including port) of the URL the webhook is configured to speak to.
# If a non-443 port is used for services, it must be included in the name when configuring 1.16+ API servers.
#
# For a webhook configured to speak to a service on the default port (443), specify the DNS name of the service:
# - name: webhook1.ns1.svc
#   user: ...
#
# For a webhook configured to speak to a service on non-default port (e.g. 8443), specify the DNS name and port of the service in 1.16+:
# - name: webhook1.ns1.svc:8443
#   user: ...
# and optionally create a second stanza using only the DNS name of the service for compatibility with 1.15 API servers:
# - name: webhook1.ns1.svc
#   user: ...
#
# For webhooks configured to speak to a URL, match the host (and port) specified in the webhook's URL. Examples:
# A webhook with `url: https://www.example.com`:
# - name: www.example.com
#   user: ...
#
# A webhook with `url: https://www.example.com:443`:
# - name: www.example.com:443
#   user: ...
#
# A webhook with `url: https://www.example.com:8443`:
# - name: www.example.com:8443
#   user: ...
#
- name: 'webhook1.ns1.svc'
  user:
    client-certificate-data: "<pem encoded certificate>"
    client-key-data: "<pem encoded key>"
# The `name` supports using * to wildcard-match prefixing segments.
- name: '*.webhook-company.org'
  user:
    password: "<password>"
    username: "<name>"
# '*' is the default match.
- name: '*'
  user:
    token: "<token>"
```

Of course you need to set up the webhook server to handle these authentication requests.

## Webhook request and response

### Request

Webhooks are sent as POST requests, with `Content-Type: application/json`,
with an `AdmissionReview` API object in the `admission.k8s.io` API group
serialized to JSON as the body.

Webhooks can specify what versions of `AdmissionReview` objects they accept
with the `admissionReviewVersions` field in their configuration:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  admissionReviewVersions: ["v1", "v1beta1"]
```

`admissionReviewVersions` is a required field when creating webhook configurations.
Webhooks are required to support at least one `AdmissionReview`
version understood by the current and previous API server.

API servers send the first `AdmissionReview` version in the `admissionReviewVersions` list they support.
If none of the versions in the list are supported by the API server, the configuration will not be allowed to be created.
If an API server encounters a webhook configuration that was previously created and does not support any of the `AdmissionReview`
versions the API server knows how to send, attempts to call to the webhook will fail and be subject to the [failure policy](#failure-policy).

This example shows the data contained in an `AdmissionReview` object
for a request to update the `scale` subresource of an `apps/v1` `Deployment`:

```yaml
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "request": {
    # Random uid uniquely identifying this admission call
    "uid": "705ab4f5-6393-11e8-b7cc-42010a800002",

    # Fully-qualified group/version/kind of the incoming object
    "kind": {
      "group": "autoscaling",
      "version": "v1",
      "kind": "Scale"
    },

    # Fully-qualified group/version/kind of the resource being modified
    "resource": {
      "group": "apps",
      "version": "v1",
      "resource": "deployments"
    },

    # Subresource, if the request is to a subresource
    "subResource": "scale",

    # Fully-qualified group/version/kind of the incoming object in the original request to the API server
    # This only differs from `kind` if the webhook specified `matchPolicy: Equivalent` and the original
    # request to the API server was converted to a version the webhook registered for
    "requestKind": {
      "group": "autoscaling",
      "version": "v1",
      "kind": "Scale"
    },

    # Fully-qualified group/version/kind of the resource being modified in the original request to the API server
    # This only differs from `resource` if the webhook specified `matchPolicy: Equivalent` and the original
    # request to the API server was converted to a version the webhook registered for
    "requestResource": {
      "group": "apps",
      "version": "v1",
      "resource": "deployments"
    },

    # Subresource, if the request is to a subresource
    # This only differs from `subResource` if the webhook specified `matchPolicy: Equivalent` and the original
    # request to the API server was converted to a version the webhook registered for
    "requestSubResource": "scale",

    # Name of the resource being modified
    "name": "my-deployment",

    # Namespace of the resource being modified, if the resource is namespaced (or is a Namespace object)
    "namespace": "my-namespace",

    # operation can be CREATE, UPDATE, DELETE, or CONNECT
    "operation": "UPDATE",

    "userInfo": {
      # Username of the authenticated user making the request to the API server
      "username": "admin",

      # UID of the authenticated user making the request to the API server
      "uid": "014fbff9a07c",

      # Group memberships of the authenticated user making the request to the API server
      "groups": [
        "system:authenticated",
        "my-admin-group"
      ],

      # Arbitrary extra info associated with the user making the request to the API server
      # This is populated by the API server authentication layer
      "extra": {
        "some-key": [
          "some-value1",
          "some-value2"
        ]
      }
    },

    # object is the new object being admitted. It is null for DELETE operations
    "object": {
      "apiVersion": "autoscaling/v1",
      "kind": "Scale"
    },

    # oldObject is the existing object. It is null for CREATE and CONNECT operations
    "oldObject": {
      "apiVersion": "autoscaling/v1",
      "kind": "Scale"
    },

    # options contain the options for the operation being admitted, like meta.k8s.io/v1 CreateOptions,
    # UpdateOptions, or DeleteOptions. It is null for CONNECT operations
    "options": {
      "apiVersion": "meta.k8s.io/v1",
      "kind": "UpdateOptions"
    },

    # dryRun indicates the API request is running in dry run mode and will not be persisted
    # Webhooks with side effects should avoid actuating those side effects when dryRun is true
    "dryRun": false
  }
}
```

### Response

Webhooks respond with a 200 HTTP status code, `Content-Type: application/json`,
and a body containing an `AdmissionReview` object (in the same version they were sent),
with the `response` stanza populated, serialized to JSON.

At a minimum, the `response` stanza must contain the following fields:

* `uid`, copied from the `request.uid` sent to the webhook
* `allowed`, either set to `true` or `false`

Example of a minimal response from a webhook to allow a request:

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true
  }
}
```

Example of a minimal response from a webhook to forbid a request:

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": false
  }
}
```

When rejecting a request, the webhook can customize the http code and message returned to the user
using the `status` field. The specified status object is returned to the user.
See the [API documentation](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#status-v1-meta)
for details about the `status` type.
Example of a response to forbid a request, customizing the HTTP status code and message presented to the user:

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": false,
    "status": {
      "code": 403,
      "message": "You cannot do this because it is Tuesday and your name starts with A"
    }
  }
}
```

When allowing a request, a mutating admission webhook may optionally modify the incoming object as well.
This is done using the `patch` and `patchType` fields in the response.
The only currently supported `patchType` is `JSONPatch`.
See [JSON patch](https://jsonpatch.com/) documentation for more details.
For `patchType: JSONPatch`, the `patch` field contains a base64-encoded array of JSON patch operations.

As an example, a single patch operation that would set `spec.replicas` would be
`[{"op": "add", "path": "/spec/replicas", "value": 3}]`

Base64-encoded, this would be `W3sib3AiOiAiYWRkIiwgInBhdGgiOiAiL3NwZWMvcmVwbGljYXMiLCAidmFsdWUiOiAzfV0=`

So a webhook response to add that label would be:

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true,
    "patchType": "JSONPatch",
    "patch": "W3sib3AiOiAiYWRkIiwgInBhdGgiOiAiL3NwZWMvcmVwbGljYXMiLCAidmFsdWUiOiAzfV0="
  }
}
```

Admission webhooks can optionally return warning messages that are returned to the requesting client
in HTTP `Warning` headers with a warning code of 299. Warnings can be sent with allowed or rejected admission responses.

If you're implementing a webhook that returns a warning:

* Don't include a "Warning:" prefix in the message
* Use warning messages to describe problems the client making the API request should correct or be aware of
* Limit warnings to 120 characters if possible

{{< caution >}}
Individual warning messages over 256 characters may be truncated by the API server before being returned to clients.
If more than 4096 characters of warning messages are added (from all sources), additional warning messages are ignored.
{{< /caution >}}

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true,
    "warnings": [
      "duplicate envvar entries specified with name MY_ENV",
      "memory request less than 4MB specified for container mycontainer, which will not start successfully"
    ]
  }
}
```

## Webhook configuration

To register admission webhooks, create `MutatingWebhookConfiguration` or `ValidatingWebhookConfiguration` API objects.
The name of a `MutatingWebhookConfiguration` or a `ValidatingWebhookConfiguration` object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

Each configuration can contain one or more webhooks.
If multiple webhooks are specified in a single configuration, each must be given a unique name.
This is required in order to make resulting audit logs and metrics easier to match up to active
configurations.

Each webhook defines the following things.

### Matching requests: rules

Each webhook must specify a list of rules used to determine if a request to the API server should be sent to the webhook.
Each rule specifies one or more operations, apiGroups, apiVersions, and resources, and a resource scope:

* `operations` lists one or more operations to match. Can be `"CREATE"`, `"UPDATE"`, `"DELETE"`, `"CONNECT"`,
  or `"*"` to match all.
* `apiGroups` lists one or more API groups to match. `""` is the core API group. `"*"` matches all API groups.
* `apiVersions` lists one or more API versions to match. `"*"` matches all API versions.
* `resources` lists one or more resources to match.

  * `"*"` matches all resources, but not subresources.
  * `"*/*"` matches all resources and subresources.
  * `"pods/*"` matches all subresources of pods.
  * `"*/status"` matches all status subresources.

* `scope` specifies a scope to match. Valid values are `"Cluster"`, `"Namespaced"`, and `"*"`.
  Subresources match the scope of their parent resource. Default is `"*"`.

  * `"Cluster"` means that only cluster-scoped resources will match this rule (Namespace API objects are cluster-scoped).
  * `"Namespaced"` means that only namespaced resources will match this rule.
  * `"*"` means that there are no scope restrictions.

If an incoming request matches one of the specified `operations`, `groups`, `versions`,
`resources`, and `scope` for any of a webhook's `rules`, the request is sent to the webhook.

Here are other examples of rules that could be used to specify which resources should be intercepted.

Match `CREATE` or `UPDATE` requests to `apps/v1` and `apps/v1beta1` `deployments` and `replicasets`:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
...
webhooks:
- name: my-webhook.example.com
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: ["apps"]
    apiVersions: ["v1", "v1beta1"]
    resources: ["deployments", "replicasets"]
    scope: "Namespaced"
  ...
```

Match create requests for all resources (but not subresources) in all API groups and versions:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
webhooks:
  - name: my-webhook.example.com
    rules:
      - operations: ["CREATE"]
        apiGroups: ["*"]
        apiVersions: ["*"]
        resources: ["*"]
        scope: "*"
```

Match update requests for all `status` subresources in all API groups and versions:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
webhooks:
  - name: my-webhook.example.com
    rules:
      - operations: ["UPDATE"]
        apiGroups: ["*"]
        apiVersions: ["*"]
        resources: ["*/status"]
        scope: "*"
```

### Matching requests: objectSelector

Webhooks may optionally limit which requests are intercepted based on the labels of the
objects they would be sent, by specifying an `objectSelector`. If specified, the objectSelector
is evaluated against both the object and oldObject that would be sent to the webhook,
and is considered to match if either object matches the selector.

A null object (`oldObject` in the case of create, or `newObject` in the case of delete),
or an object that cannot have labels (like a `DeploymentRollback` or a `PodProxyOptions` object)
is not considered to match.

Use the object selector only if the webhook is opt-in, because end users may skip
the admission webhook by setting the labels.

This example shows a mutating webhook that would match a `CREATE` of any resource (but not subresources) with the label `foo: bar`:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  objectSelector:
    matchLabels:
      foo: bar
  rules:
  - operations: ["CREATE"]
    apiGroups: ["*"]
    apiVersions: ["*"]
    resources: ["*"]
    scope: "*"
```

See [labels concept](/docs/concepts/overview/working-with-objects/labels)
for more examples of label selectors.

### Matching requests: namespaceSelector

Webhooks may optionally limit which requests for namespaced resources are intercepted,
based on the labels of the containing namespace, by specifying a `namespaceSelector`.

The `namespaceSelector` decides whether to run the webhook on a request for a namespaced resource
(or a Namespace object), based on whether the namespace's labels match the selector.
If the object itself is a namespace, the matching is performed on object.metadata.labels.
If the object is a cluster scoped resource other than a Namespace, `namespaceSelector` has no effect.

This example shows a mutating webhook that matches a `CREATE` of any namespaced resource inside a namespace
that does not have a "runlevel" label of "0" or "1":

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
webhooks:
  - name: my-webhook.example.com
    namespaceSelector:
      matchExpressions:
        - key: runlevel
          operator: NotIn
          values: ["0","1"]
    rules:
      - operations: ["CREATE"]
        apiGroups: ["*"]
        apiVersions: ["*"]
        resources: ["*"]
        scope: "Namespaced"
```

This example shows a validating webhook that matches a `CREATE` of any namespaced resource inside
a namespace that is associated with the "environment" of "prod" or "staging":

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
webhooks:
  - name: my-webhook.example.com
    namespaceSelector:
      matchExpressions:
        - key: environment
          operator: In
          values: ["prod","staging"]
    rules:
      - operations: ["CREATE"]
        apiGroups: ["*"]
        apiVersions: ["*"]
        resources: ["*"]
        scope: "Namespaced"
```

See [labels concept](/docs/concepts/overview/working-with-objects/labels)
for more examples of label selectors.

### Matching requests: matchPolicy

API servers can make objects available via multiple API groups or versions.

For example, if a webhook only specified a rule for some API groups/versions
(like `apiGroups:["apps"], apiVersions:["v1","v1beta1"]`),
and a request was made to modify the resource via another API group/version (like `extensions/v1beta1`),
the request would not be sent to the webhook.

The `matchPolicy` lets a webhook define how its `rules` are used to match incoming requests.
Allowed values are `Exact` or `Equivalent`.

* `Exact` means a request should be intercepted only if it exactly matches a specified rule.
* `Equivalent` means a request should be intercepted if it modifies a resource listed in `rules`,
  even via another API group or version.

In the example given above, the webhook that only registered for `apps/v1` could use `matchPolicy`:

* `matchPolicy: Exact` would mean the `extensions/v1beta1` request would not be sent to the webhook
* `matchPolicy: Equivalent` means the `extensions/v1beta1` request would be sent to the webhook
  (with the objects converted to a version the webhook had specified: `apps/v1`)

Specifying `Equivalent` is recommended, and ensures that webhooks continue to intercept the
resources they expect when upgrades enable new versions of the resource in the API server.

When a resource stops being served by the API server, it is no longer considered equivalent to
other versions of that resource that are still served.
For example, `extensions/v1beta1` deployments were first deprecated and then removed (in Kubernetes v1.16).

Since that removal, a webhook with a `apiGroups:["extensions"], apiVersions:["v1beta1"], resources:["deployments"]` rule
does not intercept deployments created via `apps/v1` APIs. For that reason, webhooks should prefer registering
for stable versions of resources.

This example shows a validating webhook that intercepts modifications to deployments (no matter the API group or version),
and is always sent an `apps/v1` `Deployment` object:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  matchPolicy: Equivalent
  rules:
  - operations: ["CREATE","UPDATE","DELETE"]
    apiGroups: ["apps"]
    apiVersions: ["v1"]
    resources: ["deployments"]
    scope: "Namespaced"
```

The `matchPolicy` for an admission webhooks defaults to `Equivalent`.

### Matching requests: `matchConditions`

{{< feature-state feature_gate_name="AdmissionWebhookMatchConditions" >}}

You can define _match conditions_ for webhooks if you need fine-grained request filtering. These
conditions are useful if you find that match rules, `objectSelectors` and `namespaceSelectors` still
doesn't provide the filtering you want over when to call out over HTTP. Match conditions are
[CEL expressions](/docs/reference/using-api/cel/). All match conditions must evaluate to true for the
webhook to be called.

Here is an example illustrating a few different uses for match conditions:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
webhooks:
  - name: my-webhook.example.com
    matchPolicy: Equivalent
    rules:
      - operations: ['CREATE','UPDATE']
        apiGroups: ['*']
        apiVersions: ['*']
        resources: ['*']
    failurePolicy: 'Ignore' # Fail-open (optional)
    sideEffects: None
    clientConfig:
      service:
        namespace: my-namespace
        name: my-webhook
      caBundle: '<omitted>'
    # You can have up to 64 matchConditions per webhook
    matchConditions:
      - name: 'exclude-leases' # Each match condition must have a unique name
        expression: '!(request.resource.group == "coordination.k8s.io" && request.resource.resource == "leases")' # Match non-lease resources.
      - name: 'exclude-kubelet-requests'
        expression: '!("system:nodes" in request.userInfo.groups)' # Match requests made by non-node users.
      - name: 'rbac' # Skip RBAC requests, which are handled by the second webhook.
        expression: 'request.resource.group != "rbac.authorization.k8s.io"'
  
  # This example illustrates the use of the 'authorizer'. The authorization check is more expensive
  # than a simple expression, so in this example it is scoped to only RBAC requests by using a second
  # webhook. Both webhooks can be served by the same endpoint.
  - name: rbac.my-webhook.example.com
    matchPolicy: Equivalent
    rules:
      - operations: ['CREATE','UPDATE']
        apiGroups: ['rbac.authorization.k8s.io']
        apiVersions: ['*']
        resources: ['*']
    failurePolicy: 'Fail' # Fail-closed (the default)
    sideEffects: None
    clientConfig:
      service:
        namespace: my-namespace
        name: my-webhook
      caBundle: '<omitted>'
    # You can have up to 64 matchConditions per webhook
    matchConditions:
      - name: 'breakglass'
        # Skip requests made by users authorized to 'breakglass' on this webhook.
        # The 'breakglass' API verb does not need to exist outside this check.
        expression: '!authorizer.group("admissionregistration.k8s.io").resource("validatingwebhookconfigurations").name("my-webhook.example.com").check("breakglass").allowed()'
```

{{< note >}}
You can define up to 64 elements in the `matchConditions` field per webhook.
{{< /note >}}

Match conditions have access to the following CEL variables:

- `object` - The object from the incoming request. The value is null for DELETE requests. The object
  version may be converted based on the [matchPolicy](#matching-requests-matchpolicy).
- `oldObject` - The existing object. The value is null for CREATE requests.
- `request` - The request portion of the [AdmissionReview](#request), excluding `object` and `oldObject`.
- `authorizer` - A CEL Authorizer. May be used to perform authorization checks for the principal
  (authenticated user) of the request. See
  [Authz](https://pkg.go.dev/k8s.io/apiserver/pkg/cel/library#Authz) in the Kubernetes CEL library
  documentation for more details.
- `authorizer.requestResource` - A shortcut for an authorization check configured with the request
  resource (group, resource, (subresource), namespace, name).

For more information on CEL expressions, refer to the
[Common Expression Language in Kubernetes reference](/docs/reference/using-api/cel/).

In the event of an error evaluating a match condition the webhook is never called. Whether to reject
the request is determined as follows:

1. If **any** match condition evaluated to `false` (regardless of other errors), the API server skips the webhook.
2. Otherwise:
    - for [`failurePolicy: Fail`](#failure-policy), reject the request (without calling the webhook).
    - for [`failurePolicy: Ignore`](#failure-policy), proceed with the request but skip the webhook.

### Contacting the webhook

Once the API server has determined a request should be sent to a webhook,
it needs to know how to contact the webhook. This is specified in the `clientConfig`
stanza of the webhook configuration.

Webhooks can either be called via a URL or a service reference,
and can optionally include a custom CA bundle to use to verify the TLS connection.

#### URL

`url` gives the location of the webhook, in standard URL form
(`scheme://host:port/path`).

The `host` should not refer to a service running in the cluster; use
a service reference by specifying the `service` field instead.
The host might be resolved via external DNS in some API servers
(e.g., `kube-apiserver` cannot resolve in-cluster DNS as that would
be a layering violation). `host` may also be an IP address.

Please note that using `localhost` or `127.0.0.1` as a `host` is
risky unless you take great care to run this webhook on all hosts
which run an API server which might need to make calls to this
webhook. Such installations are likely to be non-portable or not readily
run in a new cluster.

The scheme must be "https"; the URL must begin with "https://".

Attempting to use a user or basic auth (for example `user:password@`) is not allowed.
Fragments (`#...`) and query parameters (`?...`) are also not allowed.

Here is an example of a mutating webhook configured to call a URL
(and expects the TLS certificate to be verified using system trust roots, so does not specify a caBundle):

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  clientConfig:
    url: "https://my-webhook.example.com:9443/my-webhook-path"
```

#### Service reference

The `service` stanza inside `clientConfig` is a reference to the service for this webhook.
If the webhook is running within the cluster, then you should use `service` instead of `url`.
The service namespace and name are required. The port is optional and defaults to 443.
The path is optional and defaults to "/".

Here is an example of a mutating webhook configured to call a service on port "1234"
at the subpath "/my-path", and to verify the TLS connection against the ServerName
`my-service-name.my-service-namespace.svc` using a custom CA bundle:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  clientConfig:
    caBundle: <CA_BUNDLE>
    service:
      namespace: my-service-namespace
      name: my-service-name
      path: /my-path
      port: 1234
```

{{< note >}}
You must replace the `<CA_BUNDLE>` in the above example by a valid CA bundle
which is a PEM-encoded CA bundle for validating the webhook's server certificate.
{{< /note >}}

### Side effects

Webhooks typically operate only on the content of the `AdmissionReview` sent to them.
Some webhooks, however, make out-of-band changes as part of processing admission requests.

Webhooks that make out-of-band changes ("side effects") must also have a reconciliation mechanism
(like a controller) that periodically determines the actual state of the world, and adjusts
the out-of-band data modified by the admission webhook to reflect reality.
This is because a call to an admission webhook does not guarantee the admitted object will be persisted as is, or at all.
Later webhooks can modify the content of the object, a conflict could be encountered while writing to storage,
or the server could power off before persisting the object.

Additionally, webhooks with side effects must skip those side-effects when `dryRun: true` admission requests are handled.
A webhook must explicitly indicate that it will not have side-effects when run with `dryRun`,
or the dry-run request will not be sent to the webhook and the API request will fail instead.

Webhooks indicate whether they have side effects using the `sideEffects` field in the webhook configuration:

* `None`: calling the webhook will have no side effects.
* `NoneOnDryRun`: calling the webhook will possibly have side effects, but if a request with
  `dryRun: true` is sent to the webhook, the webhook will suppress the side effects (the webhook
  is `dryRun`-aware).

Here is an example of a validating webhook indicating it has no side effects on `dryRun: true` requests:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
webhooks:
  - name: my-webhook.example.com
    sideEffects: NoneOnDryRun
```

### Timeouts

Because webhooks add to API request latency, they should evaluate as quickly as possible.
`timeoutSeconds` allows configuring how long the API server should wait for a webhook to respond
before treating the call as a failure.

If the timeout expires before the webhook responds, the webhook call will be ignored or
the API call will be rejected based on the [failure policy](#failure-policy).

The timeout value must be between 1 and 30 seconds.

Here is an example of a validating webhook with a custom timeout of 2 seconds:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
webhooks:
  - name: my-webhook.example.com
    timeoutSeconds: 2
```

The timeout for an admission webhook defaults to 10 seconds.

### Reinvocation policy

A single ordering of mutating admissions plugins (including webhooks) does not work for all cases
(see https://issue.k8s.io/64333 as an example). A mutating webhook can add a new sub-structure
to the object (like adding a `container` to a `pod`), and other mutating plugins which have already
run may have opinions on those new structures (like setting an `imagePullPolicy` on all containers).

To allow mutating admission plugins to observe changes made by other plugins,
built-in mutating admission plugins are re-run if a mutating webhook modifies an object,
and mutating webhooks can specify a `reinvocationPolicy` to control whether they are reinvoked as well.

`reinvocationPolicy` may be set to `Never` or `IfNeeded`. It defaults to `Never`.

* `Never`: the webhook must not be called more than once in a single admission evaluation.
* `IfNeeded`: the webhook may be called again as part of the admission evaluation if the object
  being admitted is modified by other admission plugins after the initial webhook call.

The important elements to note are:

* The number of additional invocations is not guaranteed to be exactly one.
* If additional invocations result in further modifications to the object, webhooks are not
  guaranteed to be invoked again.
* Webhooks that use this option may be reordered to minimize the number of additional invocations.
* To validate an object after all mutations are guaranteed complete, use a validating admission
  webhook instead (recommended for webhooks with side-effects).

Here is an example of a mutating webhook opting into being re-invoked if later admission plugins
modify the object:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  reinvocationPolicy: IfNeeded
```

Mutating webhooks must be [idempotent](#idempotence), able to successfully process an object they have already admitted
and potentially modified. This is true for all mutating admission webhooks, since any change they can make
in an object could already exist in the user-provided object, but it is essential for webhooks that opt into reinvocation.

### Failure policy

`failurePolicy` defines how unrecognized errors and timeout errors from the admission webhook
are handled. Allowed values are `Ignore` or `Fail`.

* `Ignore` means that an error calling the webhook is ignored and the API request is allowed to continue.
* `Fail` means that an error calling the webhook causes the admission to fail and the API request to be rejected.

Here is a mutating webhook configured to reject an API request if errors are encountered calling the admission webhook:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
webhooks:
- name: my-webhook.example.com
  failurePolicy: Fail
```

The default `failurePolicy` for an admission webhooks is `Fail`.

## Monitoring admission webhooks

The API server provides ways to monitor admission webhook behaviors. These
monitoring mechanisms help cluster admins to answer questions like:

1. Which mutating webhook mutated the object in a API request?

2. What change did the mutating webhook applied to the object?

3. Which webhooks are frequently rejecting API requests? What's the reason for a rejection?

### Mutating webhook auditing annotations

Sometimes it's useful to know which mutating webhook mutated the object in a API request, and what change did the
webhook apply.

The Kubernetes API server performs [auditing](/docs/tasks/debug/debug-cluster/audit/) on each
mutating webhook invocation. Each invocation generates an auditing annotation
capturing if a request object is mutated by the invocation, and optionally generates an annotation
capturing the applied patch from the webhook admission response. The annotations are set in the
audit event for given request on given stage of its execution, which is then pre-processed
according to a certain policy and written to a backend.

The audit level of a event determines which annotations get recorded:

- At `Metadata` audit level or higher, an annotation with key
  `mutation.webhook.admission.k8s.io/round_{round idx}_index_{order idx}` gets logged with JSON
  payload indicating a webhook gets invoked for given request and whether it mutated the object or not.

  For example, the following annotation gets recorded for a webhook being reinvoked. The webhook is
  ordered the third in the mutating webhook chain, and didn't mutated the request object during the
  invocation.

  ```yaml
  # the audit event recorded
  {
      "kind": "Event",
      "apiVersion": "audit.k8s.io/v1",
      "annotations": {
          "mutation.webhook.admission.k8s.io/round_1_index_2": "{\"configuration\":\"my-mutating-webhook-configuration.example.com\",\"webhook\":\"my-webhook.example.com\",\"mutated\": false}"
          # other annotations
          ...
      }
      # other fields
      ...
  }
  ```
  
  ```yaml
  # the annotation value deserialized
  {
      "configuration": "my-mutating-webhook-configuration.example.com",
      "webhook": "my-webhook.example.com",
      "mutated": false
  }
  ```
  
  The following annotation gets recorded for a webhook being invoked in the first round. The webhook
  is ordered the first in the mutating webhook chain, and mutated the request object during the
  invocation.

  ```yaml
  # the audit event recorded
  {
      "kind": "Event",
      "apiVersion": "audit.k8s.io/v1",
      "annotations": {
          "mutation.webhook.admission.k8s.io/round_0_index_0": "{\"configuration\":\"my-mutating-webhook-configuration.example.com\",\"webhook\":\"my-webhook-always-mutate.example.com\",\"mutated\": true}"
          # other annotations
          ...
      }
      # other fields
      ...
  }
  ```
  
  ```yaml
  # the annotation value deserialized
  {
      "configuration": "my-mutating-webhook-configuration.example.com",
      "webhook": "my-webhook-always-mutate.example.com",
      "mutated": true
  }
  ```

- At `Request` audit level or higher, an annotation with key
  `patch.webhook.admission.k8s.io/round_{round idx}_index_{order idx}` gets logged with JSON payload indicating
  a webhook gets invoked for given request and what patch gets applied to the request object.

  For example, the following annotation gets recorded for a webhook being reinvoked. The webhook is ordered the fourth in the
  mutating webhook chain, and responded with a JSON patch which got applied to the request object.
  
  ```yaml
  # the audit event recorded
  {
      "kind": "Event",
      "apiVersion": "audit.k8s.io/v1",
      "annotations": {
          "patch.webhook.admission.k8s.io/round_1_index_3": "{\"configuration\":\"my-other-mutating-webhook-configuration.example.com\",\"webhook\":\"my-webhook-always-mutate.example.com\",\"patch\":[{\"op\":\"add\",\"path\":\"/data/mutation-stage\",\"value\":\"yes\"}],\"patchType\":\"JSONPatch\"}"
          # other annotations
          ...
      }
      # other fields
      ...
  }
  ```
  
  ```yaml
  # the annotation value deserialized
  {
      "configuration": "my-other-mutating-webhook-configuration.example.com",
      "webhook": "my-webhook-always-mutate.example.com",
      "patchType": "JSONPatch",
      "patch": [
          {
              "op": "add",
              "path": "/data/mutation-stage",
              "value": "yes"
          }
      ]
  }
  ```

### Admission webhook metrics

The API server  exposes Prometheus metrics from the `/metrics` endpoint, which can be used for monitoring and
diagnosing API server status. The following metrics record status related to admission webhooks.

#### API server admission webhook rejection count

Sometimes it's useful to know which admission webhooks are frequently rejecting API requests, and the
reason for a rejection.

The API server exposes a Prometheus counter metric recording admission webhook rejections. The
metrics are labelled to identify the causes of webhook rejection(s):

- `name`: the name of the webhook that rejected a request.
- `operation`: the operation type of the request, can be one of `CREATE`,
  `UPDATE`, `DELETE` and `CONNECT`.
- `type`: the admission webhook type, can be one of `admit` and `validating`.
- `error_type`: identifies if an error occurred during the webhook invocation
  that caused the rejection. Its value can be one of:

  - `calling_webhook_error`: unrecognized errors or timeout errors from the admission webhook happened and the
    webhook's [Failure policy](#failure-policy) is set to `Fail`.
  - `no_error`: no error occurred. The webhook rejected the request with `allowed: false` in the admission
    response. The metrics label `rejection_code` records the `.status.code` set in the admission response.
  - `apiserver_internal_error`: an API server internal error happened.

- `rejection_code`: the HTTP status code set in the admission response when a
  webhook rejected a request.

Example of the rejection count metrics:

```
# HELP apiserver_admission_webhook_rejection_count [ALPHA] Admission webhook rejection count, identified by name and broken out for each admission type (validating or admit) and operation. Additional labels specify an error type (calling_webhook_error or apiserver_internal_error if an error occurred; no_error otherwise) and optionally a non-zero rejection code if the webhook rejects the request with an HTTP status code (honored by the apiserver when the code is greater or equal to 400). Codes greater than 600 are truncated to 600, to keep the metrics cardinality bounded.
# TYPE apiserver_admission_webhook_rejection_count counter
apiserver_admission_webhook_rejection_count{error_type="calling_webhook_error",name="always-timeout-webhook.example.com",operation="CREATE",rejection_code="0",type="validating"} 1
apiserver_admission_webhook_rejection_count{error_type="calling_webhook_error",name="invalid-admission-response-webhook.example.com",operation="CREATE",rejection_code="0",type="validating"} 1
apiserver_admission_webhook_rejection_count{error_type="no_error",name="deny-unwanted-configmap-data.example.com",operation="CREATE",rejection_code="400",type="validating"} 13
```

## Best practices and warnings

For recommendations and considerations when writing mutating admission webhooks,
see
[Admission Webhooks Good Practices](/docs/concepts/cluster-administration/admission-webhooks-good-practices).
